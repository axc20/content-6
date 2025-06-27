# Funding

```ruby
module DispositionTemplateFunding
  include DocumentAbstraction::DistributionsHelper

  def funding_summary; end
  def other_distribution_funding
    root_dt = topmost_parent_template
    distribution_dts = root_dt.child_templates.where.not(id:).to_a
    # ...
    sort_distributions
    # ...
    funding_options_by_dt_id = FundingOption
      .where(from_disposition_template_id: distribution_dt_ids)
      .group_by(&:from_disposition_template_id)
    distribution_rules = DistributionRule.where(from_disposition_template: distribution_dt_ids).to_a
    # ...
  end
end
```
