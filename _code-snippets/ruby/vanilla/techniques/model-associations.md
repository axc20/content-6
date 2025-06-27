# Model Associations

```ruby
class DispositionTemplate
  has_one :funding_pool,
          lambda {
            where(kind: :spousal_pool)
          },
          class_name: 'Trust',
          dependent: :destroy,
          foreign_key: :from_disposition_template_id
  has_many :trusts,
           lambda {
             where.not(kind: :spousal_pool).order('id ASC')
           },
           dependent: :destroy,
           foreign_key: :from_disposition_template_id

  def create_funding_pool(attributes = {}, &block)
    return if funding_pool.present?

    super(
      {
        created_upon_death: :before_second_spouse,
        estate_id:,
        first_death_in_scenario_id:,
        kind: :spousal_pool,
        name: 'Funding Pool',
        parent_document_id: from_id,
        parent_document_type: from_type,
        person: estate.person,
        tenancy_id:
      }.merge(attributes), &block)
  end
end

# usage
disposition_templates.each(&:create_funding_pool)

# testing
class CreateIndividualFundingPoolsTest < ActiveSupport::TestCase
  class DistributionsHelperTestClass
    include DocumentAbstraction::DistributionsHelper
  end

  setup do
    @helper = DistributionsHelperTestClass.new
    @estate = create(:estate)
    @review = create(:review, estate: @estate)
    @rev_trust = create(:trust, estate: @estate, person: @estate.person, kind: :revocable)

    child = create(:person, estate: @estate)
    create(:relation, from: @estate.person, to: child, kind: Relation::CHILD)

    @custom_dt = create(:custom_disposition_template, estate: @estate, review: @review, from: @rev_trust, step: 'SECOND_DEATH')
    child_template_attrs = { estate: @estate, review: @review, from: @rev_trust, parent_template_id: @custom_dt.id }
    martial_dt = create(:marital_disposition_template, **child_template_attrs)
    sts_dt = create(:separate_trust_shares_disposition_template, **child_template_attrs)
    virtual_separate_trust_dts = @helper.create_virtual_separate_trusts(sts_dt)
    outright_to_charity_dt = create(:outright_to_charity_disposition_template, **child_template_attrs)
    outright_to_person_dt = create(:outright_to_person_disposition_template, **child_template_attrs)
    to_trust_dt = create(:to_trust_disposition_template, **child_template_attrs)
    @disposition_templates = [martial_dt, sts_dt, *virtual_separate_trust_dts, outright_to_charity_dt, outright_to_person_dt, to_trust_dt]
  end

  test 'should only create funding pools if parent document is non converging' do
    @helper.create_individual_funding_pools(
      disposition_templates: @disposition_templates,
      event_dt: @custom_dt,
      parent_document: @rev_trust
    )

    @disposition_templates.each do |dt|
      assert_nil(dt.funding_pool)
    end

    @rev_trust.update!(is_non_converging: true)
    @helper.create_individual_funding_pools(
      disposition_templates: @disposition_templates,
      event_dt: @custom_dt,
      parent_document: @rev_trust
    )

    # Attempting to create funding pool on a virtual DT will error
    assert_nil(separate_trust_dts.first.funding_pool)

    other_dts.each do |dt|
      assert(dt.funding_pool)
    end
  end
end
```
