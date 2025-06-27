# Datadog

## Dashboards

```
RUM - Wep App Performance

viewName: /estate/?/builder/?/trust
env: production
loadType: initial_load
sessionType: user
```

## APM (Application Performance Monitoring) Span/Trace Search

```
# GraphQL Queries/Mutations
env:production service:"service-api-backend:production" operation_name:execute.graphql resource_name:GetDeathScenariosDispositionTemplates

env:production service:"service-api-backend:production" resource_name:(GetFeatureEnabledForEstate OR GetEstateFamily OR GetWaterfallDiagram)

# PDF
# Check logs for "Loading URL..." -> Click trace -> Event Attributes -> URL (has reports ID)
env:production service:"service-rendering-pdf-inline:production" operation_name:execute_render resource_name:"execute render" @version:20241206.18.22 status:error
```

## Log Explorer Search

```
# Balance Sheet
service:"Service-API-Backend:Staging" @tags:"BalanceSheet::V2::AssetAccountView"

# PDF
@dd.service:"Service-Rendering-PDF-Inline:Staging" "Failed to load"

# GraphQL Errors
service:"Service-API-Backend:Production" @name:GraphqlController status:error -@named_tags.class:ScenarioBuilderWorker
```
