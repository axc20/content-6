### example

```ruby
account_to_portfolio_id_mapping = {
  'account1' => [1, 2, 3],
  'account2' => 3,
  'account3' => [4, 5, 6]
}

# values -> [[1, 2, 3], '3', [4, 5, 6]]
# flat_map(&:to_a) -> [1, 2, 3, 3, 4, 5, 6]
# to_a converts to an array
# uniq -> [1, 2, 3, 4, 5, 6]
output = account_to_portfolio_id_mapping.values.flat_map(&:to_a).uniq
```

## case, when

```ruby
fruit = 'apple'
color = case fruit
        when 'apple'
          'red'
        when 'banana'
          'yellow'
        when 'kiwi'
          'green'
        else
          'unknown'
        end
color = case fruit
        when 'apple' then 'red'
        when 'banana' then 'yellow'
        when 'kiwi' then 'green'
        else 'unknown'
        end
```

## Dates

```ruby
require 'date'

date = Date.today
datetime = date.to_datetime
# "2023-07-22T00:00:00+00:00" (+00:00 is timezone offset in UTC)
iso8601_format = datetime.iso8601
```
