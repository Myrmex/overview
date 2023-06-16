---
layout: page
title: Cadmus Development
subtitle: History - RDBMS Refactoring
---

## RDBMS Refactoring

📆 Date: 2023-06-16

### Rationale

This refactoring affects the way relational databases are used for index and graph. Before it, we had a single MySql database used for both index and graph. With this update instead, we have **two distinct databases**, and the default underlying RDBMS provider is no more MySql, but **PostgreSQL** (though you can still use MySql if you prefer).

Since its origin, Cadmus has been designed to use several types of RDBMS for its index, even though traditionally using MySql, which is one of the most popular choices. Since MySql is showing glitches when containerized in hardware environments like MacOS with Apple CPUs, its software support is not ideal, open alternatives like MariaDB are not 100% binary compatible, and more extensible choices like PostgreSQL are preferable, especially in a developer perspective, I implemented new libraries to migrate indexes to other providers.

Also, to maximize the codebase shared among libraries targeting different providers the new components were based on **Entity Framework Core**, which offers a higher level of abstraction, and mostly shields from differences in SQL details.

Creating these new libraries also required **renaming fields** so that they can be uniform and have no case-sensitivity issues across different databases. Thus, camel cased field names have been renamed to snake case and so completely lowercased; for instance, `itemId` became `item_id`. Yet, to preserve compatibility with existing databases and code, MySql-based libraries where necessary provide a `legacy` flag, which once set to true keeps the old naming.

Finally, the graph database is now separate from the index database. So, a project named `x` using both index and graph will have these databases:

(a) MongoDB:

- `cadmus-x`: the data database.
- `cadmus-x-auth`: the authentication database.
- `cadmus-x-log`: the log database.

(b) RDBMS:

- `cadmus-x`: the index database.
- `cadmus-x-graph`: the graph database.

### Affected Products

The Cadmus backend core and graph components have no breaking change. They just got some additions:

- `ItemGraphFactory` was added to separate index from graph. You should change your code to use this factory (usually via `IItemGraphFactoryProvider`) rather than using the same factory used for item indexes. The graph methods in the `ItemIndexFactory` dealing with graph have been marked as obsolete, and will be removed in future versions.
- Entity Framework Core-based components have been added to provide new implementations for MySql and PostgreSQL for the index (the same has been done for the Graph components). These components are found in new libraries you should replace for the old ones.

In the previous architecture, the **index** was supported by these libraries:

- `Cadmus.Index.Sql` providing a code base shared among all the SQL-based implementations.
- `Cadmus.Index.MySql` providing MySql-specific code.
- `Cadmus.Index.PgSql` providing PostgreSQL-specific code.

Similarly, the **graph** was supported by:

- `Cadmus.Graph.Sql` providing a code base shared among all the SQL-based implementations.
- `Cadmus.Graph.MySql` providing MySql-specific code.
- `Cadmus.Graph.PgSql` providing PostgreSQL-specific code.

All these SQL-based libraries deal with low-level SQL, eventually (for graph) with a thin abstraction layer by SqlKata.

The new libraries instead rely on Entity Framework Core, and they are named accordingly:

- `Cadmus.Index.Ef`
- `Cadmus.Index.Ef.MySql`
- `Cadmus.Index.Ef.PgSql`
- `Cadmus.Graph.Ef`
- `Cadmus.Graph.Ef.MySql`
- `Cadmus.Graph.Ef.PgSql`

So, in the end the only affected product is the library providing default services to the backend API (`Cadmus.Api.Services`), as it's task is orchestrating all the essential Cadmus services including databases.

### Upgrade Path

Typically you will migrate code and also change the underlying database from MySql to PostgreSQL, though you can also use the new components for MySql.

To migrate your project from SQL-based to EF-based components, and from MySql to PostgreSQL, you must target two backend areas:

- your core area, if any, including models and services. Services here are the components which will be affected, as you need to switch the index and graph components.
- your API area.

#### Core

If your project uses custom models, you typically have a core project with your parts, seeders, and services. For this project:

(1) update all the libraries via NuGet.

(2) remove the MySql-based components used in your service library, replacing them with the EF-based, PostgreSQL counterparts:

- replace `Cadmus.Index.Sql` with `Cadmus.Index.Ef.PgSql`, or just remove it if it is not used (which is often the case).

#### API

The main breaking change in API v.8 is that `Cadmus.Api.Services`, which provides default Cadmus services for the API, has now switched to Entity Framework Core and PostgreSQL. Its legacy counterpart is now named `Cadmus.Api.Services.Legacy`, and can be used for old projects before migrating them.

Change your API as follows:

(1) update all the libraries via NuGet.

(2) remove the MySql-based components if any, replacing them with the EF-based, PostgreSQL counterparts. Usually this is not required, as your project typically uses `Cadmus.Api.Services`, which has already been updated.

- remove `Cadmus.Index.MySql`;
- add `Cadmus.Index.Ef.PgSql` and `Cadmus.Graph.Ef.PgSql`.

(3) in `Startup.cs`, replace `ConfigureIndexServices`, with two separate methods:

```cs
/// <summary>
/// Configures the item index services with the connection string template
/// from <c>ConnectionStrings:Index</c>, whose database name is defined in
/// <c>DatabaseNames:Data</c>.
/// </summary>
/// <param name="services">The services.</param>
private void ConfigureIndexServices(IServiceCollection services)
{
    // item index factory provider (from ConnectionStrings/Index)
    string cs = string.Format(Configuration.GetConnectionString("Index")!,
        Configuration.GetValue<string>("DatabaseNames:Data"));

    services.AddSingleton<IItemIndexFactoryProvider>(_ =>
        new StandardItemIndexFactoryProvider(cs));
}

/// <summary>
/// Configures the item graph services with the connection string template
/// from <c>ConnectionStrings:Graph</c> (falling back to <c>:Index</c> if
/// not found), whose database name is defined in <c>DatabaseNames:Data</c>
/// plus suffix <c>-graph</c>.
/// </summary>
/// <param name="services">The services.</param>
private void ConfigureGraphServices(IServiceCollection services)
{
    string cs = string.Format(Configuration.GetConnectionString("Graph")
        ?? Configuration.GetConnectionString("Index")!,
        Configuration.GetValue<string>("DatabaseNames:Data") + "-graph");

    services.AddSingleton<IItemGraphFactoryProvider>(_ =>
        new StandardItemGraphFactoryProvider(cs));

    services.AddSingleton<IGraphRepository>(_ =>
    {
        var repository = new EfPgSqlGraphRepository();
        repository.Configure(new EfGraphRepositoryOptions
        {
            ConnectionString = cs
        });
        return repository;
    });

    // graph updater
    services.AddTransient<GraphUpdater>(provider =>
    {
        IRepositoryProvider rp = provider.GetService<IRepositoryProvider>()!;
        return new(provider.GetService<IGraphRepository>()!)
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

>Note that you can provide a totally different connection string template for the graph database (named `Graph`). Anyway, if such string does not exist, the template of `Index` will be used. This allows old code work without having to change connection strings settings.

(4) in `ConfigureServices`, ensure to call _both_ these methods:

```cs
// index and graph
ConfigureIndexServices(services);
ConfigureGraphServices(services);
```

(5) in `appsettings.json`, change the **index connection string template** in `ConnectionStrings/Index` from MySql to PostgreSQL:

```json
"Index": "Server=localhost;Database={0};User Id=postgres;Password=postgres;Include Error Detail=True"
```

>If you find a `"DatabaseType": "mysql"` entry in `Indexing`, just remove it. This is a legacy setting, no more in use.

No other change is required, because when the `Graph` connection string template is not defined, the `Index` template is used for it too; the database name will be automatically derived from the data database name suffixed with `-graph`.

(6) in `wwwroot/seed-profile.json`, change the item index reader and writer tags, and the graph repository tag, so that you use the new Entity Framework Core-based libraries:

```json
"index": {
    "writer": {
        "id": "item-index-writer.ef-pg"
    },
    "reader": {
        "id": "item-index-reader.ef-pg"
    }
},
"graph": {
    "repository": {
        "id": "graph-repository.ef-pg"
    }
},
```

(7) of course, if you are using a Docker compose script, replace MySql with PostgreSQL:

```yml
  # PostgreSQL
  cadmus-pgsql:
    image: postgres
    container_name: cadmus-pgsql
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres
      - POSTGRES_DB=postgres
    ports:
      - 5432:5432
    networks:
      - cadmus-network
    # volumes:
      # ensure you have created the var/db/pgsql folder in the host
      # https://hub.docker.com/r/postgis/postgis
      # - /var/db/pgsql:/var/lib/postgresql/data
```

Remember to change the connection string in the API container service, too:

```yml
  cadmus-api:
    image: vedph2020/cadmus-api:8.0.0
    container_name: cadmus-api
    # ... etc.
    environment:
      - CONNECTIONSTRINGS__INDEX=Server=cadmus-pgsql;port=5432;Database={0};User Id=postgres;Password=postgres;Include Error Detail=True
```
