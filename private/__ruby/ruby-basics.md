# Basics - Ruby

- Ruby does not have a module system. There is no such thing as `import`. Any code that is loaded in the current process is always available.
- When declaring a function, adding a parameter as `param:` as opposed to `param: 'some default value'` implies that the param is a required parameter.

## Syntax - `try`, `<<`, lambda, `||=`, BigDecimal

```ruby
account.person.memberships << Membership.create!(...)

# lambda
def foo
  f = ->(x) { x }
  g = lambda do |x|
        x
      end
  f.call(5)
end
```

## `.member?`

```ruby
array = [1, 2, 3, 4]
array.member? 1 # true
```

##

```ruby
# include?, exclude?
[1, 2, 3].include?(3) # true
'hello'.exclude?('ell') # false

# %i[ ] - non-interpolated Array of symbols, separated by whitespace
%i[test] # [:test]

###
def my_method(numbers)
  add_two = ->(n) { n + 2 }
  numbers.map(&add_two)
end
my_method([2, 4, 6])

###
Time.now.utc
Random.rand(0..9)
[5, 10].min
'666'.to_s
'666'.is_a?(String)
JSON.parse('{"a":123}')

my_array.grep(Thing)
```
