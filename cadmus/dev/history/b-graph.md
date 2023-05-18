---
layout: page
title: Cadmus Development
subtitle: History - Backend Graph Upgrade
---

## Backend Graph Upgrade

📆 Date: 2023-05-18

### Rationale

The graph subsystem has been further developed, providing new functionalities. To enable these functionalities, your API backend should change services injection for graph.

Before the change, you just injected `IGraphRepository`; now we need to inject also `GraphUpdater`, as this needs to be configured with additional metadata providers. The metadata provider required is the EID metadata provider, which given an item ID finds its EID by looking at a metadatum named eid inside its `MetadataPart`, if any. This is required to ease some graph mapping rules which need access to the item's EID as defined in this way, via metadata, which conforms to the [generic lookup mechanisms](../concepts/lookup.md) in place.

### Affected Products

⚠️ The API libraries (`Cadmus.Api.Services`, `Cadmus.Api.Controllers`) since **version 6.3.0** have been updated to get instances of `GraphUpdater` from the injected services rather than directly instantiating them. So, you must ensure to update your DI configuration in the API startup to get a properly configured `GraphUpdater`. The libraries anyway will fallback to a "standard", unconfigured `GraphUpdater` when the DI mechanism cannot provide one, thus avoiding older API code not using graph to break. So, if using graph ensure that you properly configure DI, or your mappings will silently and partially fail whenever they try to retrieve an item's EID from its metadata part.

- all the API backend startup code using graph.

### Upgrade Path

(1) ensure to update the API dependencies, so that updated libraries using DI will be used.

(2) add this new method to your `Startup.cs` file:

```cs
private void ConfigureIndexServices(IServiceCollection services)
{
    // item index factory provider
    string indexCS = string.Format(
        Configuration.GetConnectionString("Index"),
        Configuration.GetValue<string>("DatabaseNames:Data"));

    services.AddSingleton<IItemIndexFactoryProvider>(_ =>
        new StandardItemIndexFactoryProvider(indexCS));

    // graph repository
    services.AddSingleton<IGraphRepository>(_ =>
    {
        var repository = new MySqlGraphRepository();
        repository.Configure(new SqlOptions
        {
            ConnectionString = indexCS
        });
        return repository;
    });

    // graph updater
    services.AddTransient<GraphUpdater>(provider =>
    {
        IRepositoryProvider rp = provider.GetService<IRepositoryProvider>();
        return new(provider.GetService<IGraphRepository>())
        {
            // we want item-eid as an additional metadatum, derived from
            // eid in the role-less MetadataPart of the item, when present
            MetadataSupplier = new MetadataSupplier()
                .SetCadmusRepository(rp.CreateRepository())
                .AddItemEid()
        };
    });
}
```

(3) in your API `Startup.cs` file, locate the `ConfigureServices` method, and replace the index configuration:

```cs
// item index factory provider
string indexCS = string.Format(
    Configuration.GetConnectionString("Index"),
    Configuration.GetValue<string>("DatabaseNames:Data"));
services.AddSingleton<IItemIndexFactoryProvider>(_ =>
    new StandardItemIndexFactoryProvider(indexCS));

// graph repository
services.AddSingleton<IGraphRepository>(_ =>
{
    var repository = new MySqlGraphRepository();
    repository.Configure(new SqlOptions
    {
        ConnectionString = indexCS
    });
    return repository;
});
```

with a call to the new method:

```cs
// index and graph
ConfigureIndexServices(services);
```
