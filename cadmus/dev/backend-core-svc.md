---
layout: page
title: Creating Services
subtitle: "Cadmus Backend Development"
---

1. add these packages to the services project (updating version numbers as required):

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

2. add a `<PRJ>RepositoryProvider` class, using this template (the only part which requires customization is the constructor):

```cs
using System;
using System.Reflection;
using Cadmus.Core;
using Cadmus.Core.Config;
using Cadmus.Core.Storage;
using Cadmus.Mongo;
using Cadmus.Gisarc.Parts;
using Cadmus.General.Parts;
using Cadmus.Philology.Parts;

namespace Cadmus.__PRJ__.Services
{
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
        /// <param name="configuration">The configuration.</param>
        /// <exception cref="ArgumentNullException">configuration</exception>
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
}
```

3. add a `<PRJ>PartSeederFactoryProvider` class, following this template:

```cs
using Cadmus.Core.Config;
using Cadmus.Seed;
using Cadmus.Seed.General.Parts;
using Cadmus.Seed.__PRJ__.Parts;
using Cadmus.Seed.Philology.Parts;
using Fusi.Microsoft.Extensions.Configuration.InMemoryJson;
using Microsoft.Extensions.Configuration;
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
