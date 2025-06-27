## Estates, Documents

```ruby
Review.last.notes.create!(body: 'Hello World', person_id: 18, estate_id: 15)

DocumentPackage.find(3).estate.id
Estate.find(73).charitables.destroy_all
Estate.find(73).people.pluck(:name, :id)
```

## Waterfall

```ruby
EstateDocument.find(95).distribution_rules.first.update(death: 'second_spouse')
EstateDocument.find(96).distribution_rules.first.update(death: 'first_spouse')
```

## Balance Sheet - Assets

```ruby
Asset.asset_types[:cash_and_equivalents]
```
