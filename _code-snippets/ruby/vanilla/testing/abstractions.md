# Abstractions

## Provision Notes

```ruby
vanilla_provision_note = create(:organization_provision, :vanilla_provision, text: 'n1')
```

## Disposition Templates

```ruby
# Finding the event (root) DTs:
first_to_die_id = estate.person.id
event_dt = rev_trust.disposition_templates.detect { |dt| dt.step == 'SECOND_DEATH' && dt.first_death_in_scenario_id == first_to_die_id }

# Creating child DTs:
child_dt_attrs = { estate:, review:, from: rev_trust, parent_template_id: event_dt.id, first_death_in_scenario_id: first_to_die_id }

to_trust_dt_1 = create(:to_trust_disposition_template, **child_dt_attrs)
irrev_parent_doc = create(:trust, estate:, kind: :irrevocable, name: 'IRREV parent doc', owner: estate.person, person: estate.person)
DocumentAbstraction::DistributionsService.new(
  to_trust_dt_1,
  DocumentAbstraction::Types::DistributionDetailsParams.new(
    amount_kind: 'dollars',
    amount: '10000',
    target_id: irrev_parent_doc.id.to_s,
    target_trust_kind: irrev_parent_doc.kind
  )
).update

to_trust_dt_2 = create(:to_trust_disposition_template, **child_dt_attrs)
revocable_subtrust = rev_trust.synthetic_trusts.create!(
  estate:,
  review:,
  kind: :revocable,
  name: 'REV subtrust',
  owner: estate.person,
  person: estate.person,
  created_upon_death: 'second_spouse',
  first_death_in_scenario_id: first_to_die_id
)
revocable_subtrust.fiduciaries.create!(person: child1, role: 'primary::trustee')
revocable_subtrust.update!(provision_notes_v2: [vanilla_provision_note.id])
DocumentAbstraction::DistributionsService.new(
  to_trust_dt_2,
  DocumentAbstraction::Types::DistributionDetailsParams.new(
    amount_kind: 'dollars',
    amount: '10000',
    target_id: revocable_subtrust.id.to_s,
    target_trust_kind: revocable_subtrust.kind
  )
).update

to_trust_dt_3 = create(:to_trust_disposition_template, **child_dt_attrs)
stfd = create(:trust, estate:, review:, kind: :stfd, parent_document: rev_trust, created_upon_death: :second_spouse, first_death_in_scenario_id: first_to_die_id)
stfd.fiduciaries.create!(person: child1, role: 'primary::trustee')
stfd.update!(provision_notes_v2: [vanilla_provision_note.id])
DocumentAbstraction::DistributionsService.new(
  to_trust_dt_3,
  DocumentAbstraction::Types::DistributionDetailsParams.new(
    amount_kind: 'dollars',
    amount: '10000',
    target_id: stfd.id.to_s,
    target_name: 'STFD',
    beneficiary_id: child1.id.to_s
  )
).update

#####
custom_dt = create(:custom_disposition_template, estate: @estate, review: @review, from: @trust, step: 'SECOND_DEATH', first_death_in_scenario_id: @spouse.id)

spousal_pool = @estate.synthetic_trusts.create!(
  person: @estate.person,
  kind: :spousal_pool,
  parent_document: @trust,
  created_upon_death: :before_second_spouse
)

create(:remainder_to_trust, estate:, review:, from: @pourover_will, remainder_trust_id: @revocable_trust.id)
```
