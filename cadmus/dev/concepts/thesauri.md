---
layout: page
title: Thesauri
subtitle: "Cadmus Development"
---

- [Requirements](#requirements)
- [Overview](#overview)
- [Role-Dependent Thesauri](#role-dependent-thesauri)
- [Model](#model)
  - [Hierarchical Thesauri](#hierarchical-thesauri)
  - [Aliases](#aliases)
  - [Models Thesaurus](#models-thesaurus)
- [Bulk Import](#bulk-import)

## Requirements

To deal with thesauri, you will need:

- rudimentary knowledge of [JSON](https://www.json.org/json-en.html).
- a code editor like [VSCode](https://code.visualstudio.com/download), or an online JSON editor like [JSON editor online](https://jsoneditoronline.org).

💡 Also, there are some **tools** you can use:

- [Cadmus command line tool](https://github.com/vedph/cadmus_tool): this tool has functions to variously import thesauri into an existing database with different modes (replacing, patching, or synching), and from different formats (JSON, CSV, XLSX, XLS).
- [online tools](https://cadmus.fusi-soft.com/#/profile): the Cadmus presentation website has some visual editing tools to help you work with thesauri.

## Overview

Thesauri are taxonomies you can share across any number of data models (parts), wherever users need to pick a value from a predefined set.

These taxonomies can be **simple**, "flat" lists, or **hierarchical** lists.

In both cases, they are defined as objects having each a list of entries. Each **entry** in a thesaurus object has an arbitrarily assigned `id`, which must be unique in the context of that object; and a text `value` to be displayed to the end user.

For instance, you might have a list of languages to pick from, like English, French, Italian, etc.: in this case, your thesaurus would have its own ID, e.g. languages, plus a number of entries like `eng`=`English`, `fre`=`French`, `ita`=`Italian`, etc. Here I'm using ISO 639 for language codes, and using standards is always recommended when available; but IDs are totally arbitrary.

>You may also want to take a mixed approach where useful. For instance, say you want a list of epigraphic categories to be later mapped to the LOD identifiers defined by [EAGLE](https://www.eagle-network.eu/voc/typeins.html). Such identifiers are full URIs like `https://www.eagle-network.eu/voc/typeins/lod/76` representing a `tabula defixionis`; so, if we want shorter identifiers but still be able to map to this taxonomy, we might strip off the constant prefix and just use some shorter, human-friendly ID followed by the numeric ID, which is the only variable part of these URIs. Just like in RDF we use prefixes to shorten URIs, we will thus build IDs like `defixio-76`. Later, rebuilding the full URI by replacing the short prefix `defixio` with the full URI portion `https://www.eagle-network.eu/voc/typeins/lod/` will be trivial processing. This way, we have short and readable IDs, which is the recommended practice for Cadmus, without losing the connection to an existing third-party ontology.

Internally, in the database _the only values stored in parts will be IDs_. So you are free to label each entry as you prefer, as far as you provide a consistent and immutable ID for it. This has the obvious advantage of separating language-specific labels, targeting human users, from the unique ID, targeting machines.

All the thesauri are initially defined in the **JSON document** with the configuration for your project. You can thus easily edit this document with any text editor, adding all the taxonomies you might need. On first startup, when seeding the initial database, the Cadmus API service will read taxonomies from this JSON document and inject them into the corresponding collection in the database.

Once you have started working on your data, you can still **edit** and add new taxonomies via the UI provided by the Cadmus editor. Alternatively, you can always import data manually in the database using your favorite bulk import method.

## Role-Dependent Thesauri

In this context, each type of UI component (part or fragment editor) can request all the thesauri it requires. This links the component type with these IDs. Yet, in some cases we might need to have the same type requiring different thesauri according to its role in the context of the same item.

For instance, say we want not just one [categories part](../models.md#general), but two of them: one for the epigraphic categories, and another for a set of arbitrarily defined periods (e.g. archaic, hellenistic, early imperial, etc.). So, we can have a second part of the same type, categories, in an inscription item; this way, the original categories part continues to represent the epigraphic categories, while the periods categories part will represent the date in a more approximate and conventional way.

To this end, the newly added categories part will get a _role_, which distinguishes it from the original one. Now we can have two categories part in the inscription item; but we still need to tell the periods cayegories part to use a different thesaurus to get its categories.

>The role is a standard mechanism to have more than a single part type inside the same item, and is present since the beginning in the system.

In this scenario, your editor component can opt into a feature which allows the thesauri requested to the server to get an additional suffix equal to underscore (`_`) plus the role ID. This is what happens in the categories part.

>Opting in is thus an option for the model developer. To opt in, just set `this.roleIdInThesauri = true;` in the constructor of your part editor wrapper (the component derived from `EditPartFeatureBase`).

So, you could have parts defined like (in your profile's `facets/partDefinitions`; note the `roleId` representing the different roles for these two parts having the same `typeId`):

```json
{
  "typeId": "it.vedph.categories",
  "name": "categories",
  "description": "Epigraphic categories.",
  "isRequired": true,
  "colorKey": "98F8F8",
  "groupKey": "general",
  "sortKey": "a"
},
{
  "typeId": "it.vedph.categories",
  "roleId": "periods",
  "name": "categories",
  "description": "Historical periods.",
  "isRequired": false,
  "colorKey": "98F8F0",
  "groupKey": "general",
  "sortKey": "b"
},
```

and two distinct thesauri:

```json
thesauri: [
  {
    "id": "categories@en",
    "entries": [
        {
          "id": "defixio-76",
          "value": "defixio"
        },
        // ...etc.
    ]
  },
  {
    "id": "categories_periods@en",
    "entries": [
      {
        "id": "arc",
        "value": "archaic"
      },
      // ...etc.
    ]
  },
]
```

>If some of the requested thesauri need not to change, just add in the backend profile an [alias thesaurus](#aliases) with the role suffix, pointing to the original thesaurus.

## Model

Thesauri can be edited directly in the JSON document containing the Cadmus system configuration for your project (`seed-profile.json`), under the property named `thesauri`. This is an array (a list) of objects. So, the general skeleton for a set of thesauri is:

```json
{
  "thesauri": [
    {
      "id": "thesaurus-1-id",
      "entries": [
        {
          "id": "the-first-entry-id",
          "value": "Some label for the first entry"
        },
        {
          "id": "the-second-entry-id",
          "value": "Some label for the second entry"
        }
      ]
    },
    {
      "id": "thesaurus-2-id",
      "entries": [
        {
          "id": "the-first-entry-id",
          "value": "Some label for the first entry"
        },
        {
          "id": "the-second-entry-id",
          "value": "Some label for the second entry"
        }
      ]
    }
  ]
}
```

As for the **order of thesauri**, by convention, you should write your thesauri in the Cadmus configuration file sorting them by their ID. This is not a technical requirement, but it's handy for human operators, who need to browse through the list of thesauri while editing.

As for the **order of entries** inside each thesaurus, this is usually reflected in the UI, so that you can be sure that the entries will be presented in the same order you entered them in the source JSON document. Thus, _be sure to place the entries in the order you want to be used when displaying them_, except for those thesauri having no intrinsic order as they are used only as lookup lists, like for the reserved `model-types` (see [below](#models-thesaurus)).

Each thesaurus object has these properties:

- `id`: the thesaurus unique ID, suffixed with `@` followed by its ISO639-2 language code; e.g. `languages@en`. Currently, the language suffix is not used in the frontend implementation, so just leave it to english (`en`), whatever the language you use in your entry's value.
- `entries`: an array of entries, each being an object having:
  - `id`: the entry ID.
  - `value`: the entry value.

>⚠️ An ID should never contain spaces, and dots are reserved for hierarchical IDs (see below). You should just use lowercase letters `a`-`z` without diacritics, digits, and dashes. Also, IDs must be reasonably short, and be readable also for human users.

Here is a sample thesaurus with a couple of entries:

```json
{
  "id": "languages@en",
  "entries": [
    {
      "id": "lat",
      "value": "Latin"
    },
    {
      "id": "grc",
      "value": "Greek"
    }
  ]
}
```

>If you manually edit thesauri, please pay attention to the JSON syntax for the usage of quotes wrapping any identifier or value, commas for separating properties, square brackets for lists, and braces for objects. It is recommended that you use a code editor which is aware of this syntax and can thus help you quickly write JSON code and spot eventual errors.

### Hierarchical Thesauri

You can also have hierarchical thesauri, where entries are arranged in a hierarchy represented with _dots_ in their IDs. For instance, say you want to represent this simple 2-levels hierarchy:

- language:
  - language: phonology
  - language: morphology
  - language: syntax

You can use a dot in each entry to represent three children entries under the same node:

```json
[
  { "id": "lang.pho", "value": "language: phonology" },
  { "id": "lang.mor", "value": "language: morphology" },
  { "id": "lang.syn", "value": "language: syntax" }
]
```

Should you want to have a selectable entry also for the parent language node, you just have to add another one, like this:

```json
[
  { "id": "lang.-", "value": "language" },
  { "id": "lang.pho", "value": "language: phonology" },
  { "id": "lang.mor", "value": "language: morphology" },
  { "id": "lang.syn", "value": "language: syntax" }
]
```

There is no limit to the level of nesting in such thesauri.

>By convention, you should have your labels contain the full path to the entry, i.e. all the ancestor labels, separated by a colon. So for instance here `lang.pho` corresponds to `language: phonology`. Many editors rely on this convention to shorten the label when this is useful, while still preserving the full information about their placement in the hierarchy.

It is up to the model editor software to decide which thesauri they require, and whether these are to be flat or hierarchical. So, while adding thesauri to your configuration be sure to use the thesauri IDs expected by the parts you have selected, and to comply with its type.

Also, most of the part editors are implemented so that they _tolerate the absence of any of the thesauri they request_: in this case, they provide a different UI according to whether the requested thesaurus is found or not. For instance, an editor expecting a list of from a thesaurus with ID `languages@en` will provide a dropdown list used to select any of the thesaurus entries; or, if this thesaurus is not found, it will just provide a text box where users will be free to type whatever they want.

### Aliases

Also, a thesaurus can be just an **alias** targeting another, existing thesaurus. In this case it has no entries, but only a single `targetId` property, with the ID of the thesaurus it refers to.

This can be useful when you have several part editors which happen to expect different thesauri IDs for the same taxonomy.

Example:

```json
[
    {
      "id": "languages@en",
      "entries": [
        {
          "id": "lat",
          "value": "Latin"
        },
        {
          "id": "grc",
          "value": "Greek"
        }
      ]
    },
    {
      "id": "name-languages@en",
      "targetId": "languages"
    }
]
```

Here we have a `languages` thesaurus with some languages, used by a keywords part, and a `name-languages` thesaurus, used by a proper names part. In this case, we want the same list of languages for both the keywords and the proper names, so instead of duplicating the entries we just create an alias thesaurus `name-languages` targeting `languages`. If instead we want two different list of languages, we just replace the alias with a full thesaurus having its own entries.

### Models Thesaurus

The ID `model-types@en` is conventionally **reserved** to provide a mapping between machine-targeted type IDs (like `it.vedph.categories`) and their human-friendly names (like `categories`).

This is systematically used in the editor UI to show human friendly names instead of full IDs when referring to data parts. You should thus always provide this thesaurus in your project, unless you want to deal with ugly, longer part IDs.

>By convention, entries in this thesaurus are sorted by their ID, but all the fragment-related entries come after the others, like in the sample below.

Example:

```json
"id": "model-types@en",
"entries": [
    {
        "id": "it.vedph.bibliography",
        "value": "bibliography"
    },
    {
        "id": "it.vedph.categories",
        "value": "categories"
    },
    {
        "id": "it.vedph.comment",
        "value": "comment"
    },
    {
        "id": "it.vedph.doc-references",
        "value": "references"
    },
    {
        "id": "it.vedph.external-ids",
        "value": "external IDs"
    },
    {
        "id": "it.vedph.historical-date",
        "value": "date"
    },
    {
        "id": "it.vedph.index-keywords",
        "value": "index keywords"
    },
    {
        "id": "it.vedph.metadata",
        "value": "metadata"
    },
    {
        "id": "it.vedph.names",
        "value": "names"
    },
    {
        "id": "it.vedph.note",
        "value": "note"
    },
    {
        "id": "it.vedph.token-text",
        "value": "text"
    },
    {
        "id": "it.vedph.token-text-layer",
        "value": "text layer"
    },
    {
        "id": "fr.it.vedph.apparatus",
        "value": "apparatus fr."
    },
    {
        "id": "fr.it.vedph.comment",
        "value": "comment fr."
    },
    {
        "id": "fr.it.vedph.orthography",
        "value": "orthography fr."
    }
]
```

## Bulk Import

Typically most thesauri are created with the editor; you can then use the editing UI to make modifications. Anyway, when you massively add or edit data, it may be convenient to just remove the thesauri collection from the Mongo database and re-import it. To this end, you can use the Cadmus [command line tool](https://github.com/vedph/cadmus_tool?tab=readme-ov-file#thesaurus-import-command) for more fine-grained imports, or also generic MongoDB commands, should you need to directly patch the database.

The preferred bulk import or update method is via the [CLI tool](https://github.com/vedph/cadmus_tool?tab=readme-ov-file#thesaurus-import-command).

To manually patch the database (for advanced users!):

(1) export the `thesauri` collection from an up-to-date database. This can be created by just launching the editor with updated thesauri in an environment where there is no database for that project. In this case, the databases will be created anew, together with their thesauri. The fastest export type is probably a gzipped archive resulting in a single compressed file, like e.g. `thesauri.agz`.

(2) login into the target database with MongoDB shell, and drop the thesauri collection:

```bash
mongo
use cadmus-yourproject;
db.thesauri.drop();
```

(3) restore the `thesauri` collection from the archive file:

```bash
mongorestore --gzip --archive=thesauri.agz --nsInclude 'cadmus-itinera.thesauri'
```

>Note: in Linux you might need to specify the host in your server, e.g. adding the option `-h 127.0.0.1`.

🏠 [developer's home](../toc.md)
