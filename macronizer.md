---
layout: page
title: Macronizer API
subtitle: "Alatius Latin Macronizer Wrapper"
---

>This software is part of a project that has received funding from the European Research Council (ERC) under the European Union's Horizon 2020 research and innovation program under grant agreement No. 101001991.

- 👀 [API demo](https://macronizer-api.fusi-soft.com/swagger/index.html): please notice that this service is for demonstrative purposes only, and is not production-oriented, as it relies on a low-end VM on shared hardware resources.
- [higher layer source code](https://github.com/Myrmex/macronizer-api)
- [lower layer source code](https://github.com/Myrmex/alatius-macronizer-api)

In the context of the [Chiron](chiron.md) framework, the Latin language poses the problem of detecting vocalic lengths. This problem is much more relevant than in ancient Greek, where on the purely graphical level further indications about lengths come from graphemes (η vs. ε, and ο vs. ω, while in Latin all the vowels are _dichronae_) and accent marks combined with Greek accentuation rules. This is particularly important for a framework tendentially oriented to collect as much data as possible, independently from their intended usage. For instance, should we interested only in syllabic weights, we might even be able to ignore the issue for all closed syllables, which are anyway heavy, whatever the length of their vocalic nucleus. Yet, often we are interested in more fine-grained details, which require detecting vowel lengths independently from their syllabic context.

For all these cases, the typical solution is using a "macronizer", i.e. a piece of software which explicitly marks all the long vowels with macron diacritics, while leaving unmarked all the short ones. This allows a much better analysis of prosodical data, both in poetry and in prose. One of the best Latin macronizers is John Winge's [Alatius Macronizer](https://alatius.com/macronizer/). This is essentially based on Python running on top of C tools like [RFTagger](http://www.cis.uni-muenchen.de/~schmid/tools/RFTagger/) and [Latin dependency treebank](http://www.dh.uni-leipzig.de/wo/projects/ancient-greek-and-latin-dependency-treebank-2-0/). It also provides an Apache-based web page on top of its engine. Yet, what is really needed for projects requiring to integrate it is a web API, so that any software client, whatever its technology, can take advantage of that functionality.

To this end, I added a couple of layers on top of this tool:

1. a very thin [Flask API layer](https://github.com/Myrmex/alatius-macronizer-api) on top of the Python-based engine, based on Python too (via [Flask](https://flask.palletsprojects.com/) and [waitress](https://docs.pylonsproject.org/projects/waitress/en/latest/)): this already provides a ready-to-run service, which just passes data down to the engine by running its Python script.
2. a more robust [ASP.NET API layer](https://github.com/Myrmex/macronizer-api) on top of the preceding layer, adding rate limiting, service auditing and monitoring, plus some essential preprocessing and postprocessing functions.

Both these layers are distributed in Docker images. Once you have a Docker image wrapping the macronizer, all its software dependencies, and its PostgreSQL database, it becomes much easier to consume its functionality: you just have to add a layer to your Docker stack, and consume the API endpoint for macronization. Currently, both Chiron and its prosodical rhythm extensions with 3 client UIs (CLI, native desktop, and web-based) are using the service wrapped in these layers.

## Usage

>For more information, see the documentation in the [GitHub repository](https://github.com/Myrmex/macronizer-api).

Apart from endpoints used for diagnostic purposes, the API exposes a single endpoint for the macronization service, `api/macronize`, for a POST request whose body corresponds to a JSON object having this model (\* marks required properties):

- `text` (string)\*: the text to macronize. Max 50,000 characters.
- `maius` (boolean): true to macronize capitalized words.
- `utov` (boolean): true to convert U to V.
- `itoj` (boolean): true to convert I to J.
- `ambiguous` (boolean): true to mark ambiguous results. In this case, the output will be HTML instead of plain text, with `span` elements wrapping each word, with a `class` attribute equal to `ambig` or `unknown` (or `auto` for unmarked vowels). In turn, each of these spans will wrap the vowels inside an attribute-less span `element`. You can use the options below to convert it before returning the result.
- `normalizeWS` (boolean): true to normalize whitespace in text before macronization. This normalizes space/tab characters by replacing them with a single space, and trimming the text at both edges. It also normalizes CR+LF into LF only.
- `precomposeMN` (boolean): true to to apply Mn-category Unicode characters precomposition before macronization. This precomposes Unicode Mn-category characters with their letters wherever possible. Apply this filter when the input text has Mn-characters to avoid potential issues with macronization.
- `dropNonMacronEscapes`: whether to drop escapes referred to vowels not having a macron. When macronizer returns a marked word, all the vowels in it are wrapped in a span, which can be rendered here according to the values set for the escape properties of these options. So, a word like `Gallia` might come out marked as ambiguous, having vowels `a`, `i`, and `ā` marked inside it; yet, while the marks have the purpose of locating vowels, it's only the `ā` with the macron which should be intended as ambiguous. When this option is true, the escapes specified by the other escape properties will be applied only to vowels having a macron.
- `unmarkedEscapeOpen` (string): the optional opening escape to use for an unmarked-form vowel.
- `unmarkedEscapeClose` (string): the optional closing escape to use for an unmarked-form vowel.
- `ambiguousEscapeOpen` (string): the optional opening escape to use for an ambiguous-form vowel.
- `ambiguousEscapeOpen` (string): the optional closing escape to use for an ambiguous-form vowel.
- `unknownEscapeOpen` (string): the optional opening escape to use for an unknown-form vowel.
- `unknownEscapeClose` (string): the optional closing escape to use for an unknown-form vowel.

The result is a JSON object having this model:

- `result` (string): text. When set, there is no `error`.
- `error` (string): error. When set, there is no `result`.
- `maius` (boolean): true to macronize capitalized words.
- `utov` (boolean): true to convert U to V.
- `itoj` (boolean): true to convert I to J.
- `ambiguous` (boolean): true to mark ambiguous results.

For instance, this request:

```bash
curl -X 'POST' \
  'https://macronizer-api.fusi-soft.com/api/macronize' \
  -H 'accept: application/json' \
  -H 'Content-Type: application/json' \
  -d '{
"ambiguous": true,
"text": "Gallia est omnis divisa in partes tres, quarum unam incolunt Belgae, aliam Aquitani, tertiam qui ipsorum lingua Celtae, nostra Galli appellantur.",
"unmarkedEscapeOpen": "",
"unmarkedEscapeClose": "",
"ambiguousEscapeOpen": "",
"ambiguousEscapeClose": "¿",
"unknownEscapeOpen": "",
"unknownEscapeClose": "¡"
}'
```

results in this response, where all the ambiguous forms vowels have been marked by a `¿` suffix:

```json
{
  "result": "Ga¿lli¿ā¿ e¿st o¿mnī¿s dī¿vī¿sa¿ in partēs trēs, quārum ūnam incolunt Belgae, aliam Aquītānī, tertiam quī ipsōrum li¿ngu¿ā¿ Celtae, no¿stra¿ Gallī appellantur.",
  "error": null,
  "maius": false,
  "utov": false,
  "itoj": false,
  "ambiguous": true
}
```
