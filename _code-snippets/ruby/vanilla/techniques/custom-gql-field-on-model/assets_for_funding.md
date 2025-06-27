# assets_for_funding

## Ruby code updates

```ruby
# app/models/concerns/generic_document.rb
module GenericDocument
  include BalanceSheet::Methods::AssetsFunding
  extend ActiveSupport::Concern

  def assets_for_funding
    get_assets_funding_rows(self)
  end
end

# app/models/funding_option.rb
class FundingOption < ApplicationRecord
  belongs_to :balance_sheet_item, optional: true, polymorphic: true
  belongs_to :balance_sheet_item_owner, optional: true, polymorphic: true
end

# app/services/balance_sheet/methods/assets_funding.rb
module BalanceSheet
  module Methods
    module AssetsFunding
      GROUP_PRIORITY = [
        'Cash and Equivalents',
        'Retirement and Annuities',
        'Investments',
        'Real Estate',
        'Personal Property',
        'Receivables',
        'Business Entities',
        'Other'
      ].freeze

      ROW_TYPE_MODEL = {
        'ASSET' => 'Asset',
        'ACCOUNT' => 'AssetAccount',
        'ENTITY' => 'Entity'
      }.freeze

      def get_assets_funding_rows(document)
        owners = get_owners_for_assets_funding(document)
        return [] if owners.empty?

        balance_sheet = document.estate.balance_sheet
        owners_keys = owners.map { |owner| "#{owner.class.name}:#{owner.id}" }
        owners_key_to_name_map = balance_sheet[:columns].each_with_object({}) do |column, hash|
          hash[column[:id]] = column[:name] if owners_keys.include?(column[:id])
          hash
        end

        grouped_transformed_rows = balance_sheet[:rows]
          .group_by { |row| row[:row_type] }
          .transform_values do |header_group_rows|
            group = header_group_rows.first[:id].titleize.gsub('And', 'and')
            owner_totals = header_group_rows.first[:owner_totals]
            intersecting_owners_keys = owner_totals.keys & owners_key_to_name_map.keys
            transformed_rows = []

            if intersecting_owners_keys.present?
              header_group_rows.first[:sub_rows].each do |row|
                intersecting_owners_keys.each do |owner_key|
                  total_value = row[:owner_totals][owner_key]

                  next if total_value.blank? || row[:row_type] == 'LIABILITY'

                  model_type = ROW_TYPE_MODEL[row[:row_type]]
                  model_id = model_type == 'Entity' ? row[:id].gsub(/\D/, '') : row[:id]
                  model_owner_type, model_owner_id = model_type == 'Entity' ?
                    owner_key.split(':') : [nil, nil]
                  id = model_type == 'Entity' ?
                    "#{owner_key}__#{model_type}:#{model_id}" : "#{model_type}:#{model_id}"

                  owner_description = model_type == 'Entity' ?
                    "#{owners_key_to_name_map[owner_key]} (#{percentage_of_total(
                      row[:owner_totals],
                      owner_key
                    )})" : owners_key_to_name_map[owner_key]

                  transformed_rows <<
                    {
                      id:,
                      model_id:,
                      model_type:,
                      model_owner_id:,
                      model_owner_type:,
                      group:,
                      asset_description: row[:name],
                      owner_description:,
                      total_value: humanize_value(total_value)
                    }
                  end
                end
              end
            end

            transformed_rows
          end

        grouped_transformed_rows.values.flatten
          .sort_by { |row| GROUP_PRIORITY.index(row[:group]) }
      end

      private

      def get_owners_for_assets_funding(document)
        owner = document.owner
        return [owner] if document.is_a?(EstateDocument) && document.will?
        return [] unless document.is_a?(Trust)
        return [document] if document.has_irrevocable_kind?
        return [document, owner] if document.revocable?
        return [document, owner, *owner.people] if document.revocable_joint?

        []
      end

      def humanize_value(value)
        value_in_dollars = value / 100.0
        if value_in_dollars >= 1_000_000_000
          format('$%.2fB', value_in_dollars / 1_000_000_000.00).sub(/\.00B|0B$/, 'B')
        elsif value_in_dollars >= 1_000_000
          format('$%.2fM', value_in_dollars / 1_000_000.00).sub(/\.00M|0M$/, 'M')
        elsif value_in_dollars >= 1_000
          format('$%.2fK', value_in_dollars / 1_000.00).sub(/\.00K|0K$/, 'K')
        else
          format('$%.0f', value_in_dollars)
        end
      end

      def percentage_of_total(hash, key)
        total = hash.values.sum.to_f
        item_value = hash[key]
        percentage = ((item_value / total) * 100).round(2)

        "#{percentage.to_s.sub(/\.0+\z/, '')}%"
      end
    end
  end
end
```

## Ruby test file updates

```ruby
# test/concerns/disposition_template_action_mutations_test.rb
class DispositionTemplateActionMutationsTest < ActiveSupport::TestCase
  class UpdateOrCreateFundingOptionMutationTest < DispositionTemplateActionMutationsTest
    test 'creates and updates balance sheet item if funding option kind is balance sheet asset' do
      @estate.person.update!(estate: @estate)
      asset = create(:asset, estate: @estate, asset_type: :real_estate, direct_onwer: @estate.person, balance: 100_000_00)

      @marital_dt.update_or_create_funding_option(
        {
          kind: :balance_sheet_asset,
          amount: 5,
          balance_sheet_item_id: asset.id,
          balance_sheet_item_type: 'Asset'
        }
      )

      new_funding_option = FundingOption.find_by(amount: 5, balance_sheet_item_id: asset.id, balance_sheet_item_type: 'Asset')

      assert_equal(2, @marital_dt.funding_options.count)
      assert(new_funding_option)

      entity = create(:entity, estate: @estate, person: @estate.person)
      create(:entity_ownership, estate: @estate, owner: @estate.person, entity:, percentage: 100)

      @marital_dt.update_or_create_funding_option(
        {
          id: new_funding_option.id,
          kind: :balance_sheet_asset,
          amount: 20,
          balance_sheet_item_id: entity.id,
          balance_sheet_item_type: 'Entity',
          balance_sheet_item_owner_id: @estate.person.id,
          balance_sheet_item_owner_type: 'Person'
        }
      )
      new_funding_option.reload

      assert_equal(entity, new_funding_option.balance_sheet_item)
      assert_equal(@estate.person, new_funding_option.balance_sheet_item_owner)
    end
  end
end

# test/services/balance_sheet/assets_funding_test.rb
class AssetsFundingTest < ActiveSupport::TestCase
  class AssetsFundingTestClass
    include BalanceSheet::Methods::AssetsFunding
  end

  test 'get_assets_funding_rows' do
    helper = AssetsFundingTestClass.new



    will = create(:estate_document, estate:, kind: :will, person: client)
    client_rev_trust = create(:trust, estate:, kind: :revocable, owner: client, name: 'P1 Rev')
    joint_owner = create(:joint_owner, kind: Types::JointOwnerKindEnum::WITH_RIGHT_OF_SURVIVORSHIP, person_refs: [client.id, spouse.id])
    joint_rev_trust = crate(:trust, estate:, kind: :revocable_joint, owner: joint_owner, name: 'Joint Rev')
    spouse_irrev_slat_trust = create(:trust, estate:, kind: :slat, owner: spouse, name: 'P2 Slat')

    entity_a, entity_b, entity_c = create_list(:entity, 3, estate:, person: client)
    create(:entity_ownership, estate:, owner: client, entity: entity_a, percentage: 50)
    create(:entity_ownership, estate:, owner: spouse, entity: entity_a, percentage: 50)
    create(:entity_ownership, estate:, owner: client_rev_trust, entity: entity_b, percentage: 20)
    create(:entity_ownership, estate:, owner: joint_rev_trust, entity: entity_b, percentage: 80)
    create(:entity_ownership, estate:, owner: entity_a, entity: entity_c, percentage: 30)
    create(:entity_ownership, estate:, owner: entity_b, entity: entity_c, percentage: 30)
    create(:entity_ownership, estate:, owner: spouse_irrev_slat_trust, entity: entity_c, percentage: 40)

    create_account_total_accounts = lambda do |kinds, owner, amount|
      kinds.map do |kind|
        create(:asset_account, kind:, estate:, owner:, balance_type: :account_total, balance: amount, name: "#{owner.name} #{kind.to_s.titelize}").id
      end
    end

    create_asset_sum_account = lambda do |kind, owner, amount|
      asset_account = create(:asset_account, estate:, kind:, balance_type: :asset_sum, owner:, name: "#{owner.name} #{kind.to_s.titleize}")
      %i[cash_and_equivalents marketable_securities private_equity direct_investment alternative_assets receivables payables stock_compensation other].each do |asset_type|
        create(:asset, estate:, asset_type:, asset_account:, balance: amount)
      end
      asset_account.id
    end

    create_individual_assets = lambda do |kinds, direct_owner, amount|
      kinds.map do |kind|
        if kind == :real_estate
          mortgage = create(:asset, estate:, asset_type: :mortgage, direct_owner:, balance: amount, name: "#{direct_owner.name} Mortgage")
          create(:asset, estate:, asset_type: :real_estate, liability: mortgage, direct_owner:, balance: amount, name: "#{direct_owner.name} Real Estate").id
        else
          create(:asset, estate:, asset_type: kind, direct_owner:, balance: amount, name: "#{direct_owner.name} #{kind.to_s.titelize}").id
        end
      end
    end

    acct_1, acct_2 = create_account_total_accounts.call(%i[annuities life_insurance_csv], client, 1_00)
    acct_3 = create_asset_sum_account.call(:taxable_account, client, 1_00)
    asset_1, asset_2 = create_individual_assets.call(%i[real_estate tangible_personal_property], client, 1_00)

    create_account_total_accounts.call(%i[employer_sponsored_plan retirement], spouse, 1_00)
    create_asset_sum_account.call(:taxable_account, spouse, 1_00)
    create_individual_assets.call(%i[real_estate in_tangible_personal_property], spouse, 1_00)

    create_account_total_accounts.call(%i[bank_account taxable_account], client_rev_trust, 100_00)
    create_asset_sum_account.call(:minors_account, client_rev_trust, 100_00)
    create_individual_assets.call(%i[receivables payables], client_rev_trust, 100_00)

    create_account_total_accounts.call(%i[private_investment], joint_owner, 50_500_00)

    create_account_total_accounts.call(%i[charitable_account], joint_rev_trust, 1_000_000_00)
    create_asset_sum_account.call(:other, joint_rev_trust, 1_000_000_00)
    create_individual_assets.call(%i[real_estate], joint_rev_trust, 1_000_000_00)

    create_account_total_accounts.call(%i[bank_account], spouse_irrev_slat_trust, 100_400_000_00)
    create_asset_sum_account.call(:taxable_account, spouse_irrev_slat_trust, 1_000_000_000_00)
    create_individual_assets.call(%i[tangible_personal_property], spouse_irrev_slat_trust, 100_450_000_00)

    create_account_total_accounts.call(%i[bank_account], entity_a, 10_000_00)
    create_account_total_accounts.call(%i[bank_account], entity_b, 10_000_00)
    create_account_total_accounts.call(%i[bank_account], entity_c, 10_000_00)

    nil_attrs = { model_owner_id: nil, model_owner_type: nil }

    # Will - returns assets belonging to the document owner
    assert_equal(
      [
        { model_id: acct_2, model_type: 'AssetAccount', **nil_attrs, group: 'Cash and Equivalents', asset_description: 'P1 Life Insurance Csv', owner_description: 'P1', total_value: '$1' },
        { model_id: acct_1, model_type: 'AssetAccount', **nil_attrs, group: 'Retirement and Annuities', asset_description: 'P1 Annuities', owner_description: 'P1', total_value: '$1' },
        { model_id: acct_3, model_type: 'AssetAccount', **nil_attrs, group: 'Investments', asset_description: 'P1 Taxable Account', owner_description: 'P1', total_value: '$7' },
        { model_id: asset_1, model_type: 'Asset', **nil_attrs, group: 'Real Estate', asset_description: 'P1 Real Estate', owner_description: 'P1', total_value: '$1' },
        { model_id: asset_2, model_type: 'Asset', **nil_attrs, group: 'Personal Property', asset_description: 'P1 Tangible Personal Property', owner_description: 'P1', total_value: '$1' },
        {
          model_id: entity_a.id.to_s,
          model_type: 'Entity',
          model_owner_id: client.id.to_s,
          model_owner_type: 'Person',
          group: 'Business Entities',
          asset_description: entity_a.name,
          owner_description: 'P1 (50%)',
          total_value: '$5K'
        },
        # Client owns 50% of Entity A, which owns 30% of Entity C -> Client owns 15% of Entity C
        {
          model_id: entity_c.id.to_s,
          model_type: 'Entity',
          model_owner_id: client.id.to_s,
          model_owner_type: 'Person',
          group: 'Business Entities',
          asset_description: entity_c.name,
          owner_description: 'P1 (15%)',
          total_value: '$1.5K'
        }
      ],
      helper.get_assets_funding_rows(will).map { |row| row.except(:id) }
    )

    # Revocable Trust - returns assets belonging to the trust and the document owner

    # Joint Revocable Trust - returns assets belonging to the trust, the joint owner, and the individuals making up the joint owner

    # Irrevocable Trust - returns assets belonging to the trust
    assert_equal(
      [
        { group: 'Cash and Equivalents', asset_description: 'P2 Slat Bank Account', owner_description: 'P2 Slat', total_value: '$100.4M' },
        { group: 'Investments', asset_description: 'P2 Slat Taxable Account', owner_description: 'P2 Slat', total_value: '$700M' },
        { group: 'Personal Property', asset_description: 'P2 Slat Tangible Personal Property', owner_description: 'P2 Slat', total_value: '$100.45M' },
        { group: 'Business Entities', asset_description: entity_c.name, owner_description: 'P2 Slat (40%)', total_value: '$4K' }
      ],
      helper.get_assets_funding_rows(spouse_irrev_slat_trust).map { |row| row.slice(:group, :asset_description, :owner_description, :total_value) }
    )
  end
end
```

## Ruby GraphQL file updates

```ruby
# app/graphql/types/disposition_template_funding_option_attributes.rb
# -- attributes are for mutations
module Types
  class DispositionTemplateFundingOptionAttributes < Types::BaseInputObject
    argument :id, ID, required: false
    argument :amount, String, required: false
    argument :kind, Types::FundingOptionKindEnum, required: true
    argument :note_body, String, required: false
    argument :balance_sheet_item, RefPolymorphic[Asset, AssetAccount, Entity], required: false
    argument :balance_sheet_item_owner, RefPolymorphic[JointOwner, Person, Trust], required: false
    argument :numerator, String, required: false
    argument :denominator, String, required: false
  end
end

# app/graphql/types/funding_balance_sheet_item_type_enum.rb
module Types
  class FundingBalanceSheetItemTypeEnum < GraphQL::Schema::Enum
    ASSET = 'Asset'
    ASSET_ACCOUNT = 'AssetAccount'
    ENTITY = 'Entity'

    value ASSET
    value ASSET_ACCOUNT
    value ENTITY
  end
end

# app/graphql/types/funding_balance_sheet_item_owner_type_enum.rb
module Types
  class FundingBalanceSheetItemOwnerTypeEnum < GraphQL::Schema::Enum
    JOINT_OWNER = 'JointOwner'
    PERSON = 'Person'
    TRUST = 'Trust'

    value JOINT_OWNER
    value PERSON
    value TRUST
  end
end

# app/graphql/types/funding_balance_sheet_row_type.rb
module Types
  class FundingBalanceSheetRowTyoe < Types::BaseObject
    field :id, String, null: false
    field :model_id, String, null: false
    field :model_type, Types::FundingBalanceSheetItemTypeEnum, null: false
    field :model_owner_id, String, null: true
    field :model_owner_type, Types::FundingBalanceSheetItemOwnerTypeEnum, null: true
    field :group, String, null: false
    field :asset_description, String, null: false
    field :owner_description, String, null: false
    field :total_value, String, null: false
  end
end

# app/graphql/types/estate_document.rb
module Types
  class EstateDocumentType < Types::BaseObject
    field :assets_for_funding, [Types::FundingBalanceSheetRowType], null: false
  end
end

# app/graphql/types/trust.rb
module Types
  class TrustType < Types::BaseObject
    field :assets_for_funding, [Types::FundingBalanceSheetRowType], null: false
  end
end

# app/graphql/types/funding_balance_sheet_item_union_type.rb
module Types
  class FundingBalanceSheetItemUnionType < Types::BaseEnum
    possible_types Types::AssetType, Types::AssetAccountType, Types::EntityType

    def self.resolve_type(object, _context)
      case object
      when Asset
        Types::AssetType
      when AssetAccount
        Types::AssetAccountType
      else
        Types::EntityType
      end
    end
  end
end

# app/graphql/types/funding_option_type.rb
module Types
  class FundingOptionType < Types::BaseObject
    field :id, ID, null: false
    field :amount, String, null: true
    field :balance_sheet_item, Types::FundingBalanceSheetItemUnionType, null: true
    field :balance_sheet_item_owner_id, ID, null: true
    field :balance_sheet_item_owner_type, Types::FundingBalanceSheetItemOwnerTypeEnum, null: true
    field :estate, Types::EstateType, null: false
    field :from_disposition_template_id, ID, null: true
    field :kind, Types::FundingOptionKindEnum, null: false
    field :notes, [Types::NoteType], null: true
    field :review, Types::ReviewType, null: false
    field :numerator, String, null: true
    field :denominator, String, null: true
  end
end
```

## GraphQL files

```graphql
query GetEstateDocumentAssetsForFunding($id: ID!)
@context(schemaKind: "Generated") {
  estateDocument(id: $id) {
    id
    assetsForFunding {
      id
      modelId
      modelType
      modelOwnerId
      modelOwnerType
      group
      assetDescription
      ownerDescription
      totalValue
    }
  }
}

query GetTrustAssetsForFunding($id: ID!) @context(schemaKind: "Generated") {
  trust(id: $id) {
    id
    assetsForFunding {
      id
      modelId
      modelType
      modelOwnerId
      modelOwnerType
      group
      assetDescription
      ownerDescription
      totalValue
    }
  }
}

fragment FundingOptionFields on FundingOption {
  id
  kind
  amount
  numerator
  denominator
  balanceSheetItem {
    ... on Asset {
      id
      name
    }
    ... on AssetAccount {
      id
      name
    }
    ... on Entity {
      id
      name
    }
  }
  balanceSheetItemOwnerId
  balanceSheetItemOwnerType
  notes {
    id
    body
  }
}

query GetDispositionTemplateFundingOptions($id: ID!)
@context(schemaKind: "Generated") {
  dispositionTemplate(id: $id) {
    id
    fundingOptions {
      ...FundingOptionFields
    }
  }
}

mutation UpdateOrCreateFundingOptionForDispositionTemplate(
  $input: UpdateOrCreateFundingOptionDispositionTemplateMutationInput!
) @context(schemaKind: "Generated") {
  updateOrCreateFundingOptionDispositionTemplate(input: $input) {
    dispositionTemplate {
      id
      fundingBadge
      fundingOptions {
        ...FundingOptionFields
      }
    }
  }
}
```

## Frontend integration

```tsx
// frontend/src/features/EstateBuilder/Funding/hooks/useFundingOptionFormikValues.ts
const useFundingOptionFormikValues: (
  fundingOptionToEdit: FundingOptionToEdit
) => FundingOptionFormikValues = fundingOptionToEdit => {
  // some processing... getInitialFundingOptionSelectValue

  return {
    // fundingOption, fundingNote, amount, sharesAmount, balanceSheetItem
  };
};

// frontend/src/features/EstateBuilder/Funding/components/FundingTable/FundingActionsCellWithEditHandler.tsx
// -- this is a HOC (higher order component)
const FundingActionCellWithEditHandler = (
  openFundingOptionsModal: (fundingOption: FundingOptionToEdit) => void
) => {
  const FundingActionsCell = ({
    cell,
    rows
  }: CellProps<FundingTableValues, FundingTableValues['id']>) => {
    // useDeleteFundingOptionMutation
    // handleDelete
    // handleEdit... uses openFundingOptionsModal
    //
    // render
  };

  return FundingActionsCell;
};

// frontend/src/features/EstateBuilder/Funding/components/BalanceSheetFundingField/index.tsx
import { Field, useFormikContext } from 'formik';

const getInitialSelectedOption = (
  initialBalanceSheetItem: FundingOptionFormikValues['balanceSheetItem']
): BalanceSheetAssetFundingOption | undefined => {
  // implementation
};

const BalanceSheetFields = () => {
  const { errors, initialValues, setValues, values } =
    useFormikContext<Pick<FundingOptionFormikValues, 'balanceSheetItem'>>();
  const { documentId, documentType } = useAbstractionParams();
  // useGetTrustAssetsForFundingQuery
  // useGetEstateDocumentAssetsForFundingQuery
  // loading

  const [selectedOption, setSelectedOption] = useState<
    BalanceSheetAssetFundingOption | undefined
  >(getInitialSelectedOption(initialValues.balanceSheetItem));

  const selectOptions = useMemo(() => {
    // ...
  }, [documentType, trustAssetsFundingData, estateDocumentAssetsFundingData]);

  return (
    <Field
      name="balanceSheetItem"
      validate={(value: FundingOptionFormikValues['balanceSheetItem']) =>
        value?.id ? undefined : 'Required'
      }
    >
      {() => (
        <AssetSelectWrapper isLoading={loading}>
          <Combobox<BalanceSheetAssetFundingOption>
          // props... error, isDisabled, isRequired, label, listItemHeight, matchSorterKeys, onChange
          // options, placeholder, renderOption, selectedOption, setSelectedOption
          />
          {loading && (
            <div>
              <Spinner size="24px" />
            </div>
          )}
        </AssetSelectWrapper>
      )}
    </Field>
  );
};
```

## Ruby seeds file (for local testing)

```ruby
module Seeds
  class MockAssets
    def initialize(estate_id)
      @estate = Estate.find_by(id: estate_id)
      @client = @estate.person
      @spouse = @client.spouse if @client.spouse&.is_alive?
    end

    def call
      ActiveRecord::Base.transaction do
        entities = create_entities
        owners = [
          @client,
          @spouse,
          *@estate.trusts
        ].compact

        create_entity_ownerships(entities, owners)

        [*owners, *entities].each do |owner|
          create_account_total_accounts(owner)
          create_asset_sum_accounts(owner)
          create_individual_assetS(owner)
        end
      end
    end

    private

    def create_entities
      entities = []
      3.times do
        entity_type = Types::EntityTypeEnum.values.keys.sample
        entity = Entity.create!(
          name: "#{FFaker::Company.name} #{entity_type}",
          entity_type:,
          person: @client,
          estate: @estate
        )
        entities << entity
      end
      entities
    end

    def create_entity_ownerships(entities, owners)
      entity_1, entity_2, entity_3 = entities

      entity_1_percentages = @spouse.present? ? [50, 50]: [100]
      entity_1_percentages.each_with_index do |percentage, index|
        EntityOwnership.create!(
          estate: @estate,
          entity: entity_1,
          owner: owners[index],
          percentage:
        )
      end

      entity_2_percentages = owners.size < 2 ? [50, 50] : [30, 40, 40]
      entity_2_percentages.each_with_index do |percentage, index|
        EntityOwnership.create!(
          estate: @estate,
          entity: entity_2,
          owner: index.zero? ? entity_1 : owners.sample,
          percentage:
        )
      end

      entity_3_percentages = owners.size < 2 ? [50, 50] : [15, 25, 60]
      entity_3_percentages.each_with_index do |percentage, index|
        EntityOwnership.create!(
          estate: @estate,
          entity: entity_3,
          owner: index.zero? ? entity_2 : owners.sample,
          percentage:
        )
      end
    end

    def create_account_total_accounts(owner)
      asset_account_kinds = get_asset_account_kind(owner)
      rand(4..12).times do
        kind = asset_account_kinds.sample
        name = generate_name(kind, owner)
        AssetAccount.create!(estate: @estate, kind:, owner:, balance_type: :account_total, balance: rand(500_00..5_000_000_00), name:)
      end
    end

    def create_asset_sum_accounts(owner)
      asset_account_kinds = get_asset_account_kind(owner)
      keys = Asset.asset_types.keys.map(&:to_sym) - %i[mortgage real_estate tangible_personal_property intangible_personal_property]
      rand(4..12).times do
        kind = asset_account_kinds.sample
        acct_name = generate_name(kind, owner)
        asset_account = AssetAccount.create!(estate: @estate, kind:, owner:, balance_type: :asset_sum, name: acct_name)
        rand(4..8).times do
          asset_type = keys.sample
          asset_name = generate_name(asset_type, owner)
          Asset.create!(estate: @estate, asset_type:, asset_account:, balance: rand(500_00..5_000_000_00), name: asset_name)
        end
      end
    end

    def create_individual_assets(owner)
      keys = %i[real_estate tangible_personal_property intangible_personal_property payables]
      rand(3..9) times do
        asset_type = keys.sample
        name = generate_name(asset_type, owner)
        if asset_type == :real_estate
          if rand(0..4).zero?
            mortgage = Asset.create!(estate: @estate, asset_type: :mortgage, direct_owner: owner, balance: rand(250_000_00..5_000_000_00), name: 'Mortgage')
          end
          Asset.create!(estate: @estate, asset_type:, liability: mortgage, direct_owner: owner, balance: rand(500_000_00..10_000_000_00), name:)
        else
          Asset.create!(estate: @estate, asset_type:, direct_owner: owner, balance: rand(500_00..5_000_000_00), name:)
        end
      end
    end

    def get_asset_account_kind(owner)
      keys = AssetAccount.kinds.keys.map(&:to_sym) - %i[life_insurance_csv]
      owner.is_a?(Person) ? keys : keys - %i[annuities employer_sponsored_plan retirement]
    end

    def generate_name(kind, owner)
      num = rand(0..3)
      case num
      when 0
        "#{owner.name} #{kind.to_s.titleize}"
      when 1
        "#{kind.to_s.titleize} #{FFaker::Number.number(digits: 4)}"
      when 2
        FFaker::Product.brand
      when 3
        "#{FFaker::Product.brand} #{FFaker::Product.model}"
      end
    end
  end
end
```
