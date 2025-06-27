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

## lambdas

- useful when you need to encapsulate a block of code in a reusable and callable object
  - anonymous functions: small, self-contained blocks of code that can be passed around as objects
  - callbacks and event handlers in event-driven programming: pass as argument to a method to be executed when a specific event or condition occurs
  - functional programming: work with functions as first-class citizens
  - iteration and data manipulation: use with iteration methods like `each`, `map`, `select`, `reduce`
  - closures: lambdas capture surrounding environment including local variables and constants
  - delayed execution: defer execution until a later time, useful for lazy evaluation or delaying expensive computations until they are needed
  - DSL (domain-specific languages): useful in creating internal DSLs to solve specific problems within a particular domain

```ruby
outer_lambda = lambda do
  puts 'outer lambda executing'

  inner_lambda = lambda do
    puts 'inner lambda executing'
  end

  inner_lambda.call
end

outer_lambda.call
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
