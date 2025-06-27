## Create data (using factories)

```ruby
# kind: :irrevocable :revocable_joint
@client_rev_trust = create(:trust, estate:, owner: estate.person, created_upon_death: nil, review: @review, kind: :revocable)
client_rev_trust_dt_1 = create(
  # :all_to_survivor_disposition_template
  # :marita_bypass_state_exemption_disposition_template
  # :pot_disposition_template
  # :separate_trust_shares_disposition_template
  :marital_bypass_disposition_template,
  estate:,
  review: @review,
  first_death_in_scenario_id: estate.person.id,
  from: @client_rev_trust,
  # 'SECOND_DEATH'
  step: 'FIRST_DEATH'
)

# :slat
create(:trust, estate:, kind: :revocable, situs: 'AK', owner: client, execute_At: Time.zone.now - 3.years)
# :power_of_attorney, :pourover_will, :will, :codicil
create(:estate_document, estate:, kind: :power_of_attorney, situs: 'AK', person: spouse)

##### estate documents, trusts, disposition templates #####
pourover_will = create(:estate_document, estate:, person: estate.person, kind: :pourover_will)
remainder_trust = create(:trust, estate:, owner: estate.person, created_upon_death: nil, review:, kind: :revocable)
disposition_template.update(remainder_trust_id: remainder_trust.id)
pourover_will.distribution_rules.first.death

estate.update(first_spouse_to_die_id: estate.person.spouse.id)
```
