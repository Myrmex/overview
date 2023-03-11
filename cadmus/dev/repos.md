---
layout: page
title: Cadmus Development
subtitle: Repositories
---

- [Backend](#backend)
  - [Models](#models)
  - [Services](#services)
  - [Tools](#tools)
- [Frontend](#frontend)
  - [Shells](#shells)
  - [Applications](#applications)

Cadmus repositories are all hosted in the [VeDPH GitHub site](https://github.com/vedph).

The general organization of Cadmus repositories follows these guidelines:

- backend and frontend repositories are always separate.
- a bunch of basic repositories provides core infrastructure for backend and frontend.
- data models and editors are grouped into repositories by subject: currently these are: general, philology, codicology, epigraphy, geography, imaging.
- some repositories provide additional services (like for external bibliography) and tools.
- each Cadmus-based project has its own repositories. At a minimum, they just have an API backend and a frontend. Most of them, which provide their own models, also have a models library. Conventional naming is as follows:
  - `cadmus-PRJ`: specialized models, if any.
  - `cadmus-PRJ-api`: API backend.
  - `cadmus-PRJ-app`: frontend app.
  - `cadmus-PRJ-doc`: optional documentation.

## Backend

All the backend Cadmus libraries are distributed via [NuGet](https://www.nuget.org).

### Models

👉 General models:

- [Cadmus core](https://github.com/vedph/cadmus_core): the core set of libraries for Cadmus backend, providing the infrastructure for core, indexing, storage, and mock data seeding.
- [Cadmus bricks](https://github.com/vedph/cadmus-bricks): the core set of libraries including sub-models shared across many of the part/fragment models.
- [Cadmus graph](https://github.com/vedph/cadmus-graph): libraries used for the semantic graph projection and editing of Cadmus data. 👀 A small [online demo for the mapping infrastructure](https://cadmus-graph-demo.fusi-soft.com/) is available.
- [Cadmus migration](https://github.com/vedph/cadmus-migration): libraries used for data migration (e.g. TEI export).
- [Cadmus general models](https://github.com/vedph/cadmus-general): libraries including generic-usage data models (parts and fragments). Practically all the Cadmus projects use one or more of these components. More specialized models are found in other libraries.
- [Cadmus philology models](https://github.com/vedph/cadmus-philology): libraries including philological data models (parts and fragments).

👉 Codicology models:

- [Cadmus codicology models](https://github.com/vedph/cadmus-codicology): libraries including codicological data models (parts).
- [Cadmus codicology API](https://github.com/vedph/cadmus-codicology-api): API using codicological models, providing the backend for the codicology frontend shell.

👉 Epigraphy models:

- [Cadmus epigraphy models](https://github.com/vedph/cadmus-epigraphy)
- [Cadmus epigraphy API](https://github.com/vedph/cadmus-epigraphy-api): API using epigraphic models, providing the backend for the epigraphy frontend shell.

👉 Geography models:

- [Cadmus geography models](https://github.com/vedph/cadmus-geo): libraries including geographic models (parts).
- [Cadmus geography API](https://github.com/vedph/cadmus-geo-api): API using geographic models, providing the backend for the epigraphy frontend shell.

👉 Imaging models:

- [Cadmus imaging models](https://github.com/vedph/cadmus-img): libraries including imaging models (parts).
- [Cadmus imaging API](https://github.com/vedph/cadmus-img-api): API using imaging models, providing the backend for the imaging frontend shell.

### Services

External bibliography service:

- [Bibliography](https://github.com/vedph/cadmus_biblioapi): this is a generic bibliography service, based on its own database (MySql). Yet, it can be used by Cadmus projects with a high level of integration when you prefer a top-down approach for bibliography.

### Tools

- [Cadmus CLI tool](https://github.com/vedph/cadmus_tool): Cadmus CLI utility tool.
- [Cadmus preview templates](https://github.com/vedph/cadmus-previews): preview templates for Cadmus, used as helpers for designing HTML output in the editor, based on XSLT.

## Frontend

All the frontend Cadmus libraries are distributed via [NPM](https://www.npmjs.com).

### Shells

General shells:

- [Cadmus shell](https://github.com/vedph/cadmus-shell-2): the core set of frontend libraries of any Cadmus application. This can be run as an application, but its intended usage is just providing a playground (and an app sample) used for development. This shell includes the generical and philological frontend libraries, corresponding to the backend models in their respective libraries.
- [Cadmus bricks shell](https://github.com/vedph/cadmus-bricks-shell): the shell used for developing frontend sub-models. 👀 Its [online version](https://cadmus-bricks.fusi-soft.com) can be used to play with all the controls.
- [Cadmus graph shell](https://github.com/vedph/cadmus-graph-shell): the shell used for developing frontend semantic graph components.

👉 Bibliography shell:

- [Bibliography shell](https://github.com/vedph/cadmus_biblio_shell): the shell used for developing frontend bibliographic models. Note that these are independent from Cadmus, except for a single component used to adapt the external bibliographic service to Cadmus.

👉 Codicology shell:

- [Cadmus codicology shell](https://github.com/vedph/cadmus-codicology-shell): the shell used for developing frontend codicological models, including the codicology frontend libraries.

👉 Epigraphy shell:

- [Cadmus epigraphy shell](https://github.com/vedph/cadmus-epigraphy-shell): the shell used for developing frontend epigraphic models, including the epigraphy frontend libraries.

👉 Geography shell:

- [Cadmus geography shell](https://github.com/vedph/cadmus-geo-shell): the shell used for developing frontend geographic models, including the geography frontend libraries.

👉 Imaging shell:

- [Cadmus imaging shell](https://github.com/vedph/cadmus-img-shell): the shell used for developing frontend imaging models, including the imaging frontend libraries.

### Applications

- [Cadmus presentation app](https://github.com/vedph/cadmus_show_app): a short interactive introduction to Cadmus with some online tools: 👀 its [online version](https://cadmus.fusi-soft.com) is available.

👉 [GISARC](https://6001.cophilab-cloud.ilc.cnr.it/home):

- [models](https://github.com/vedph/cadmus-gisarc)
- [API](https://github.com/vedph/cadmus-gisarc-api)
- [app](https://github.com/vedph/cadmus-gisarc-app)

👉 Inquisitions Graffiti:

- [models](https://github.com/vedph/cadmus_ingra)
- [API](https://github.com/vedph/cadmus_ingra_api)
- [app](https://github.com/vedph/cadmus_ingra_app)

👉 [Itinera](https://itinera.unisi.it):

- [models](https://github.com/vedph/cadmus_itinera)
- [API](https://github.com/vedph/cadmus_itinera_api)
- [app](https://github.com/vedph/cadmus_itinera_app)

👉 MapAeg:

- [API](https://github.com/vedph/cadmus_bdm_api)
- [app](https://github.com/vedph/cadmus-bdm-app)

👉 Musisque Deoque:

- [Musisque Deoque API](https://github.com/vedph/cadmus_mqdq_api)
- [Musisque Deoque app](https://github.com/vedph/cadmus_mqdq_app)
- [conversion tool](https://github.com/vedph/mqdq_mqutil)

👉 [PAGES](https://web.uniroma1.it/pages) and related projects:

- [models](https://github.com/vedph/cadmus_tgr)
- [API](https://github.com/vedph/cadmus_tgr_api)
- [app](https://github.com/vedph/cadmus_tgr_app)
- [documentation](https://github.com/vedph/cadmus_tgr_doc)

👉 [PURA](https://6008.cophilab-cloud.ilc.cnr.it):

- [models](https://github.com/vedph/cadmus_pura)
- [API](https://github.com/vedph/cadmus_pura_api)
- [app](https://github.com/vedph/cadmus_pura_app)
- [documentation](https://github.com/vedph/cadmus_pura_doc)

👉 [Re.Novella](http://renovella.unisi.it):

- [models](https://github.com/vedph/cadmus-renovella)
- [API](https://github.com/vedph/cadmus-renovella-api)
- [app](https://github.com/vedph/cadmus-renovella-app)

👉 VeLA:

- [API](https://github.com/vedph/cadmus-vela-api)
- [app](https://github.com/vedph/cadmus-vela-app)

👉 Sidonius Letters:

- [API](https://github.com/vedph/cadmus-sidon-api)
- [app](https://github.com/vedph/cadmus-sidon-app)

🏠 [developer's home](toc.md)
