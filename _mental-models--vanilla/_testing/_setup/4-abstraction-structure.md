# Abstraction Structure

```ruby
# This is a more verbose way
DocumentAbstraction::DistributionsService.new(
  dt,
  DocumentAbstraction::Types::DistributionDetailsParams.new(
    amount_kind: 'remainder',
    target_id: node_id_mapping[item[:target].to_sym].id.to_s
  )
).update

source_rev_to_trust_distribution_details = revocable_trust.disposition_templates
  .where(kind: DispositionTemplate.types[:to_trust])
  .map(&:distribution_details)
```

```ruby
# Direct creation of disposition templates
custom_dt = create(
  :custom_disposition_template,
  estate:,
  review:,
  from: trust,
  step: 'FIRST_DEATH',
  first_death_in_scenario: estate.person
)
child_dt_attrs = { estate:, review:, from: trust, parent_template: custom_dt, first_death_in_scenario: estate.person }
marital_dt = create(:marital_disposition_template, **child_dt_attrs)
sts_dt = create(:separate_trust_shares_disposition_template, **child_dt_attrs)

# Outright to charity/person, class distribution, and to trust DTs are distributions with no targets
```
