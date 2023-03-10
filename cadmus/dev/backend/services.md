---
layout: page
title: Creating Services
subtitle: "Cadmus Backend Development"
---

📌 Create a repository provider service to provide models, and a part seeder factory provider to provide part or fragment seeders when using mock data.

1. [core](core.md)
2. [parts](parts.md)
3. [part seeders](part-seeders.md)
4. [fragments](fragments.md)
5. [fragment seeders](fragment-seeders.md)
6. **services**
7. [API](api.md)

## Creating Services

If the models you created are part of a specific project, you will typically require to collect all the required models together using a couple of services. These services will then be consumed by the [API layer](api.md).

>If instead you are just creating models to be consumed by several different projects, you will not need such services, as every project will have its own services collecting the parts/fragments it requires.

Add these packages to the services project (updating version numbers as required):

```xml
<ItemGroup>
  <PackageReference Include="Cadmus.Core" Version="5.0.0" />
  <PackageReference Include="Cadmus.General.Parts" Version="3.0.0" />
  <PackageReference Include="Cadmus.Index.Sql" Version="5.0.0" />
  <PackageReference Include="Cadmus.Mongo" Version="5.0.0" />
  <PackageReference Include="Cadmus.Philology.Parts" Version="5.0.0" />
  <PackageReference Include="Cadmus.Seed.General.Parts" Version="3.0.0" />
  <PackageReference Include="Cadmus.Seed.Philology.Parts" Version="5.0.0" />
  <PackageReference Include="Fusi.Microsoft.Extensions.Configuration.InMemoryJson" Version="2.0.0" />
</ItemGroup>
```

## Repository Provider

Add a `<PRJ>RepositoryProvider` class, using this template (the only part which requires customization is the constructor):

```cs
using System.Reflection;
using Cadmus.Core;
using Cadmus.Core.Config;
using Cadmus.Core.Storage;
using Cadmus.Mongo;
using Cadmus.General.Parts;
using Cadmus.Philology.Parts;

namespace Cadmus.__PRJ__.Services;

/// <summary>
/// Cadmus __PRJ__ repository provider.
/// </summary>
/// <seealso cref="IRepositoryProvider" />
public sealed class __PRJ__RepositoryProvider : IRepositoryProvider
{
    private readonly IPartTypeProvider _partTypeProvider;

    /// <summary>
    /// The connection string.
    /// </summary>
    public string ConnectionString { get; set; }

    /// <summary>
    /// Initializes a new instance of the <see cref="__PRJ__RepositoryProvider"/>
    /// class.
    /// </summary>
    public __PRJ__RepositoryProvider()
    {
        ConnectionString = "";
        TagAttributeToTypeMap map = new();
        map.Add(new[]
        {
            // Cadmus.General.Parts
            typeof(NotePart).GetTypeInfo().Assembly,
            // Cadmus.Philology.Parts
            typeof(ApparatusLayerFragment).GetTypeInfo().Assembly,
            // Cadmus.__PRJ__.Parts
            // typeof(MYPART).GetTypeInfo().Assembly,
        });

        _partTypeProvider = new StandardPartTypeProvider(map);
    }

    /// <summary>
    /// Gets the part type provider.
    /// </summary>
    /// <returns>part type provider</returns>
    public IPartTypeProvider GetPartTypeProvider()
    {
        return _partTypeProvider;
    }

    /// <summary>
    /// Creates a Cadmus repository.
    /// </summary>
    /// <returns>repository</returns>
    public ICadmusRepository CreateRepository()
    {
        // create the repository (no need to use container here)
        MongoCadmusRepository repository = new(_partTypeProvider,
                new StandardItemSortKeyBuilder());

        repository.Configure(new MongoCadmusRepositoryOptions
        {
            ConnectionString = ConnectionString ??
            throw new InvalidOperationException(
                "No connection string set for IRepositoryProvider implementation")
        });

        return repository;
    }
}
```

## Part Seeder Factory Provider

Add a `<PRJ>PartSeederFactoryProvider` class.

The **current** template after the [component factory update](../history.md#2023-02-01---backend-infrastructure-upgrade) is:

```cs
using Cadmus.Core.Config;
using Cadmus.Seed;
using Cadmus.Seed.General.Parts;
using Cadmus.Seed.Philology.Parts;
using Cadmus.Seed.__PRJ__.Parts;
using Cadmus.Seed.Tgr.Parts.Grammar;
using Fusi.Microsoft.Extensions.Configuration.InMemoryJson;
using Microsoft.Extensions.Hosting;
using System;
using System.Reflection;

namespace Cadmus.__PRJ__.Services;

/// <summary>
/// __PRJ__ part seeders provider.
/// </summary>
/// <seealso cref="IPartSeederFactoryProvider" />
public sealed class __PRJ__PartSeederFactoryProvider :
    IPartSeederFactoryProvider
{
    private static IHost GetHost(string config)
    {
        // build the tags to types map for parts/fragments
        Assembly[] seedAssemblies = new[]
        {
            // TODO: include all the assemblies required by your project
            // Cadmus.Seed.General.Parts
            typeof(NotePartSeeder).Assembly,
            // Cadmus.Seed.Philology.Parts
            typeof(ApparatusLayerFragmentSeeder).Assembly,
            // Cadmus.Seed.__PRJ__.Parts
            // typeof(MYSEEDER).GetTypeInfo().Assembly,
        };
        TagAttributeToTypeMap map = new();
        map.Add(seedAssemblies);

        return new HostBuilder()
            .ConfigureServices((hostContext, services) =>
            {
                PartSeederFactory.ConfigureServices(services,
                    new StandardPartTypeProvider(map),
                    seedAssemblies);
            })
            // extension method from Fusi library
            .AddInMemoryJson(config)
            .Build();
    }

    /// <summary>
    /// Gets the part/fragment seeders factory.
    /// </summary>
    /// <param name="profile">The profile.</param>
    /// <returns>Factory.</returns>
    /// <exception cref="ArgumentNullException">profile</exception>
    public PartSeederFactory GetFactory(string profile)
    {
        if (profile == null) throw new ArgumentNullException(nameof(profile));

        return new PartSeederFactory(GetHost(profile));
    }
}
```

This is the **old** template, still using `Fusi.Tools.Config`:

```cs
using Cadmus.Core.Config;
using Cadmus.Seed;
using Cadmus.Seed.General.Parts;
using Cadmus.Seed.__PRJ__.Parts;
using Cadmus.Seed.Philology.Parts;
using Fusi.Microsoft.Extensions.Configuration.InMemoryJson;
using SimpleInjector;
using System;
using System.Reflection;

namespace Cadmus.__PRJ__.Services
{
    /// <summary>
    /// __PRJ__ part seeders provider.
    /// </summary>
    /// <seealso cref="IPartSeederFactoryProvider" />
    public sealed class __PRJ__PartSeederFactoryProvider :
        IPartSeederFactoryProvider
    {
        /// <summary>
        /// Gets the part/fragment seeders factory.
        /// </summary>
        /// <param name="profile">The profile.</param>
        /// <returns>Factory.</returns>
        /// <exception cref="ArgumentNullException">profile</exception>
        public PartSeederFactory GetFactory(string profile)
        {
            if (profile == null) throw new ArgumentNullException(nameof(profile));

            // build the tags to types map for parts/fragments
            Assembly[] seedAssemblies = new[]
            {
                // TODO: include here all the assemblies required by your prj
                // Cadmus.Seed.General.Parts
                typeof(NotePartSeeder).Assembly,
                // Cadmus.Seed.Philology.Parts
                typeof(ApparatusLayerFragmentSeeder).Assembly,
                // Cadmus.Seed.__PRJ__.Parts
                // typeof(MYSEEDER).GetTypeInfo().Assembly,
            };
            TagAttributeToTypeMap map = new();
            map.Add(seedAssemblies);

            // build the container for seeders
            Container container = new();
            PartSeederFactory.ConfigureServices(
                container,
                new StandardPartTypeProvider(map),
                seedAssemblies);

            container.Verify();

            // load seed configuration
            IConfigurationBuilder builder = new ConfigurationBuilder()
                .AddInMemoryJson(profile);
            var configuration = builder.Build();

            return new PartSeederFactory(container, configuration);
        }
    }
}
```

🏠 [developer's home](../toc.md)

▶️ next: [API](api.md)
