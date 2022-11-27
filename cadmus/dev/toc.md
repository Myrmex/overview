---
layout: page
title: Cadmus Development
subtitle: Summary
---

To create a Cadmus editor for your project, you need a backend and a frontend. For the most part, you will end up just customizing the provided code templates, so you do not need advanced programming skills to setup an editor. Of course, a minimal familiarity with the languages and technologies involved is required.

## Environment

Cadmus has a layered architecture, essentially relying on .NET (C#) for the backend core and API, and on Angular 15+ (Typescript) for the frontend. This implies setting up a full-stack [development environment](devenv.md), including:

- MongoDB
- MySql
- [NodeJS](https://nodejs.org/en/download/)
- [Angular CLI](https://github.com/angular/angular-cli)
- [Visual Studio Community Edition](https://visualstudio.microsoft.com/vs/community/) or higher.
- a code editor like [VSCode](https://code.visualstudio.com/)
- [Docker](../docker-setup.md)

## Summary

You start with creating backend core libraries, eventually adding your own parts and fragments, with their seeders and tests. Then, you add a couple of services which put all the pieces together, and are consumed by the API layer. Once done, you create the backend API by just assembling the pieces you created with those coming from Cadmus infrastructure.

Then, you create an Angular workspace for your frontend app, eventually adding the editors corresponding to your own parts and fragments. Apart from this, the app is built by just assembling pieces from the Cadmus infrastructure.

1. [backend](backend.md)
   1. [core data models](backend-core.md) (optional)
      1. [parts](backend-part.md)
      2. [part seeders](backend-part-seeder.md)
      3. [fragments](backend-fragment.md)
      4. [fragment seeders](backend-fragment-seeder.md)
      5. [services](backend-core-svc.md)
   2. [backend services](backend-core-svc.md)
   3. [backend API](backend-api.md)
2. [frontend](frontend.md)
   1. [parts](frontend-part.md) (optional)
   2. [fragments](frontend-fragment.md) (optional)
