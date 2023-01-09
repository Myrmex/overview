---
layout: page
title: Pythia
subtitle: "Text Search Engine"
---

📖 [source and documentation](https://github.com/vedph/pythia)

Pythia is a simple and modular concordance search engine, first designed in the context of [Chiron](chiron.md), to provide a concordance-based search engine capable of handling an overwhelming metadata set, even coming from different sources, and deal with text structures longer than just tokens, even when overlapping. It was designed to be easy to integrate in other systems and fully customizable, with a new approach to text, "dematerialized" into a set of objects. It is currently adopted by another research project, [Atti Chiari](https://attichiari.unige.it/), in the context of a wider flow including also a couple of other software tools I created for a new type of pseudonymization tool for documents with sensitive data, and a very simple [BLOB remote archiving system](https://github.com/vedph/simple-blob) to collect them.

For a general introduction see D. Fusi, _Text Searching Beyond the Text: a Case Study_, «Rationes Rerum» 15 (2020) 199-230, and the more detailed and up to date [documentation](https://github.com/vedph/pythia). With relation to the cited paper, the implementation of the system here is more advanced, and query syntax was changed; but the approach is the same.

Main features:

- _full stack architecture_, from database to business layer to web API and web frontend, all deployed in a Docker compose stack.

- _concordance-based_: designed from the ground up with concordances in mind: word locations here are not an afterthought or an additional payload attached to an existing location-less engine. The whole architecture is based on positions in documents, and these positions may also refer to other text structures besides words. In this higher abstraction level, a text is somewhat "dematerialized" into a set of token-based _positions_ linked to an open set of _metadata_. Rather than a long sequence of characters, a text is viewed as an abstract set of entities, each having any metadata, in most cases also including their position in the original text. These entities may represent documents, groups of documents (corpora), and words and any other textual structure (e.g. a verse, a strophe, a sentence, a phrase, etc.), with no limits, even when multiple structures overlap. Searching for a verse or a sentence, or whatever other textual structure is equal to searching for a word; and we can freely mix and combine these different entity types in a query.

- _minimal dependencies_: simple implementation with widely used standard technologies: the engine relies on a RDBMS, and is wrapped in a REST API. The only dependency is the database service. The index is just a standard RDBMS, so that you can easily integrate it into your own project. You might even bypass the search engine, and directly query or otherwise manipulate it via SQL.

- _flexible, modular and open_: designed to be totally configurable via external parameters: you decide every relevant aspect of the indexing pipeline (filtering, tokenization, etc.), and can use any kind of input format (e.g. plain text, TEI, etc.) and source (e.g. file system, BLOB storage, web resources etc.).

- UDPipe plugins to incorporate any subset of POS tagging data into the index.
