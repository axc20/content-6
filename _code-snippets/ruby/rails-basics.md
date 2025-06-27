# Rails - basics

- `rails c`

```ruby
Rails.logger.info("...#{something}")
puts '...'

Model.find(1)
Model.find_by(name: 'Bob')
Model.find_by(:name => 'Bob') # hash rocket syntax
Model.where(status: 'draft')
Model.where(id: 1).or(Model.where(id : 2))
Model.all
Model.first
Model.last # Model.last(2)
Model.destroy([1, 2])
Model.where(id: [1, 2, 3]).destroy_all
Model.where(some_condition: true).update_all(some_attribute: 'new_value')

# `in_batches` method divides the records into smaller batches and processes
# them one batch at a time, reducing memory usage and improving performance
# default batch size is 1,000
# can do `in_batches(of: 500)` to adjust batch size
ModelName.in_batches.each do |batch|
  batch.each do |record|
    # perform custom logic or update on each record
    record.update(some_field: 'new_value')
  end
end

Person.find(1).memberships.distinct.pluck(:organization_id)

record = Model.find(1)
record.some_attribute = true
record.save!
```

## Action View

```ruby
# sanitize HTML content
raw_html = "<script>alert('Hello, world!');</script><p>This is some <b>bold</b> text.</p>"
sanitized_html = ActionView::Base.full_sanitizer.sanitize(raw_html)
```

## Modules

```ruby
module Waterfalls
  module Edges
    class DistributionRuleEdge < ModelEdge
      include Modules::ClassDistributionable
    end
  end
end

module Waterfalls
  module Modules
    module ClassDistributionable
      # .blank?
      # .flatten
    end
  end
end
```
