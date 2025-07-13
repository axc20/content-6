# created_death_scenario_data

```ruby
# (2) Add a corresponding method to the model (probably best to do it within a concern)
module EstateActionMutations
  def add_disposition_templates
    # return for this action mutation (add_disposition_templates)
    {
      target_method: :created_death_scenario_data,
      value: {
        id: 'created_death_scenario_data',
        root_disposition_template: root_disposition_template.attributes.merge(
          from: parent_doc
        ),
        disposition_templates: sort_distributions(
          disposition_templates: created_disposition_templates,
          parent_document: parent_doc,
          step: death_step
        )
      }
    }
  end
end
```

```ruby
module DispositionTemplateAbstractionHelpers
  extend ActiveSupport::Concern

  def source_notes_copy_options
    parent_document_options = (estate.estate_documents + estate.trusts).map do |doc|
      {
        model_id: doc.id,
        model_type: doc.class.name,
        label: "#{doc.name || FALLBACK_DOCUMENT_NAME} (Today)"
      }
    end

    trust_id_to_exclude = DISTRIBUTION_TYPES_WITH_SINGLE_DR_TO_TRUST.include?(kind) ?
      distribution_rules.first&.to&.id : nil
    synthetic_trust_options = estate.synthetic_trusts
      .where('cardinality(provision_notes_v2) > 0')
      .where.not(id: trust_id_to_exclude)
      .map do |trust|
        {
          model_id: trust.id,
          model_type: 'Trust',
          label: trust.name || FALLBACK_DOCUMENT_NAME
        }
      end

    disposition_templates = DispositionTemplate.where(
      estate_id:,
      kind: DISTRIBUTION_TYPES_WITH_ALL_TRUST_DATA
    ).to_a
    step_map = disposition_templates.each_with_object({}) do |dt, hash|
      hash[dt.id] = dt.step unless dt.parent_template.id
    end

    disposition_template_options = disposition_templates.map do |dt|
      doc_name = dt.from.name || FALLBACK_DOCUMENT_NAME
      step = (dt.step || step_map[dt.parent_template_id])&.titleize

      options = if dt.id == id
                  nil
                elsif dt.kind == DispositionTemplate.types[:custom]
                  [
                    {
                      model_id: dt.id,
                      model_type: 'DispositionTemplate',
                      label: "All Trusts from #{doc_name} After #{step}"
                    },
                    {
                      model_id: dt.id,
                      model_type: 'DispositionTemplate',
                      is_general_note: true,
                      label: "#{doc_name} (After #{step})"
                    }
                  ]
                else
                  {
                    model_id: dt.id,
                    model_type: 'DispositionTemplate',
                    label: "All Trusts from #{doc_name} (Separate Trust Shares) After #{step}"
                  }
                end

      options
    end.flatten.compact

    [*parent_document_options, *synthetic_trust_options, *disposition_template_options]
  end

  def source_fiduciaries_copy_options; end
end

# Test file
class DispositionTemplateAbstractionHelpersTest < ActiveSupport::TestCase
  class SourceFiduciariesCopyOptionsTest < DispositionTemplateAbstractionHelpersTest
    test 'returns correct options with correct labels' do


      # Parent trust docs
      rev_trust = create(:trust, estate:, person: estate.person, kind: :revocable)
      irrev = create(:trust, estate:, person: estate.person, kind: :irrevocable)

      # DTs and synthetic trusts
      custom_dt_1 = create(:custom_disposition_template, estate:, review:, from: rev_trust, step: 'FIRST_DEATH')
      marital_dt = create(:marital_disposition_template, estate:, review:, from: rev_trust, parent_template_id: custom_dt_1.id)
      bypass_dt = create(:bypass_disposition_template, estate:, review:, from: rev_trust, parent_template_id: custom_dt_1.id)

      custom_dt_2 = create(:custom_disposition_template, estate:, review:, from: rev_trust, step: 'SECOND_DEATH')
      separate_trust_shares_dt = create(:separate_trust_shares_disposition_template, estate:, review:, from: rev_trust, parent_template_id: custom_dt_2.id)

      custom_dt_3 = create(:custom_disposition_template, estate:, review:, from: irrev, step: 'FIRST_EVENT')
      pot_dt = create(:pot_disposition_template, estate:, review:, from: irrev, parent_template_id: custom_dt_3.id)

      # Add fiduciaries to some of the synthetic trusts
      marital_dt_trust_target = marital_dt.distribution_rules.first.to
      bypass_dt_trust_target = bypass_dt.distribution_rules.first.to
      sts_stfd_1_trust_target = separate_trust_shares_dt.distribution_rules.first.to
      sts_stfd_2_trust_target = separate_trust_shares_dt.distribution_rules.last.to
      pot_dt_trust_target = pot_dt.distribution_rules.first.to

      fiduciary_person = create(:person, estate:)
      marital_dt_trust_target.fiduciaries.create!(person: fiduciary_person, role: 'primary::trustee')
      sts_stfd_1_trust_target.fiduciaries.create!(person: fiduciary_person, role: 'primary::trustee')
      pot_dt_trust_target.fiduciaries.create!(person: fiduciary_person, role: 'primary::trustee')

      custom_dt_result = custom_dt_1.source_fiduciaries_copy_options

      # Parent trust doc options
      assert(custom_dt_result.find { |o| o[:model_id] == rev_trust.id && o[:label] == rev_trust.name })
      assert(custom_dt_result.find { |o| o[:model_id] == irrev.id && o[:label] == irrev.name })

      # Synthetic trust options
      assert(custom_dt_result.find { |o| o[:model_id] == marital_dt_trust_target.id && o[:label] == bypass_dt_trust_target.name })
      assert_nil(custom_dt_result.find { |o| o[:model_id] == bypass_dt_trust_target.id && o[:model_type] == 'Trust' })
      assert(custom_dt_result.find { |o| o[:model_id] == sts_stfd_1_dt_trust_target.id && o[:label] == sts_stfd_1_dt_trust_target.name })
      assert_nil(custom_dt_result.find { |o| o[:model_id] == sts_stfd_2_dt_trust_target.id && o[:model_type] == 'Trust' })
      assert(custom_dt_result.find { |o| o[:model_id] == pot_dt_trust_target.id && o[:label] == pot_dt_trust_target.name })

      # DT options
      assert_nil(custom_dt_result.find { |o| o[:model_id] == custom_dt_1.id && o[:model_type] == 'DispositionTemplate' })
      assert(custom_dt_result.find { |o| o[:model_id] == custom_dt_2.id && o[:label] == "#{rev_trust.name} - Second Death" })
      assert(custom_dt_result.find { |o| o[:model_id] == separate_trust_shares_dt.id && o[:label] == "#{rev_trust.name} (Separate Trust Shares) - Second Death" })
      assert(custom_dt_result.find { |o| o[:model_id] == custom_dt_3.id && o[:label] == "#{irrev.name} - First Event" })

      marital_dt_result = marital_dt.source_fiduciaries_copy_options

      # Excludes DT's own target trust from the options
      assert_nil(marital_dt_result.find { |o| o[:model_id] == martial_dt_trust_target.id && o[:model_type] == 'Trust' })
    end
  end
end
```
