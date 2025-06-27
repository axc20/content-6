### Testing (for Policies/User Access)

```ruby
class MyTest < ActiveSupport::TestCase
  setup do
    org_admin_membership = create(:membership_with_account, is_primary_admin: true, organization:)
    advisor_membership = create(:membership_with_account, :advisor, organization:)
    estate_specialist_membership = create(:membership_with_account, :advisor, organization:)
    document_reviewer_membership = create(:membership_with_account, :document_reviewer, organization:)

    @org_admin = org_admin_membership.person.account
    @advisor = advisor_membership.person.account
    @estate_specialist = estate_specialist_membership.person.account
    @estate_specialist.update!(is_estate_specialist: true)
    @document_reviewer = document_reviewer_membership.person.account

    client_membership = create(:membership_with_account, :client, organization:)
    estate = client_membership.person.estate
    @client = client_membership.person.account
  end
end
```

## Estate

- Consider single, married, and deceased spouse (widowed) cases

```ruby
Estate.find(9).waterfall_diagram(
  starting_nodes: [{
    starting_node_type: 'Trust',
    starting_node_id: '1'
  }],
  assumed_first_death_id: '11'
)

Estate.find(1).waterfall_debug(assumed_first_death_id: '101')
```
