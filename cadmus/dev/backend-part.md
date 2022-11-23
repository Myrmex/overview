---
layout: page
title: Cadmus Development
subtitle: "Creating Parts"
---

- [Parts](#parts)
  - [Packages](#packages)
  - [Procedure](#procedure)
  - [Part Templates](#part-templates)
    - [Part - Single Entity](#part---single-entity)
    - [Part - Multiple Entities](#part---multiple-entities)

## Parts

Guidelines for **implementing a part**:

- _derive_ from `PartBase`, even if this is not strictly a requirement, but rather a commodity. The part class must anyway implement the `IPart` interface.

- _decorate_ the class with a `TagAttribute` providing the part's type ID.

- _do not add any logic_ to the part. The part is just a POCO object modeling the data it represents, and should have no logic. The only piece of logic required is the method returning the part's data pins, which is just a form of reflecting on the part's data themselves, e.g. for indexing or semantic projection.

- if creating a part representing a _base text_ for text layers, implement the `IHasText` interface by providing a `GetText()` method which, whatever the part's model, produces a single string representing its whole text. The same interface should be implemented whenever your part has some rather long piece of free, unstructured text you might want to be included in processes like full-text indexing. Also, when the part represents the base text in a stack of textual layers, set the role ID to `base-text` (defined in `PartBase.BASE_TEXT_ROLE_ID`).

- consider that the part will be subject to automatic serialization and deserialization. As the part is just a POCO object, this should not pose any issue.

### Packages

The part project requires at least these packages:

- `Cadmus.Core`

### Procedure

The typical procedure when adding a new part is:

1. create the part class.
2. create the part seeder class.
3. create the part seeder test.
4. create the part test (using both the part under test and its part seeder).

Here I provide a number of templates, according to whether your part represents a single entity or a collection of entities. For instance, a part listing the names assigned to an item has a collection of names, each being a model on its own. Instead, a part representing a free text note is a single entity part.

### Part Templates

#### Part - Single Entity

In the following template replace `__NAME__` with your part's name, minus the `Part` suffix:

```cs
using System;
using System.Collections.Generic;
using System.Text;
using Cadmus.Core;
using Fusi.Tools.Config;

// ...

/// <summary>
/// TODO: add summary
/// <para>Tag: <c>it.vedph.__PRJ__.__NAME__</c>.</para>
/// </summary>
[Tag("it.vedph.__PRJ__.__NAME__")]
public sealed class __NAME__Part : PartBase
{
    // TODO: add your properties here...

    /// <summary>
    /// Get all the key=value pairs (pins) exposed by the implementor.
    /// </summary>
    /// <param name="item">The optional item. The item with its parts
    /// can optionally be passed to this method for those parts requiring
    /// to access further data.</param>
    /// <returns>The pins.</returns>
    public override IEnumerable<DataPin> GetDataPins(IItem? item = null)
    {
        // TODO: build pins, eventually using DataPinBuilder like this:
        // DataPinBuilder builder = new(new StandardDataPinTextFilter());
        //// latitude
        // builder.AddValue("lat", Latitude);
        //// tot-count
        //builder.Set("tot", Entries?.Count ?? 0, false);
        //return builder.Build(this);

        // ...or just use a simpler logic, like:
        // sample:
        // return Tag != null
        //    ? new[]
        //    {
        //        CreateDataPin("tag", Tag)
        //    }
        //    : Enumerable.Empty<DataPin>();

        throw new NotImplementedException();
    }

    /// <summary>
    /// Gets the definitions of data pins used by the implementor.
    /// </summary>
    /// <returns>Data pins definitions.</returns>
    public override IList<DataPinDefinition> GetDataPinDefinitions()
    {
        return new List<DataPinDefinition>(new[]
        {
            // TODO: add pins definitions...
            // sample:
            // new DataPinDefinition(DataPinValueType.Integer,
            //    "tot-count",
            //    "The total count of entries.")
        });
    }

    /// <summary>
    /// Converts to string.
    /// </summary>
    /// <returns>
    /// A <see cref="string" /> that represents this instance.
    /// </returns>
    public override string ToString()
    {
        StringBuilder sb = new();

        sb.Append("[__NAME__]");

        // TODO: append summary data...

        return sb.ToString();
    }
}
```

#### Part - Multiple Entities

```cs
using System;
using System.Collections.Generic;
using System.Text;
using Cadmus.Core;
using Fusi.Tools.Config;

// ...

/// <summary>
/// TODO: add summary
/// <para>Tag: <c>it.vedph.__PRJ__.__NAME__</c>.</para>
/// </summary>
[Tag("it.vedph.__PRJ__.__NAME__")]
public sealed class __NAME__Part : PartBase
{
    /// <summary>
    /// Gets or sets the entries.
    /// </summary>
    public List<__ENTRY__> Entries { get; set; }

    /// <summary>
    /// Initializes a new instance of the <see cref="__NAME__Part"/> class.
    /// </summary>
    public __NAME__Part()
    {
        Entries = new List<__ENTRY__>();
    }

    /// <summary>
    /// Get all the key=value pairs (pins) exposed by the implementor.
    /// </summary>
    /// <param name="item">The optional item. The item with its parts
    /// can optionally be passed to this method for those parts requiring
    /// to access further data.</param>
    /// <returns>The pins: <c>tot-count</c> and a collection of pins with
    /// these keys: ....</returns>
    public override IEnumerable<DataPin> GetDataPins(IItem? item = null)
    {
        DataPinBuilder builder = new(new StandardDataPinTextFilter());

        builder.Set("tot", Entries?.Count ?? 0, false);

        if (Entries?.Count > 0)
        {
            foreach (var entry in Entries)
            {
                // TODO: add values or increase counts like:
                // id unique values if not null:
                // builder.AddValue("id", entry.Id);
                // type-X-count counts if not null, unfiltered:
                // builder.Increase(entry.Type, false, "type-");
            }
        }

        return builder.Build(this);
    }

    /// <summary>
    /// Gets the definitions of data pins used by the implementor.
    /// </summary>
    /// <returns>Data pins definitions.</returns>
    public override IList<DataPinDefinition> GetDataPinDefinitions()
    {
        return new List<DataPinDefinition>(new[]
        {
            // TODO: add pins definitions...
            new DataPinDefinition(DataPinValueType.Integer,
               "tot-count",
               "The total count of entries.")
        });
    }

    /// <summary>
    /// Converts to string.
    /// </summary>
    /// <returns>
    /// A <see cref="string" /> that represents this instance.
    /// </returns>
    public override string ToString()
    {
        StringBuilder sb = new();

        sb.Append("[__NAME__]");

        if (Entries?.Count > 0)
        {
            sb.Append(' ');
            int n = 0;
            foreach (var entry in Entries)
            {
                if (++n > 3) break;
                if (n > 1) sb.Append("; ");
                sb.Append(entry);
            }
            if (Entries.Count > 3)
                sb.Append("...(").Append(Entries.Count).Append(')');
        }

        return sb.ToString();
    }
}
```
