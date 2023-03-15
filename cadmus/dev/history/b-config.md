---
layout: page
title: Cadmus Development
subtitle: History - Backend Infrastructure Upgrade
---

- [Backend Infrastructure Upgrade](#backend-infrastructure-upgrade)
  - [Rationale](#rationale)
  - [Affected Products](#affected-products)
  - [Upgrade Path](#upgrade-path)

## Backend Infrastructure Upgrade

📆 Date: 2023-02-01.

### Rationale

As many other projects of mine requiring to instantiate and configure a number of pluggable components, Cadmus has always been relying on my library `Fusi.Tools.Config` for this purpose. In turn, this library depends on [SimpleInjector](https://simpleinjector.org/), because when it was first designed there were no Microsoft-based alternatives to it. Also, JSON-based configurations served from sources other than files were handled via another library of mine, `Fusi.Microsoft.Extensions.Configuration.InMemoryJson`, which in turn depends on Netwonsoft [Json.NET](https://www.newtonsoft.com/json).

With time, Microsoft-based open source components were added or improved in many tangent areas, like a high performance JSON library, or an improved DI ecosystem. This also called for more integration in this ecosystem, e.g. to allow using the standard DI mechanism provided by standard MS technologies.

So, I developed a new version of `Fusi.Tools.Config`, named `Fusi.Tools.Configuration`, with these purposes:

- remove third party dependencies.
- allow DI in the context of MS-based solutions ([ServiceCollection](https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.dependencyinjection.servicecollection?view=dotnet-plat-ext-7.0)).
- extend the original library capabilities.

>⚠️ Since version 3.1.0, which was updated in compliance with the new library, `Fusi.Tools` moved its configuration-related components (`TagAttribute` and `IConfigurable<T>`) to `Fusi.Tools.Config`, so that we can continue to use `Fusi.Tools` without overlapping some configuration-related code from `Fusi.Tools.Config` or `Fusi.Tools.Configuration`. So, pay attention to the namespace of these components in your code: ensure to update all your code to 3.1.0 or later, so that these components get not accidentally drawn from `Fusi.Tools` rather than from the new `Fusi.Tools.Configuration` (or the old `Fusi.Tools.Config`).

While the new library provides all the functions of the old one, plus many new ones, it is of course no more compatible with it, as its dependencies have changed. The new components factory no more uses `SimpleInjector`'s container, but rather adopts the [IHost](https://learn.microsoft.com/en-us/dotnet/core/extensions/generic-host) paradigm, widely used in MS technologies like ASP.NET, but equally applicable to libraries, console apps, etc.

For instance, this is how we can build such a host:

```cs
private static IHost GetHost(string config)
{
    return new HostBuilder()
        .ConfigureServices((hostContext, services) =>
        {
            PythiaFactory.ConfigureServices(services, new[]
            {
                // Corpus.Core.Plugin
                typeof(StandardDocSortKeyBuilder).Assembly,
                // Pythia.Core.Plugin
                typeof(StandardTokenizer).Assembly,
                // Pythia.Udp.Plugin
                typeof(UdpTokenFilter).Assembly,
                // Pythia.Xlsx.Plugin
                typeof(FsExcelAttributeParser).Assembly,
                // Pythia.Sql.PgSql
                typeof(PgSqlTextRetriever).Assembly
            });
        })
        // extension method from Fusi library
        .AddInMemoryJson(config)
        .Build();
}
```

Also, when your custom factory implies the constant usage of some assemblies and interfaces, a typical helper convention is to provide a static `ConfigureServices` method, like the one used in the above sample, e.g.:

```cs
public static void ConfigureServices(IServiceCollection services,
    params Assembly[] additionalAssemblies)
{
    if (services is null) throw new ArgumentNullException(nameof(services));

    // a singleton service
    services.AddSingleton<UniData>();

    // assemblies used as basic components sources
    Assembly[] assemblies = new[]
    {
        // Pythia.Core
        typeof(PythiaFactory).Assembly,
    };
    // eventual additional assemblies
    if (additionalAssemblies?.Length > 0)
        assemblies = assemblies.Concat(additionalAssemblies).ToArray();

    // register the components for the specified interfaces
    // from all the assemblies
    foreach (Type it in new[]
    {
        typeof(IAttributeParser),
        typeof(IDocSortKeyBuilder),
        typeof(IDocDateValueCalculator),
        typeof(IStructureValueFilter),
        typeof(IStructureParser),
        typeof(ILiteralFilter),
        typeof(ITextFilter),
        typeof(ITokenizer),
        typeof(ITokenFilter),
        typeof(ISourceCollector),
        typeof(ITextRetriever),
        typeof(ITextMapper),
        typeof(ITextPicker),
        typeof(ITextRenderer)
    })
    {
        foreach (Type t in GetAssemblyConcreteTypes(assemblies, it))
        {
            services.AddTransient(it, t);
        }
    }
}
```

If your components need some options which should not be placed in this document, e.g. a connection string, the factory provides an override mechanism to supply (override) them via code. In this case, the POCO options object should include that property (usually as a nullable property), and you should derive your own factory from `ComponentFactory`, like in this example:

```cs
internal sealed class AppComponentFactory : ComponentFactory
{
    /// <summary>
    /// The name of the connection string property to be supplied
    /// in POCO option objects (<c>ConnectionString</c>).
    /// </summary>
    public const string CONNECTION_STRING_NAME = "ConnectionString";

    /// <summary>
    /// The optional general connection string to supply to any component
    /// requiring an option named <see cref="CONNECTION_STRING_NAME"/>
    /// (=<c>ConnectionString</c>), when this option is not specified
    /// in its configuration.
    /// </summary>
    public string? ConnectionString { get; set; }

    public AppComponentFactory(IHost host) : base(host)
    {
    }

    protected override void OverrideOptions(object options,
        IConfigurationSection? section)
    {
        Type optionType = options.GetType();

        // if we have a default connection AND the options type
        // has a ConnectionString property, see if we should supply a value
        // for it
        PropertyInfo? property;
        if (ConnectionString != null &&
            (property = optionType.GetProperty(CONNECTION_STRING_NAME)) != null)
        {
            // here we can safely discard the returned object as it will
            // be equal to the input options, which is not null
            SupplyProperty(optionType, property, options, ConnectionString);
        }
    }
}
```

### Affected Products

The change impacted a number of backend products, in this order:

1. `Cadmus.Core`.
2. `Cadmus.Graph` for the graph used in the index.
3. `Cadmus.Migration` for the preview functions.
4. all the Cadmus parts (`Cadmus.General.Parts`, `Cadmus.Philology.Parts`, etc.), as they need to reference the correct implementation of `TagAttribute`.
5. Cadmus API.
6. all the backend components of generic (like `Codicology`, `Geography`, `Epigraphy`) or specific Cadmus projects.

Cadmus bricks were not affected, as they have minimal dependencies.

### Upgrade Path

This change impacts only backend custom parts/fragments and their services. All the other components should just update their libraries.

In your **project-specific library**, typically you just have to:

1. update all the libraries in your projects. Make sure to update `Fusi.Tools`, and replace `Fusi.Tools.Config` with `Fusi.Tools.Configuration` everywhere.
2. replace namespace reference `Fusi.Tools.Config` with `Fusi.Tools.Configuration`. This ensures that your pluggable components (in most cases parts/fragments) tagged with `TagAttribute` use the implementation found in `Fusi.Tools.Configuration`, rather than that from `Fusi.Tools` (or `Fusi.Tools.Config`). This is easy, as once removed any references to the old library you get a compile error wherever you need to apply the replacement.
3. replace the part seeder factory provider implementation following its [new template](../backend/services.md#part-seeder-factory-provider).
4. in your test library (when present), replace the seeder helper `GetFactory` method following its [new template](../backend/part-seeders.md#test-helper).

>The part repository provider needs no changes.

In your **projec-specific API**, you just have to update all the libraries.

🏠 [developer's home](../toc.md)
