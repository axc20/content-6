# Joins vs Includes

> "joins for filtering, includes for preloading"

- **Use `joins` for filtering/querying**: If you need to constrain or filter the primary records based on associated records, `.joins` is the right tool (INNER JOIN).
- **Use `includes` to avoid N+1**: If you just need to load associated data ahead of time for display (or other purposes), `.includes` is ideal. This approach avoids executing a new query every time you access the association in a loop or similar scenario.

## `.joins` Example

```ruby
# Return all posts that have comments
# which contain a certain keyword in the content
Post.joins(:comments).where('comments.content ILIKE ?', '%keyword%')
```

Because `.joins` uses an INNER JOIN, you may end up with fewer recrods returned (only recods that have a matching associated record).

## `.includes` Example

```ruby
Post.includes(:comments).each do |post|
  # Accessing post.comments does not fire a new query for each post
  puts post.comments.map(&:content)
end
```
