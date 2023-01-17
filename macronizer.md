---
layout: page
title: Macronizer API
subtitle: "Alatius Latin Macronizer Wrapper"
---

>This software is part of a project that has received funding from the European Research Council (ERC) under the European Union's Horizon 2020 research and innovation program under grant agreement No. 101001991.

- 👀 [API demo](https://macronizer-api.fusi-soft.com/swagger/index.html): please notice that this service is for demonstrative purposes only, and is not production-oriented, as it relies on a low-end VM on shared hardware resources.
- [higher layer source code](https://github.com/Myrmex/macronizer-api)
- [lower layer source code](https://github.com/Myrmex/alatius-macronizer-api)

In the context of the [Chiron](chiron.md) framework, the Latin language poses the issue of detecting vocalic lengths. This issue is much more relevant than in ancient Greek, where further indications about lengths come from graphemes (η vs. ε, and ο vs. ω) and accent marks combined with Greek accentuation rules. This is particularly important for a framework tendentially oriented to collect as much data as possible, independently from their intended usage. For instance, should we interested only in syllabic weights, we might even be able to ignore the issue for all closed syllables, which are anyway heavy, whatever the length of their vocalic nucleus. Yet, often we are interested in more fine-grained details, which require detecting vowel lengths independently from their syllabic context.

For all these cases, the typical solution is using a "macronizer", i.e. a piece of software which explicitly marks all the long vowels with macron diacritics, while leaving unmarked all the short ones. This allows a much better analysis of prosodical data, both in poetry and in prose. One of the best Latin macronizers is John Winge's [Alatius Macronizer](https://alatius.com/macronizer/). This is essentially based on Python running on top of C tools like [RFTagger](http://www.cis.uni-muenchen.de/~schmid/tools/RFTagger/) and [Latin dependency treebank](http://www.dh.uni-leipzig.de/wo/projects/ancient-greek-and-latin-dependency-treebank-2-0/). It also provides an Apache-based web page on top of its engine. Yet, what is really needed for projects requiring to integrate it is a web API, so that any software client, whatever its language, can take advantage of that functionality.

To this end, I added a couple of layers on top of this tool:

1. a very thin [Flask API layer](https://github.com/Myrmex/alatius-macronizer-api) on top of the Python-based engine, based on Python too (via [Flask](https://flask.palletsprojects.com/) and [waitress](https://docs.pylonsproject.org/projects/waitress/en/latest/)): this already provides a ready-to-run service, which just passes data down to the engine by running its Python script.
2. a more robust [ASP.NET API layer](https://github.com/Myrmex/macronizer-api) on top of the preceding layer, adding rate limiting, service auditing and monitoring, plus some essential preprocessing and postprocessing functions.

Both these layers are distributed in Docker images. Once you have a Docker image wrapping the macronizer, all its software dependencies, and its PostgreSQL database, it becomes much easier to consume its functionality: you just have to add a layer to your Docker stack, and consume the API endpoint for macronization. Currently, both Chiron and its prosodical rhythm extensions with 3 client UIs (CLI, native desktop, and web-based) are using the service wrapped in these layers.
