# Setup

## Factories

```ruby
@estate = create(:estate)
@review = create(:review, estate: @estate)
@spouse = create(:person, estate: @estate)
Relation.create!(from: @estate.person, to: @spouse, kind: Relation::SPOUSE)
@joint_owner = @estate.person.joint_owners.find_by(kind: Types::JointOwnerKindEnum::TENANTS_IN_COMMON, person_refs: [@estate.person.id, @spouse.id])

@joint_revocable = @estate.trusts.create!(name: 'Joint Trust Revocable', kind: :revocable_joint, person: @estate.person, owner: @joint_owner)
@irrev = @estate.trusts.create!(name: 'Irrevocable Trust', kind: :irrevocable, person: @estate.person, owner: @estate.person)

pot_trusts = @joint_revocable.synthetic_trusts.select(&:pot_trust?)
pot_1, pot_2 = pot_trusts.partition { |t| t.created_upon_death == 'first_spouse' }

spouse_slat = create(:trust, estate:, name: 'Spouse Slat Trust', owner: spouse, created_upon_death: nil, review: nil, kind: :slat)
create(:life_insurance, estate:, owner: spouse_slat, insured: spouse, policy_kind: Types::LifeInsuranceKindEnum::WHOLE, net_death_benefit: 1_000_000_00, csv: 2_000_000_00)

# EstateSharedExamples
# metadata[:test_year], metadata[:married], metadata[:joint]
estate.person.update(state_exemption_used: 0, federal_exemption_used: 0)
@revocable_trust = estate.trusts.create!(name: '...', kind: :revocable, person: estate.person)
create(:asset, estate:, balance: 10_000_000_00, direct_owner: @revocable_trust)
@pourover_will = estate.estate_documents.create!(name: '...', kind: :pourover_will, person: estate.person)
@pourover_will.distribution_rules.create!(to: @revocable_trust, kind: :remainder, tax_kind: :bequest, estate:, review:, death: 'first_spouse')
```
