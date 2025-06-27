# "Polymorphic" Joins

```ruby
class Asset < ApplicationRecord
  # eagerly loading two different polymorphic associations,
  # one via direct_owner and anotehr via asset_account.owner
  scope :with_owner, -> { includes(:direct_owner, { asset_account: :owner }) }
end
```

Note that because `direct_owner` here is a polymorphic association, `includes(:direct_owner)` won't perform a true SQL join under the hood. Rails will first load the record(s), then issue separate queries for the associated polymorphic models. This is usually fine if your goal is to prevent N+1 queries, but be aware that you can't directly add WHERE clauses on a polymorphic relationship using `joins` in quite the same way.
