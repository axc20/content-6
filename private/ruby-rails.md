## Ruby on Rails

```ruby
a, b = "review_abstractions.ggg.5".split('.', 2)

# pluck
Review.where(organization: 7, status: 'insufficient_information').ids
Review.where(organization: 7, status: 'insufficient_information').pluck(:id)

today = Time.now().to_s
ff = ActiveRecord::Base.connection.execute(
  "INSERT INTO feature_flags (name, created_at, updated_at) VALUES ('new_review_ui', '#{today}', '#{today}')"
)

Organization.where("settings -> 'review' ->> 'basic' = ?", 'organization')
Organization.where("settings -> 'review' ->> 'basic' = ? and settings -> 'review' ->> 'ultra' = ?", 'vanilla', 'vanilla').count
Organization.where("settings @> ?", '{"review": {"basic": "organization", "ultra": "organization"}}')

Organization.find(12).members.map { |m| m.person&.account&.email_address }
```

### first_or_create, find_or_initialize_by

```ruby
# if no user with given email exists, create a new user with the provided attributes
# first_or_create cannot be used to update an existing record
user = User.where(email: 'john@example.com').first_or_create.do |u|
  u.name = 'John'
  u.password = '123456'
end

# find the user with given email, or initalize a new user with the provided attributes
user = User.find_or_initialize_by(email: 'john@example.com') do |u|
  u.name = 'John'
  u.password = '123456'
end

# update the user's attributes and save
user.name = 'Johnny'
user.password = '654321'
user.save
```

### Testing

- Factory Bot
  - `create(:organization, :with_parent)` (trait)
  - https://www.codewithjason.com/factory-bot-traits-vs-nested-factories/
  - https://devhints.io/factory_bot

### Model

```ruby
class Organization < ApplicationRecord
  include Vanilla::SearchEncrypted::Model

  def related_orgs
    orgs = Organization.where(id:).or(Organization.where(parent_organization_id: parent_organization.id))
    if parent_organization.present?
      orgs = orgs.or(Organization.where(parent_organization_id: parent_organization.id))
        .or(Organization.where(id: parent_organization.id))
    end
    orgs
  end

  def weps
    weps_ids = Membership.includes(person: :account)
      .where(organization_id: related_orgs, account: { is_estate_specialist: true })
      .pluck('people.id')
    Person.find(weps_ids)
  end
end
```

### Joining Tables

- [Rails includes vs joins](https://medium.com/@swapnilggourshete/rails-includes-vs-joins-9bf3a8ada00)
- [multiple joins with Active Record](https://www.ananunesdasilva.com/posts/multiple-joins-with-activerecord)

```ruby
# ActiveRecord unambiguous query
Train.joins(:schedule).where(trains: { created_at: DateTime.now - 7.days })
# or
time_range = DateTime.now - 7.days
Train.joins(:schedule).where('trains.created_at': time_range)

House.where(city: 'Paris')
  .joins(floors: { rooms: :furnitures })
  .pluck('name', 'floors.id', 'floors.name', 'rooms.name', 'furnitures.name')

Review.left_joins(:organization).where('organization.name': 'XYZ')

# includes
Organization.find(12).people.includes(:account).where(account: { is_estate_specialist: true }).ids
Membership.includes(person: :account)
  .where(organization_id: ids, account: { is_estate_specialist: true })
  .pluck('account.email_address')
```

### Edit a Database Record

```ruby
m = Meme.where(title: 'Balloon frihgtens cat').first
m.title = 'Balloon frightens cat'
m.save
```

### Quote Column Name

```ruby
Review.left_joins(:organization).where("#{ActiveRecord::Base.connection.quote_column_name('reviews')}.#{ActiveRecord::Base.connection.quote_column_name('created_at')} < ?", '2021-01-01')

key = "reviews.id is not null or reviews.created_at"
Review.left_joins(:organization).where("#{key} < ?", '2021-01-01')
Review.left_joins(:organization).where("#{ActiveRecord::Base.connection.quore_column_name(key)} < ?", '2021-01-01')
```
