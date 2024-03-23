---
layout: page
title: Frontend App - Setup
subtitle: "Cadmus Frontend Development"
---

📌 Task: create a Cadmus frontend web app in a new Angular workspace, setting up its general infrastructure.

1. **app**
2. [libraries](libs.md)
3. [parts](parts.md) (optional)
4. [fragments](fragments.md) (optional)

The typical steps for developing a Cadmus frontend (as based on the [reference shell](https://github.com/vedph/cadmus-shell-2)) are:

1. create an Angular app.
2. install NPM packages.
3. adding and customizing template code.
4. optionally (if you are using your own models) [add parts](frontend-part.md) and/or fragments.

## Requirements

- [NodeJS](https://nodejs.org/en/download/)
- Angular CLI
- a code editor like [VSCode](https://code.visualstudio.com/)
- familiarity with HTML, CSS, Typescript and Angular

## Create Angular App

(1) create a **new Angular app**: `ng new cadmus-<PRJ>-app`: when prompted, add Angular routing and use CSS (you may use SCSS if you prefer, or if you want to customize your theme).

>If you are creating an app for the only purpose of developing component libraries in it, our convention is naming it as `-shell` rather than `-app`.

(2) enter the newly created directory and **add Angular Material** (choose the Indigo/Pink theme - or whatever you prefer -, setup typography styles=yes, include and enable animations=yes) and **Angular localization package**:

```bash
ng add @angular/material
ng add @angular/localize
```

>The localization package is a development package which is required by some localization-ready components such as the authentication libraries (`@myrmidon/auth-jwt-*`). You can also just add the NPM package via `npm -i --save-dev @angular/localize`.

## Install Packages

(3) install the typical Cadmus packages via NPM:

```bash
npm i @auth0/angular-jwt @myrmidon/auth-jwt-admin @myrmidon/auth-jwt-login

npm i @myrmidon/cadmus-api @myrmidon/cadmus-core @myrmidon/cadmus-graph-ui @myrmidon/cadmus-graph-pg

npm i @myrmidon/cadmus-item-editor @myrmidon/cadmus-item-list @myrmidon/cadmus-item-search

npm i @myrmidon/cadmus-part-general-pg @myrmidon/cadmus-part-general-ui

npm i @myrmidon/cadmus-part-philology-pg @myrmidon/cadmus-part-philology-ui

npm i @myrmidon/cadmus-preview-pg @myrmidon/cadmus-preview-ui @myrmidon/cadmus-profile-core

npm i @myrmidon/cadmus-refs-asserted-chronotope @myrmidon/cadmus-flags-pg @myrmidon/cadmus-flags-ui @myrmidon/cadmus-refs-asserted-ids @myrmidon/cadmus-refs-assertion @myrmidon/cadmus-refs-decorated-ids @myrmidon/cadmus-refs-doc-references @myrmidon/cadmus-refs-external-ids @myrmidon/cadmus-refs-historical-date @myrmidon/cadmus-mat-physical-size @myrmidon/cadmus-refs-lookup @myrmidon/cadmus-refs-proper-name @myrmidon/cadmus-state @myrmidon/cadmus-text-block-view @myrmidon/cadmus-thesaurus-editor @myrmidon/cadmus-thesaurus-list @myrmidon/cadmus-thesaurus-ui @myrmidon/cadmus-ui @myrmidon/cadmus-ui-flags-picker @myrmidon/cadmus-ui-pg @myrmidon/ng-mat-tools @myrmidon/ng-tools @myrmidon/paged-data-browsers @myrmidon/ngx-dirty-check @types/diff-match-patch diff-match-patch gravatar
```

The above packages are fairly typical, but you might well omit those you are not interested in, e.g. general parts or philology parts, or some of the bricks. Some of the legacy third party libraries may require `--force`.

Typically you will also need **Monaco editor** and **Markdown**:

- [ngx-markdown](https://github.com/jfcere/ngx-markdown) if you have components _displaying_ Markdown.
- [ngx-monaco-editor](https://github.com/atularen/ngx-monaco-editor) or [this more recent version](https://github.com/miki995/ngx-monaco-editor-v2) if you components _using_ Markdown (like e.g. the general note part) or other languages in a Monaco-based editor.

Please be sure to _follow the directions provided by each library_ when installing it.

## Set Environment Variables

This is essential to let the frontend find the server, while allowing us to manually edit this URI after building a distribution, and before creating a Docker image.

(1) under `src` add an `env.js` file for project-dependent environment variables, with this content (replace the port number, in this sample 60849, with your backend API port number):

```js
// https://www.jvandemo.com/how-to-use-environment-variables-to-configure-your-angular-application-without-a-rebuild/
(function (window) {
  window.__env = window.__env || {};

  // environment-dependent settings
  window.__env.apiUrl = "http://localhost:60849/api/";
  window.__env.version = "0.0.1";
  // enable thesaurus import in thesaurus list for admins
  window.__env.thesImportEnabled = true;
})(this);
```

>💡 You might need additional settings here, like e.g. a Mapbox GL API token if using geographic components.

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
    "input": "node_modules/monaco-editor",
    "output": "/assets/monaco/"
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

## Fine-Tune Angular Settings

This is suggested to enable source maps in production and avoid nasty warnings after compilation.

(1) in `angular.json`: following the suggestions in <https://stackoverflow.com/questions/54891679/how-do-i-get-source-map-working-for-npm-linked-angular-library>, and the [Angular docs](https://angular.io/guide/workspace-config#optimization-and-source-map-configuration), explicitly opt for the source maps (under `projects/app/architect/build/options`):

```json
"sourceMap": {
  "scripts": true,
  "hidden": false,
  "vendor": true,
  "styles": true,
},
```

(2) in the same file, you will typically have to raise the warning limits for your `budget` size if getting a warning after building.

## Add Assets

This is optional and depends on your visuals.

For a quick setup, just ensure you have the required icons and images in `assets` (see `cadmus-shell/assets`): usually they are `logo-white-40.png` for the top bar logo (you can use your own), and a couple of banner images for the homepage (`banner-512.jpg`, `banner-1024.jpg`).

The logo is used in the `app.component`'s template for the main toolbar, while banner images are used in the default homepage placeholder.

## Add Cadmus Infrastructure

(1) add some extension points, eventually adding new entries for your new parts (see [dynamic lookup](https://github.com/vedph/cadmus_doc/blob/master/core/dynamic-lookup.md)):

- `src/app/index-lookup-definitions.ts` with this content:

```ts
import { IndexLookupDefinitions } from '@myrmidon/cadmus-core';

export const INDEX_LOOKUP_DEFINITIONS : IndexLookupDefinitions = {}
```

- `src/app/item-browser-keys.ts` with this content:

```ts
/**
 * Mapping between item browser keys and their routes, used to avoid
 * long and complex names in the route by replacing the ID with an alias.
 */
export const ITEM_BROWSER_KEYS = {
// e.g. ['it.vedph.item-browser.mongo.hierarchy']: 'hierarchy'
};
```

- `src/app/part-editor-keys.ts`: this is the only file with a real content, the others being just extension points. You must specify here the connection of each part or fragment ID with its hosting library in constant `PART_EDITOR_KEYS`. This object has a property named after each part/fragment type ID, with a value equal to an object with `part` equal to the library ID, and optionally `fragments` (when the part is a layer part). This property is an object with a property for each fragment type for the layer part, named after the fragment type ID, with a value equal to the library ID.

(2) copy these folders (each corresponding to an app's page component) into your app's `src/app` folder from the [reference project](https://github.com/vedph/cadmus-shell-2):

- `home` (adjust the homepage contents according to your project)
- `login-page`
- `manage-users-page`
- `register-user-page`
- `reset-password`

As currently the shell app still has module-based components, patch these with the following imports for each component:

**login-page**:

```ts
import { CommonModule } from '@angular/common';
import { Component, OnInit } from '@angular/core';
import { Router } from '@angular/router';
import { ReactiveFormsModule } from '@angular/forms';

import { MatButtonModule } from '@angular/material/button';
import { MatCardModule } from '@angular/material/card';
import { MatIconModule } from '@angular/material/icon';
import { MatFormFieldModule } from '@angular/material/form-field';
import { MatInputModule } from '@angular/material/input';
import { MatTooltipModule } from '@angular/material/tooltip';

import {
  AuthJwtLoginModule,
  AuthJwtService,
  Credentials,
} from '@myrmidon/auth-jwt-login';

@Component({
  selector: 'app-login-page',
  standalone: true,
  templateUrl: './login-page.component.html',
  styleUrls: ['./login-page.component.css'],
  imports: [
    CommonModule,
    ReactiveFormsModule,
    MatButtonModule,
    MatCardModule,
    MatFormFieldModule,
    MatIconModule,
    MatInputModule,
    MatTooltipModule,
    AuthJwtLoginModule,
  ],
})
// ... rest of code
```

**manage-users-page**:

```ts
import { CommonModule } from '@angular/common';
import { Component, OnInit } from '@angular/core';
import { ReactiveFormsModule } from '@angular/forms';

import { MatButtonModule } from '@angular/material/button';
import { MatCardModule } from '@angular/material/card';
import { MatFormFieldModule } from '@angular/material/form-field';
import { MatIconModule } from '@angular/material/icon';
import { MatInputModule } from '@angular/material/input';
import { MatProgressBarModule } from '@angular/material/progress-bar';
import { MatTooltipModule } from '@angular/material/tooltip';

import { AuthJwtAdminModule } from '@myrmidon/auth-jwt-admin';

@Component({
  selector: 'app-manage-users-page',
  standalone: true,
  templateUrl: './manage-users-page.component.html',
  styleUrls: ['./manage-users-page.component.css'],
  imports: [
    CommonModule,
    ReactiveFormsModule,
    MatButtonModule,
    MatCardModule,
    MatFormFieldModule,
    MatIconModule,
    MatInputModule,
    MatProgressBarModule,
    MatTooltipModule,
    AuthJwtAdminModule,
  ],
})
// ... rest of code
```

**register-user-page**:

```ts
import { Component, OnInit } from '@angular/core';
import { Router } from '@angular/router';
import { CommonModule } from '@angular/common';
import { ReactiveFormsModule } from '@angular/forms';

import { MatButtonModule } from '@angular/material/button';
import { MatCardModule } from '@angular/material/card';
import { MatFormFieldModule } from '@angular/material/form-field';
import { MatIconModule } from '@angular/material/icon';
import { MatInputModule } from '@angular/material/input';
import { MatProgressBarModule } from '@angular/material/progress-bar';
import { MatTooltipModule } from '@angular/material/tooltip';

import { AuthJwtAdminModule } from '@myrmidon/auth-jwt-admin';

@Component({
  selector: 'app-register-user-page',
  standalone: true,
  templateUrl: './register-user-page.component.html',
  styleUrls: ['./register-user-page.component.css'],
  imports: [
    CommonModule,
    ReactiveFormsModule,
    MatButtonModule,
    MatCardModule,
    MatFormFieldModule,
    MatIconModule,
    MatInputModule,
    MatProgressBarModule,
    MatTooltipModule,
    AuthJwtAdminModule,
  ],
})
// ... rest of code
```

**reset-password**:

```ts
import { CommonModule } from '@angular/common';
import { Component } from '@angular/core';
import {
  FormGroup,
  FormControl,
  FormBuilder,
  Validators,
  ReactiveFormsModule,
} from '@angular/forms';

import { MatButtonModule } from '@angular/material/button';
import { MatCardModule } from '@angular/material/card';
import { MatFormFieldModule } from '@angular/material/form-field';
import { MatIconModule } from '@angular/material/icon';
import { MatInputModule } from '@angular/material/input';
import { MatProgressBarModule } from '@angular/material/progress-bar';
import { MatSnackBar } from '@angular/material/snack-bar';
import { MatTooltipModule } from '@angular/material/tooltip';

import { AuthJwtAccountService, AuthJwtAdminModule } from '@myrmidon/auth-jwt-admin';

@Component({
  selector: 'cadmus-reset-password',
  standalone: true,
  templateUrl: './reset-password.component.html',
  styleUrls: ['./reset-password.component.css'],
  imports: [
    CommonModule,
    ReactiveFormsModule,
    MatButtonModule,
    MatCardModule,
    MatFormFieldModule,
    MatIconModule,
    MatInputModule,
    MatProgressBarModule,
    MatTooltipModule,
    AuthJwtAdminModule,
  ]
})
// ... rest of code
```

## Add Docker Support

You can add Docker support to create an image of your frontend app. Use as templates the files in the reference shell app:

- `Dockerfile` using NGINX to serve the Angular app:

```dockerfile
FROM nginx:alpine

COPY nginx.conf /etc/nginx/nginx.conf
RUN rm /etc/nginx/conf.d/default.conf

WORKDIR /usr/share/nginx/html
COPY dist/browser/cadmus-__PRJ__-app/ .

EXPOSE 80
```

- `nginx.conf`: the NGINX configuration for serving the web app from the Docker container:

```conf
user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log warn;
pid /var/run/nginx.pid;

events {
  worker_connections 1024;
}

http {
  include /etc/nginx/mime.types;
  default_type application/octet-stream;

  log_format main '$remote_addr - $remote_user [$time_local] "$request" '
  '$status $body_bytes_sent "$http_referer" '
  '"$http_user_agent" "$http_x_forwarded_for"';

  access_log /var/log/nginx/access.log main;

  sendfile on;

  keepalive_timeout 65;

  server {
    listen 80;
    listen [::]:80;
    server_name localhost;

    gzip on;
    gzip_min_length 1000;
    gzip_proxied expired no-cache no-store private auth;
    gzip_types text/plain text/css application/json application/javascript application/x-javascript text/xml application/xml application/xml+rss text/javascript;

    location / {
      root /usr/share/nginx/html;
      index index.html index.htm;
      try_files $uri $uri/ /index.html;
    }

    error_page 500 502 503 504 /50x.html;
    location = /50x.html {
      root /usr/share/nginx/html;
    }
  }
}
```

- `docker-compose.yml`: _customize this_ for the image names and versions:

```yml
version: "3.7"

services:
  # MongoDB
  cadmus-__PRJ__-mongo:
    image: mongo
    container_name: cadmus-__PRJ__-mongo
    environment:
      - MONGO_DATA_DIR=/data/db
      - MONGO_LOG_DIR=/dev/null
    command: mongod --logpath=/dev/null
    ports:
      - 27017:27017
    networks:
      - cadmus-__PRJ__-network

 # PostgreSQL
  cadmus-__PRJ__-pgsql:
    image: postgres
    container_name: cadmus-__PRJ__-pgsql
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres
      - POSTGRES_DB=postgres
    ports:
      - 5432:5432
    networks:
      - cadmus-__PRJ__-network

  # Cadmus __PRJ__ API
  cadmus-__PRJ__-api:
    image: vedph2020/cadmus-__PRJ__-api:0.0.1
    container_name: cadmus-__PRJ__-api
    ports:
      # TODO: change 5080 with your API port in the host
      - 5080:80
    depends_on:
      - cadmus-__PRJ__-mongo
      - cadmus-__PRJ__-pgsql
    environment:
      - CONNECTIONSTRINGS__DEFAULT=mongodb://cadmus-__PRJ__-mongo:27017/{0}
      - CONNECTIONSTRINGS__INDEX=Server=cadmus-__PRJ__-pgsql;port=5432;Database={0};User Id=postgres;Password=postgres;Include Error Detail=True
      - SERILOG__CONNECTIONSTRING=mongodb://cadmus-__PRJ__-mongo:27017/{0}-log
      - STOCKUSERS__0__PASSWORD=P4ss-W0rd!
      - SEED__INDEXDELAY=25
      - MESSAGING__APIROOTURL=http://cadmusapi.azurewebsites.net
      - MESSAGING__APPROOTURL=http://cadmusapi.com/
      - MESSAGING__SUPPORTEMAIL=support@cadmus.com
    networks:
      - cadmus-__PRJ__-network

  # Cadmus __PRJ__ App
  cadmus-app:
    image: vedph2020/cadmus-__PRJ__-app:0.0.1
    container_name: cadmus-__PRJ__-app
    ports:
      - 4200:80
    depends_on:
      - cadmus-__PRJ__-api
    networks:
      - cadmus-__PRJ__-network

networks:
  cadmus-__PRJ__-network:
    driver: bridge
```

- `dockerignore`:

```txt
e2e
node_modules
src
```

## Add Readme

Finally you can use a README template like this:

```md
- [models](https://github.com/vedph/cadmus-__PRJ__)
- [API](https://github.com/vedph/cadmus-__PRJ__-api)

## Docker

🐋 Quick Docker image build:

1. `npm run build-lib`
2. update version in `env.js` and `ng build`
3. `docker build . -t vedph2020/cadmus-__PRJ__-app:0.0.1 -t vedph2020/cadmus-__PRJ__-app:latest` (replace with the current version).
```

🏠 [developer's home](../toc.md)

▶️ next: [app components](app-components.md)
