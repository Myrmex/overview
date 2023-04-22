---
layout: page
title: Pythia
subtitle: "Text Search Engine"
---

- [Real World Applications](#real-world-applications)
- [Main Features](#main-features)
- [Architecture](#architecture)
- [Analysis](#analysis)
- [Tooling](#tooling)

📖 [source and documentation](https://github.com/vedph/pythia)

Pythia is a simple and modular concordance search engine, first designed in the context of [Chiron](chiron.md), to provide a concordance-based search engine capable of handling an overwhelming metadata set, even coming from different sources, and deal with text structures longer than just tokens, even when overlapping. It was designed to be easy to integrate in other systems and fully customizable, with a new approach to text, "dematerialized" into a set of objects. It is currently adopted by another research project, [Atti Chiari](https://attichiari.unige.it/), in the context of a wider flow including also a couple of other software tools I created for a new type of pseudonymization for documents with sensitive data, and a very simple [BLOB remote archiving system](https://github.com/vedph/simple-blob) to collect them.

For a general introduction see D. Fusi, _Text Searching Beyond the Text: a Case Study_, «Rationes Rerum» 15 (2020) 199-230, and the [source code repository](https://github.com/vedph/pythia). With relation to the cited paper, the current implementation of the system is more advanced, the query syntax was changed, and some new features were added; but the approach is the same.

## Real World Applications

While being designed for more complex texts, the first real-world application of the Pythia prototype is _Minerva_, an upcoming digital service from project [AttiChiari](https://attichiari.unige.it/). You can follow a short, totally non-technical presentation about it in this video (in Italian):

<iframe width="560" height="315" src="https://www.youtube.com/embed/EgnFYD5qpz0?start=17596" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

## Main Features

- _full stack architecture_, from database to business layer to web API and web frontend, all deployed in a Docker compose stack.

- _concordance-based_: designed from the ground up with concordances in mind: word locations here are not an afterthought or an additional payload attached to an existing location-less engine. The whole architecture is based on positions in documents, and these positions may also refer to other text structures besides words. In this higher abstraction level, a text is somewhat "dematerialized" into a set of token-based _positions_ linked to an open set of _metadata_. Rather than a long sequence of characters, a text is viewed as an abstract set of entities, each having any metadata, in most cases also including their position in the original text. These entities may represent documents, groups of documents (corpora), and words and any other textual structure (e.g. a verse, a strophe, a sentence, a phrase, etc.), with no limits, even when multiple structures overlap. Searching for a verse or a sentence, or whatever other textual structure is equal to searching for a word; and we can freely mix and combine these different entity types in a query.

- _minimal dependencies_: simple implementation with widely used standard technologies: the engine relies on a RDBMS, and is wrapped in a REST API. The only dependency is the database service. The index is just a standard RDBMS, so that you can easily integrate it into your own project. You might even bypass the search engine, and directly query or otherwise manipulate it via SQL.

- _flexible, modular and open_: designed to be totally configurable via external parameters: you decide every relevant aspect of the indexing pipeline (filtering, tokenization, etc.), and can use any kind of input format (e.g. plain text, TEI, etc.) and source (e.g. file system, BLOB storage, web resources etc.).

- [UDPipe](https://ufal.mff.cuni.cz/udpipe) plugins to incorporate any subset of POS tagging data into the index.

## Architecture

- [model](pythia/model.md)
- [query](pythia/query.md)
- [query samples](pythia/query-samples.md)
- [SQL queries](pythia/sql.md)
  - [SQL queries without location operators](pythia/sql-ex-non-locop.md)
  - [SQL queries with location operators](pythia/sql-ex-locop.md)
- [term distributions](pythia/term-list.md)
- [storage](pythia/storage.md)

## Analysis

- [analysis process](pythia/analysis.md)
- [software components](pythia/components.md)
- [integrating UDPipe](pythia/udp.md)
  - [simple example](pythia/example.md)
  - [real world example](pythia/example-ac.md)

## Tooling

- [CLI tool](pythia/cli.md)
