---
layout: page
title: Frontend
subtitle: "Cadmus Development"
---

- [Requirements](#requirements)
- [Create Angular App](#create-angular-app)
- [Install Packages](#install-packages)
- [Setting Environment Variables](#setting-environment-variables)
- [Tuning Angular Settings](#tuning-angular-settings)
- [Assets](#assets)

The typical steps for developing a Cadmus frontend (as based on the [reference shell](https://github.com/vedph/cadmus-shell-2)) are:

1. create an Angular app.
2. install NPM packages.
3. adding and customizing template code.

## Requirements

- [NodeJS](https://nodejs.org/en/download/)
- Angular CLI
- a code editor like [VSCode](https://code.visualstudio.com/)
- familiarity with HTML, CSS, Typescript and Angular

## Create Angular App

(1) create a new Angular app: `ng new cadmus-<PRJ>-app`: when prompted, add Angular routing and use CSS (you may use SCSS if you prefer, or if you want to customize your theme).

(2) enter the newly created directory and add Angular Material via `ng add @angular/material` (choose the Indigo/Pink theme - or whatever you prefer -, setup typography styles=yes, include and enable animations=yes).

(3) (optional) if adding new parts/fragments, you will need to add libraries for them. Each library is added like `ng generate library @myrmidon/cadmus-<PRJ>-<NAME> --prefix <PRJ>`. The typical structure usually includes these libraries (`@myrmidon` here is my NPM name, used as a namespace for all my Cadmus libraries):

- `@myrmidon/cadmus-<PRJ>-part-ui`: parts and fragments editors.
- `@myrmidon/cadmus-<PRJ>-part-pg`: parts and fragments editors wrappers with routing.

Eventually, for more complex projects you can add other libraries like:

- `@myrmidon/cadmus-<PRJ>-core` (optional): core models and eventually services, shared by several parts of the same project.
- `@myrmidon/cadmus-<PRJ>-ui` (optional): UI components shared by several parts of the same project.

Alternatively, a more granular architecture can be used if you plan to create part/fragment editors to be reused in other projects. In this case, you can create a library for each single editor. If you later realize that some of the editors you created can be promoted to a higher level of abstraction for sharing them across projects, you can create new libraries in a distinct shell, and then replace the original ones in your app project by importing these new libraries.

(4) to speed up builds, for each added library you can add the corresponding commands to `package.json` scripts (to be run like `npm run <SCRIPTNAME>`), e.g.:

```json
{
  "build-ui": "ng build @myrmidon/cadmus-__PRJ__-part-ui",
  "build-pg": "ng build @myrmidon/cadmus-__PRJ__-part-pg",
  "build-all": "npm run-script build-ui && npm run-script build-pg"
}
```

## Install Packages

Open a terminal in the app's folder and issue the following commands.

(1) install ELF with command `npx @ngneat/elf-cli install`. When prompted, install:

- @ngneat/elf
- @ngneat/elf-entities
- @ngneat/elf-devtools
- @ngneat/elf-requests
- @ngneat/elf-pagination
- @ngneat/elf-cli-ng

No external package is needed. Should you prefer, you can just use NPM to install these packages:

```bash
npm i @ngneat/elf @ngneat/elf-entities @ngneat/elf-devtools @ngneat/elf-requests @ngneat/elf-pagination @ngneat/elf-cli-ng --force
```

(2) install the typical Cadmus packages via NPM:

```bash
npm i @auth0/angular-jwt @myrmidon/auth-jwt-admin @myrmidon/auth-jwt-login @myrmidon/cadmus-api @myrmidon/cadmus-core @myrmidon/cadmus-item-editor @myrmidon/cadmus-item-list @myrmidon/cadmus-item-search @myrmidon/cadmus-login @myrmidon/cadmus-part-general-pg @myrmidon/cadmus-part-general-ui @myrmidon/cadmus-part-philology-pg @myrmidon/cadmus-part-philology-ui @myrmidon/cadmus-preview-pg @myrmidon/cadmus-preview-ui @myrmidon/cadmus-profile-core @myrmidon/cadmus-refs-asserted-chronotope @myrmidon/cadmus-refs-asserted-ids @myrmidon/cadmus-refs-assertion @myrmidon/cadmus-refs-decorated-ids @myrmidon/cadmus-refs-doc-references @myrmidon/cadmus-refs-external-ids @myrmidon/cadmus-refs-historical-date @myrmidon/cadmus-refs-proper-name @myrmidon/cadmus-state @myrmidon/cadmus-text-block-view @myrmidon/cadmus-thesaurus-editor @myrmidon/cadmus-thesaurus-list @myrmidon/cadmus-thesaurus-ui @myrmidon/cadmus-ui @myrmidon/cadmus-ui-flags-picker @myrmidon/cadmus-ui-pg @myrmidon/ng-mat-tools @myrmidon/ng-tools @types/diff-match-patch diff-match-patch gravatar ngx-markdown ngx-monaco-editor rangy --force
```

The above packages are fairly typical, but you might well omit those you are not interested in, e.g. general parts or philology parts, or some of the bricks. Some of the legacy third party libraries like rangy may require `--force`.

Typically you will also need `ngx-markdown` if you have components _displaying_ Markdown, and `ngx-monaco` if you components _using_ Markdown or other languages in a Monaco-based editor. Please be sure to follow the directions provided by each library when installing it. For instance, [ngx-markdown](https://github.com/jfcere/ngx-markdown) requires these packages:

```bash
npm install ngx-markdown marked --force
npm install @types/marked --save-dev --force
```

Additionally, as the library is using Marked parser you will need to add `node_modules/marked/marked.min.js` to your application. If you are using Angular CLI you can add to `scripts`:

```json
"scripts": [
 "node_modules/marked/marked.min.js"
]
```

Also, you must be sure to import `MarkdownModule.forRoot()` in your app module, as `forRoot` injects the required `MarkdownService`.

## Setting Environment Variables

This is essential to let the frontend find the server, while allowing us to manually edit this URI after building a distribution, and before creating a Docker image.

(1) under `src` add an `env.js` file for project-dependent environment variables, with this content (replace the port number, in this sample 60849, with your backend API port number):

```js
// https://www.jvandemo.com/how-to-use-environment-variables-to-configure-your-angular-application-without-a-rebuild/
(function (window) {
  window.__env = window.__env || {};

  // environment-dependent settings
  window.__env.apiUrl = "http://localhost:60849/api/";
  window.__env.version = "1.0.0";
})(this);
```

If you are going to use the [external bibliography API](https://github.com/vedph/cadmus_biblioapi), also add its URL here, e.g.:

```js
window.__env.biblioApiUrl = 'http://localhost:61691/api/';
```

In this case typically you will also need to install `@myrmidon/cadmus-biblio-core @myrmidon/cadmus-biblio-api @myrmidon/cadmus-biblio-ui @myrmidon/cadmus-part-biblio-ui`, which provide the corresponding frontend. Later, in your app's `part-editor-keys.ts`, remember to setup the route to the bibliography part editor like:

```ts
import { EXT_BIBLIOGRAPHY_PART_TYPEID } from '@myrmidon/cadmus-part-biblio-ui';

// ...

export const PART_EDITOR_KEYS: PartEditorKeys = {
  [EXT_BIBLIOGRAPHY_PART_TYPEID]: {
    part: BIBLIO,
  },
  // ... etc.
};
```

(2) in `angular.json`, under `projects/APPNAME/architect/build/options/assets`:

- add `"src/env.js"`.
- if you are using the Monaco editor, add a glob for it. The result would be something like this (see <https://www.npmjs.com/package/ngx-monaco-editor>):

```json
"assets": [
  "src/favicon.ico",
  "src/assets",
  "src/env.js",
  {
    "glob": "**/*",
    "input": "node_modules/ngx-monaco-editor/assets/monaco",
    "output": "/assets/monaco"
  }
],
```

(3) in `src/index.html` add an import for `env.js` to your `head` element:

```html
<head>
  ...
  <script src="env.js"></script>
</head>
```

Also, you can change the web app's `title` in `head` to a more human friendly name.

## Tuning Angular Settings

This is suggested to enable source maps in production and avoid nasty warnings after compilation.

(1) in `angular.json`: following the suggestions in <https://stackoverflow.com/questions/54891679/how-do-i-get-source-map-working-for-npm-linked-angular-library>, and the [Angular docs](https://angular.io/guide/workspace-config#optimization-and-source-map-configuration), explicitly opt for the source maps (under `projects/app/architect/build/options`):

```json
"sourceMap": {
  "scripts": true,
  "styles": true,
  "hidden": false,
  "vendor": true
},
```

(2) in the same file, you will typically have to raise the warning limits for your `budget` size if getting a warning after building.

## Assets

This is optional and depends on your visuals.

For a quick setup, just ensure you have the required icons and images in `assets` (see `cadmus-shell/assets`): usually they are `logo-white-40.png` for the top bar logo (you can use your own), and a couple of banner images for the homepage (`banner-512.jpg`, `banner-1024.jpg`).

The logo is used in the `app.component`'s template for the main toolbar, while banner images are used in the default homepage placeholder.
