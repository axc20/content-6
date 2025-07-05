### Destructuring

```ruby
def request_args
  # `||=`operator does not change value if truthy
  # but if falsy, the operator assigns the value.
  # Used for memoization - a way to "cache" the value
  # so that the hash is only created once.
  @request_args ||= {
    url: 'http://example.com',
    token: 'token'
  }
end

# `values_at` called on hash returns an array containing
# values corresponding to the keys passed as arguments
url, token = request_args.values_at(:url, :token)
```

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

## rand

```ruby
rand(1...1_000)
```

## Dates

```ruby
require 'date'

date = Date.today
datetime = date.to_datetime
# "2023-07-22T00:00:00+00:00" (+00:00 is timezone offset in UTC)
iso8601_format = datetime.iso8601
```

## Module imports

```ruby
# my_module.rb
module MyModule
  def my_method; end
end

# another_file.rb
require_relative 'my_module'

include MyModule
my_method

# my_class.rb
def MyClass
  def my_method; end
end

# yet_another_file.rb
require_relative 'my_class'

obj = MyClass.new
obj.my_method
```
