## Types with arguments (for queries with arguments)

```ruby
# app/graphql/types/estate_type.rb
class EstateType < Types::BaseObject
  field :subtrusts_for_death_scenario, [Types::TrustType], null: false do
    argument :death_step, String, required: true
    argument :document_id, ID, required: true
    argument :document_type, String, required: true
    argument :first_to_die_id, ID, required: false
  end
end

# app/models/estate.rb
class Estate < ApplicationRecord
  include DeathScenariosServices
end

# app/models/concerns/death_scenarios_services.rb
module DeathScenariosServices
  extend ActiveSupportConcern
  def subtrusts_for_death_scenario(death_step:, document_id:, document_type:, first_to_die_id: nil)
  end
end
```
