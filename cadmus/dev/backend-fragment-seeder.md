---
layout: page
title: Fragment Seeders
subtitle: "Cadmus Development"
---

## Fragment Seeder Template

Add a `<NAME>LayerFragmentSeeder.cs` for the seeder, using this template (replace `__NAME__` with the fragment name, using the proper case, and adjust the namespace):

```cs
using Bogus;
using Cadmus.Core;
using Cadmus.Core.Layers;
using Fusi.Tools.Config;
using System;

namespace Cadmus.Seed.Parts.Layers
{
    /// <summary>
    /// Seeder for <see cref="__NAME__LayerFragment"/>'s.
    /// Tag: <c>seed.fr.it.vedph.__PRJ__.__NAME__</c>.
    /// </summary>
    /// <seealso cref="FragmentSeederBase" />
    /// <seealso cref="IConfigurable{__NAME__LayerFragmentSeederOptions}" />
    [Tag("seed.fr.it.vedph.__PRJ__.__NAME__")]
    public sealed class __NAME__LayerFragmentSeeder : FragmentSeederBase,
        IConfigurable<__NAME__LayerFragmentSeederOptions>
    {
        private __NAME__LayerFragmentSeederOptions _options;

        /// <summary>
        /// Gets the type of the fragment.
        /// </summary>
        /// <returns>Type.</returns>
        public override Type GetFragmentType() => typeof(__NAME__LayerFragment);

        /// <summary>
        /// Configures the object with the specified options.
        /// </summary>
        /// <param name="options">The options.</param>
        public void Configure(__NAME__LayerFragmentSeederOptions options)
        {
            _options = options;
        }

        /// <summary>
        /// Creates and seeds a new part.
        /// </summary>
        /// <param name="item">The item this part should belong to.</param>
        /// <param name="location">The location.</param>
        /// <param name="baseText">The base text.</param>
        /// <returns>A new fragment.</returns>
        /// <exception cref="ArgumentNullException">location or
        /// baseText</exception>
        public override ITextLayerFragment GetFragment(
            IItem item, string location, string baseText)
        {
            if (location == null)
                throw new ArgumentNullException(nameof(location));
            if (baseText == null)
                throw new ArgumentNullException(nameof(baseText));

            // TODO: use faker to create fragment object, like:
            // return new Faker<__NAME__LayerFragment>()
            //     .RuleFor(fr => fr.Location, location)
            //     .RuleFor(fr => fr.Text, f => f.Lorem.Sentences())
            //     .RuleFor(fr => fr.Tag,
            //         f => _options.Tags?.Length > 0
            //         ? f.PickRandom(_options.Tags) : null)
            //     .Generate();
        }
    }

    /// <summary>
    /// Options for <see cref="__NAME__LayerFragmentSeeder"/>.
    /// </summary>
    public sealed class __NAME__LayerFragmentSeederOptions
    {
        // TODO: set options
    }
}
```

## Fragment Seeder Test Template

```cs
using Cadmus.Core;
using Cadmus.Core.Layers;
using Fusi.Tools.Config;
using System;
using System.Reflection;
using Xunit;

// ...

public sealed class __NAME__LayerFragmentSeederTest
{
    private static readonly PartSeederFactory _factory;
    private static readonly SeedOptions _seedOptions;
    private static readonly IItem _item;

    static __NAME__LayerFragmentSeederTest()
    {
        _factory = TestHelper.GetFactory();
        _seedOptions = _factory.GetSeedOptions();
        _item = _factory.GetItemSeeder().GetItem(1, "facet");
    }

    [Fact]
    public void TypeHasTagAttribute()
    {
        Type t = typeof(__NAME__LayerFragmentSeeder);
        TagAttribute attr = t.GetTypeInfo().GetCustomAttribute<TagAttribute>();
        Assert.NotNull(attr);
        Assert.Equal("seed.fr.it.vedph.__PRJ__.__NAME__", attr.Tag);
    }

    [Fact]
    public void GetFragmentType_Ok()
    {
        __NAME__LayerFragmentSeeder seeder = new __NAME__LayerFragmentSeeder();
        Assert.Equal(typeof(__NAME__LayerFragment), seeder.GetFragmentType());
    }

    [Fact]
    public void Seed_WithOptions_Ok()
    {
        __NAME__LayerFragmentSeeder seeder = new __NAME__LayerFragmentSeeder();
        seeder.SetSeedOptions(_seedOptions);
        seeder.Configure(new __NAME__LayerFragmentSeederOptions
        {
            // TODO: your seeder options properties here...
            // e.g.:
            // Tags = new[]
            // {
            //     "battle",
            //     "priesthood",
            //     "consulship"
            // }
        });

        ITextLayerFragment fragment = seeder.GetFragment(_item, "1.1", "alpha");

        Assert.NotNull(fragment);

        __NAME__LayerFragment fr = fragment as __NAME__LayerFragment;
        Assert.NotNull(fr);

        Assert.Equal("1.1", fr.Location);
        // TODO other assertions...
    }
}
```

For test helper and its infrastructure see about [adding parts](adding-parts.md#test-helper).
