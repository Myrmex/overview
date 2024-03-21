---
layout: page
title: Cadmus Development
subtitle: Summary
---

To create a Cadmus editor for your project, you need a backend and a frontend. For the most part, you will end up just customizing the provided code templates, so you do not need advanced programming skills to setup an editor. Of course, a minimal familiarity with the languages and technologies involved is required.

You start with creating backend core libraries, eventually adding your own parts and fragments, with their seeders and tests. Then, you add a couple of services which put all the pieces together, and are consumed by the API layer. Once done, you create the backend API by just assembling the pieces you created with those coming from Cadmus infrastructure.

Then, you create an Angular workspace for your frontend app, eventually adding the editors corresponding to your own parts and fragments. Apart from this, the app is built by just assembling pieces from the Cadmus infrastructure.

👉 Setup a Cadmus [development environment](devenv.md) and [Docker](../docker-setup.md).

## Models

- [general models](models.md)

## Backend

1. [core](backend/core.md): core backend solution. At a minimum, this typically includes the [core services](backend/services.md), even when you are just using existing parts/fragments, unless you prefer to place them directly inside your API.
2. [parts](backend/parts.md): only if you need to add your own custom parts.
3. [part seeders](backend/part-seeders.md): only if you need to add your own custom parts.
4. [fragments](backend/fragments.md): only if you need to add your own custom fragments.
5. [fragment seeders](backend/fragment-seeders.md): only if you need to add your own custom fragments.
6. [services](backend/services.md)
7. [API](backend/api.md)

## Frontend

1. [app](frontend/app-setup.md)
2. [libraries](frontend/libs.md)
3. [parts](frontend/parts.md): only if you need to add your own custom parts.
4. [fragments](frontend/fragments.md): only if you need to add your own custom fragments.

## Concepts

- [static data lookup: thesauri](concepts/thesauri.md)
- [dynamic data lookup strategies](concepts/lookup.md)
- [semantic graph](concepts/graph.md)
- [layers reconciliation](concepts/layer-reconciliation.md)
- [data migration](https://github.com/vedph/cadmus-migration/blob/master/docs/index.md): importing and exporting Cadmus data.

## Deployment

- [deployment](deploy.md)

## History

📆 [history](history.md): this lists the main changes in the software ecosystem during Cadmus development. Refer to this section if you are going to migrate some old code into releases with relevant breaking changes.
