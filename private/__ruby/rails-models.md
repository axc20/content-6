## Unambiguous query (joining tables)

```ruby
# both tables have `created_at`
# to specifiy a condition on joined tables with same column name:
Train.joins(:schedule).where(trains: { created_at: DateTime.now - 7.days })

# or
time_range = DateTime.now - 7.days
Train.joins(:schedule).where('trains.created_at' => time_range)

# could also use scopes to accomplish same thing, perhaps in cleaner way
# may be better to implement the WHERE clause on Train before joining
```

## `where.not`, scopes, merge

```ruby
Foo.includes(:bar).where.not('bars.id' => nil)
Foo.includes(:bar).where.not(bars: { id: nil })

# when working with scopes between tables,
# can leverage merge to use existing scopes more easily
Foo.includes(:bar).merge(Bar.where.not(id: nil))

# since includes does not always choose a join strategy,
# you should use references here as well
# otherwise you may end up with invalid SQL
Foo.includes(:bar).references(:bar).merge(Bar.where.not(id: nil))
```
