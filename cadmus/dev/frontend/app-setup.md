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

(1) create a **new Angular app**: `ng new cadmus-<PRJ>-app`: when prompted, add Angular routing and use SCSS. If prompted, don't enable SSR (as per default option).

>If you are creating an app for the only purpose of developing component libraries in it, our convention is naming it as `-shell` rather than `-app`.

(2) enter the newly created directory and **add Angular Material** (choose the Indigo/Pink theme - or whatever you prefer -, setup typography styles=yes, include and enable animations=yes) and **Angular localization package**:

```bash
ng add @angular/material
ng add @angular/localize
```

For Angular Material, pick the theme you prefer, answer Yes when prompted to setup global typography styles, and accept the default "Include and enable animations" option.

>The localization package is a development package which is required by some localization-ready components such as the authentication libraries (`@myrmidon/auth-jwt-*`). You can also just add the NPM package via `npm -i --save-dev @angular/localize`.

(2) ensure to apply some [M3 theme](https://material.angular.io/guide/theming) in your app's `styles.scss`, like in this example:

```scss
@use "@angular/material" as mat;

@include mat.core();

$light-theme: mat.define-theme(
  (
    color: (
      theme-type: light,
      primary: mat.$azure-palette,
    ),
  )
);

html {
  @include mat.all-component-themes($light-theme);
}
```

💡 If you are dealing with an existing app using CSS rather than SCSS:

1. rename `styles.css` to `styles.scss`.
2. in `angular.json`, rename `styles.css` in `styles.scss` in the `styles` array.
3. configure Angular CLI to Use SCSS for new components: in `angular.json`, locate the `schematics` section. If it doesn't exist, add it under your project's root. Add or update the `@schematics/angular:component` section to specify scss as the default style extension:

```json
"schematics": {
  "@schematics/angular:component": {
    "style": "scss"
  }
}
```

## Install Packages

(3) install the typical Cadmus packages via NPM:

```bash
npm i @auth0/angular-jwt @myrmidon/auth-jwt-admin @myrmidon/auth-jwt-login

npm i @myrmidon/cadmus-api @myrmidon/cadmus-core @myrmidon/cadmus-graph-ui @myrmidon/cadmus-graph-pg @myrmidon/cadmus-item-editor @myrmidon/cadmus-item-list @myrmidon/cadmus-item-search

npm i @myrmidon/cadmus-part-general-pg @myrmidon/cadmus-part-general-ui

npm i @myrmidon/cadmus-part-philology-pg @myrmidon/cadmus-part-philology-ui

npm i @myrmidon/cadmus-preview-pg @myrmidon/cadmus-preview-ui @myrmidon/cadmus-profile-core

npm i @myrmidon/cadmus-refs-asserted-chronotope @myrmidon/cadmus-flags-pg @myrmidon/cadmus-flags-ui @myrmidon/cadmus-refs-asserted-ids @myrmidon/cadmus-refs-assertion @myrmidon/cadmus-refs-decorated-ids @myrmidon/cadmus-refs-doc-references @myrmidon/cadmus-refs-external-ids @myrmidon/cadmus-refs-historical-date @myrmidon/cadmus-mat-physical-size @myrmidon/cadmus-refs-lookup @myrmidon/cadmus-refs-proper-name @myrmidon/cadmus-state @myrmidon/cadmus-text-block-view @myrmidon/cadmus-thesaurus-editor @myrmidon/cadmus-thesaurus-list @myrmidon/cadmus-thesaurus-ui @myrmidon/cadmus-ui @myrmidon/cadmus-ui-flags-picker @myrmidon/cadmus-ui-pg @myrmidon/ng-mat-tools @myrmidon/ng-tools @myrmidon/paged-data-browsers @types/diff-match-patch diff-match-patch ts-md5

npm i @myrmidon/cadmus-text-ed @myrmidon/cadmus-text-ed-md @myrmidon/cadmus-text-ed-txt
```

The above packages are fairly typical, but you might well omit those you are not interested in, e.g. general parts or philology parts, or some [bricks](https://github.com/vedph/cadmus-bricks-shell-v2). Some of the legacy third party libraries may require `--force`.

Typically you will also need **Monaco editor** and **Markdown**:

- [NG essentials](https://github.com/cisstech/nge): `npm i @cisstech/nge monaco-editor`.
- [ngx-markdown](https://github.com/jfcere/ngx-markdown) if you have components _displaying_ Markdown: `npm i ngx-markdown marked`.

>Even though you usually all what you have to do is installing the listed packages, be sure to _follow the directions provided by each library_ when installing it.

## Set Environment Variables

This is essential to let the frontend find the server, while allowing us to manually edit this URI after building a distribution, and before creating a Docker image.

(1) under `src` (or under `public` if using templates from Angular 18 onwards) add an `env.js` file for project-dependent environment variables, with this content (replace the port number, in this sample 60849, with your backend API port number):

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

>💡 You might need additional settings here, like e.g. a Mapbox GL API token, a Geonames account name, etc.

📖 If you are going to use the [external bibliography API](https://github.com/vedph/cadmus_biblioapi), also add its URL here, e.g.:

```js
window.__env.biblioApiUrl = 'http://localhost:60058/api/';
```

In this case typically you will also need to install the bibliography packages:

```bash
npm i @myrmidon/cadmus-biblio-core @myrmidon/cadmus-biblio-api @myrmidon/cadmus-biblio-ui @myrmidon/cadmus-part-biblio-ui
```

Later, in your app's `part-editor-keys.ts`, remember to _setup the route to the bibliography part editor_ like:

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

- add `"src/env.js"`:

```json
"assets": [
  "src/favicon.ico",
  "src/assets",
  "src/env.js"
],
```

⚠️ Since Angular 18 the `public` folder is the place where you should place items to be copied. So, in this case just place the `env.js` file there. No change is required in `angular.json` because it already has a glob catch-all pattern pointing to the `public` folder.

>The glob for Monaco editor is no longer needed when using NG essentials as a Monaco wrapper.

💡 If you are using legacy libraries like `gravatar`, add this option under `architect/build/options` to avoid a build warning:

```json
"allowedCommonJsDependencies": [
  "gravatar"
]
```

(3) in `src/index.html` add an import for `env.js` to your `head` element:

```html
<head>
  ...
  <script src="env.js"></script>
</head>
```

Also, you can change the web app's `title` in `head` to a more human-friendly name.

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

>⚠️ Since Angular 18 you can omit this step.

(2) in the same file, you will typically have to raise the warning limits for your `budget` size if getting a warning after building.

## Add Assets

This is optional and depends on your visuals.

For a quick setup, just ensure you have the required icons and images in `assets` (see `cadmus-shell/assets`): usually they are `logo-white-40.png` for the top bar logo (you can use your own), and a couple of banner images for the homepage (`banner-512.jpg`, `banner-1024.jpg`).

⚠️ Since Angular 18, you can rather place these assets under your `public/img` folder. This implies removing the `assets` folder from paths in the code templates if any, e.g. `/assets/img/some-image.jpg` becomes `/img/some-image.jpg`.

The logo is used in the `app.component`'s template for the main toolbar, while banner images are used in the default homepage placeholder.

>💡 If using [lookup sets](https://github.com/vedph/cadmus-bricks-shell-v2/blob/master/projects/myrmidon/cadmus-refs-lookup/README.md#lookup-set), typically you will also need an icon for each lookup source, e.g. VIAF, GeoNames, etc. You can find some of these icons in the image folder of the [Cadmus shell app](https://github.com/vedph/cadmus-shell-v3/tree/master/src/assets/img).

## Add Cadmus Infrastructure

(1) add some extension points, eventually adding new entries for your new parts (see [dynamic lookup](https://github.com/vedph/cadmus_doc/blob/master/core/dynamic-lookup.md)):

- `src/app/index-lookup-definitions.ts` with this content:

```ts
import { IndexLookupDefinitions } from '@myrmidon/cadmus-core';

export const INDEX_LOOKUP_DEFINITIONS : IndexLookupDefinitions = {}
```

You can add to the definitions object all the pin-based lookup definitions you will need in your project, e.g.:

```ts
// ...
import { METADATA_PART_TYPEID } from '@myrmidon/cadmus-part-general-ui';

export const INDEX_LOOKUP_DEFINITIONS: IndexLookupDefinitions = {
  // item's metadata
  meta_eid: {
    typeId: METADATA_PART_TYPEID,
    name: 'eid',
  },
};
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

>⚠️ If using external bibliography, remember to [add the route](#set-environment-variables) for its editor.

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

## Implement App Component

The default app component must be updated with a code like this (remove the lookup services and/or the text plugins if not using them):

```ts
// app.component.ts

import { Component, Inject, OnDestroy, OnInit } from '@angular/core';
import { CommonModule } from '@angular/common';
import { Router, RouterModule, RouterOutlet } from '@angular/router';
import { Subscription } from 'rxjs';

import { MatButtonModule } from '@angular/material/button';
import { MatIconModule } from '@angular/material/icon';
import { MatMenuModule } from '@angular/material/menu';
import { MatToolbarModule } from '@angular/material/toolbar';

// myrmidon
import {
  EnvService,
  EnvServiceProvider,
  RamStorageService,
} from '@myrmidon/ng-tools';
import {
  User,
  AuthJwtService,
  GravatarService,
} from '@myrmidon/auth-jwt-login';

// bricks
import {
  ASSERTED_COMPOSITE_ID_CONFIGS_KEY,
  AssertedCompositeIdsComponent,
} from '@myrmidon/cadmus-refs-asserted-ids';
import { DocReferencesComponent } from '@myrmidon/cadmus-refs-doc-references';
import {
  HistoricalDateComponent,
  HistoricalDatePipe,
} from '@myrmidon/cadmus-refs-historical-date';
import { FlagsPickerComponent } from '@myrmidon/cadmus-ui-flags-picker';
import { ViafRefLookupService } from '@myrmidon/cadmus-refs-viaf-lookup';
import { DbpediaRefLookupService } from '@myrmidon/cadmus-refs-dbpedia-lookup';
import { GeoNamesRefLookupService } from '@myrmidon/cadmus-refs-geonames-lookup';

// cadmus
import {
  CadmusCoreModule,
  Thesaurus,
  ThesaurusEntry,
} from '@myrmidon/cadmus-core';
import { CadmusGraphPgModule } from '@myrmidon/cadmus-graph-pg';
import { CadmusGraphUiModule } from '@myrmidon/cadmus-graph-ui';
import { CadmusProfileCoreModule } from '@myrmidon/cadmus-profile-core';
import { AppRepository, CadmusStateModule } from '@myrmidon/cadmus-state';
import { CadmusUiModule } from '@myrmidon/cadmus-ui';
import { CadmusUiPgModule } from '@myrmidon/cadmus-ui-pg';
import { CadmusItemEditorModule } from '@myrmidon/cadmus-item-editor';
import { CadmusItemListModule } from '@myrmidon/cadmus-item-list';
import { CadmusItemSearchModule } from '@myrmidon/cadmus-item-search';
import { CadmusThesaurusEditorModule } from '@myrmidon/cadmus-thesaurus-editor';
import { CadmusThesaurusListModule } from '@myrmidon/cadmus-thesaurus-list';
import { CadmusThesaurusUiModule } from '@myrmidon/cadmus-thesaurus-ui';
import { RefLookupConfig } from '@myrmidon/cadmus-refs-lookup';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [
    CommonModule,
    RouterModule,
    RouterOutlet,
    MatButtonModule,
    MatIconModule,
    MatMenuModule,
    MatToolbarModule,
    // Cadmus
    DocReferencesComponent,
    HistoricalDateComponent,
    HistoricalDatePipe,
    AssertedCompositeIdsComponent,
    FlagsPickerComponent,
    CadmusCoreModule,
    CadmusProfileCoreModule,
    CadmusStateModule,
    CadmusUiModule,
    CadmusUiPgModule,
    CadmusGraphPgModule,
    CadmusGraphUiModule,
    CadmusItemEditorModule,
    CadmusItemListModule,
    CadmusItemSearchModule,
    CadmusThesaurusEditorModule,
    CadmusThesaurusListModule,
    CadmusThesaurusUiModule,
  ],
  providers: [EnvServiceProvider],
  templateUrl: './app.component.html',
  styleUrl: './app.component.scss',
})
export class AppComponent implements OnInit, OnDestroy {
  private _subs: Subscription[];

  public user?: User;
  public logged?: boolean;
  public itemBrowsers?: ThesaurusEntry[];
  public version: string;

  constructor(
    @Inject('itemBrowserKeys')
    private _itemBrowserKeys: { [key: string]: string },
    private _authService: AuthJwtService,
    private _gravatarService: GravatarService,
    private _appRepository: AppRepository,
    private _router: Router,
    env: EnvService,
    // lookup
    storage: RamStorageService,
    viaf: ViafRefLookupService,
    dbpedia: DbpediaRefLookupService,
    geonames: GeoNamesRefLookupService
  ) {
    this.version = env.get('version') || '';
    this._subs = [];

    // configure external lookup for asserted composite IDs
    storage.store(ASSERTED_COMPOSITE_ID_CONFIGS_KEY, [
      {
        name: 'VIAF',
        iconUrl: '/img/viaf128.png',
        description: 'Virtual International Authority File',
        label: 'ID',
        service: viaf,
        itemIdGetter: (item: any) => item?.viafid,
        itemLabelGetter: (item: any) => item?.term,
      },
      {
        name: 'DBpedia',
        iconUrl: '/img/dbpedia128.png',
        description: 'DBpedia',
        label: 'ID',
        service: dbpedia,
        itemIdGetter: (item: any) => item?.uri,
        itemLabelGetter: (item: any) => item?.label,
      },
      {
        name: 'geonames',
        iconUrl: '/img/geonames128.png',
        description: 'GeoNames',
        label: 'ID',
        service: geonames,
        itemIdGetter: (item: any) => item?.geonameId,
        itemLabelGetter: (item: any) => item?.name,
      },
    ] as RefLookupConfig[]);
  }

  ngOnInit(): void {
    this.user = this._authService.currentUserValue || undefined;
    this.logged = this.user !== null;

    this._subs.push(
      this._authService.currentUser$.subscribe((user: User | null) => {
        this.logged = this._authService.isAuthenticated(true);
        this.user = user || undefined;
        if (user) {
          this._appRepository.load();
        }
      })
    );

    this._subs.push(
      this._appRepository.itemBrowserThesaurus$.subscribe(
        (thesaurus: Thesaurus | undefined) => {
          this.itemBrowsers = thesaurus ? thesaurus.entries : undefined;
        }
      )
    );
  }

  ngOnDestroy(): void {
    this._subs.forEach((sub) => {
      sub.unsubscribe();
    });
  }

  public getItemBrowserRoute(id: string): string {
    return this._itemBrowserKeys[id] || id;
  }

  public getGravatarUrl(email: string, size = 80): string | null {
    return this._gravatarService.buildGravatarUrl(email, size);
  }

  public logout(): void {
    if (!this.logged) {
      return;
    }
    this._authService.logout().subscribe((_) => {
      this._router.navigate(['/home']);
    });
  }
}
```

Styles:

```css
/* app.component.scss */

.small-icon {
  font-size: 85% !important;
  margin-left: 8px;
}

.tb-fill-remaining-space {
  flex: 1 1 auto;
}

footer {
  background-color: #f0f0f0;
  color: #808080;
  padding: 4px;
  text-align: center;
}

#logo {
  flex: 0 0 60px;
}

mat-toolbar mat-icon {
  color: white;
}
```

Template (replace `__PRJ__` with your project name):

```html
<header>
  <mat-toolbar color="primary">
    <span id="logo"
      ><img src="./assets/img/logo-white-40.png" alt="Fusisoft"
    /></span>
    <a mat-button routerLink="/home">Cadmus</a>

    <!-- items menu -->
    <button
      type="button"
      mat-button
      [matMenuTriggerFor]="itemMenu"
      *ngIf="logged && itemBrowsers"
    >
      Items
    </button>
    <mat-menu #itemMenu>
      <a mat-menu-item routerLink="/items">Items</a>
      <a
        mat-menu-item
        *ngFor="let entry of itemBrowsers"
        [routerLink]="'item-browser/' + getItemBrowserRoute(entry.id)"
      ></a>
    </mat-menu>
    <!-- item menu -->
    <ng-container *ngIf="logged && !itemBrowsers">
      <button type="button" mat-button routerLink="/items">Items</button>
    </ng-container>

    <!-- search menu -->
    <button type="button" mat-button routerLink="/search" *ngIf="logged">
      Search
    </button>
    <!-- graph menu -->
    <!-- <button type="button" mat-button routerLink="/graph" *ngIf="logged">
      Graph
    </button> -->
    <!-- profile menu -->
    <ng-container *ngIf="user && user.roles.includes('admin')">
      <button type="button" mat-button [matMenuTriggerFor]="profileMenu">
        Profile
      </button>
      <mat-menu #profileMenu>
        <a mat-menu-item routerLink="/flags"> Flags </a>
        <a mat-menu-item routerLink="/thesauri"> Thesauri </a>
      </mat-menu>
    </ng-container>

    <span class="tb-fill-remaining-space"></span>

    <!-- user -->
    <div *ngIf="user" fxLayout="row" fxLayoutAlign="start center">
      <!-- indicators -->
      <img
        alt="avatar"
        [src]="getGravatarUrl(user.email, 32)"
        [alt]="user.userName"
      />
      <mat-icon
        class="small-icon"
        *ngIf="user && user.roles.includes('admin')"
        title="admin"
        >build</mat-icon
      >
      <mat-icon
        class="small-icon"
        *ngIf="user && !user.emailConfirmed"
        title="You must verify your email address! Please check your mailbox "
        >feedback</mat-icon
      >
      <!-- <button mat-icon-button [mat-menu-trigger-for]="menu">
        <mat-icon>more_vert</mat-icon>
      </button> -->

      <!-- user menu -->
      <button type="button" mat-button [matMenuTriggerFor]="userMenu">
        User
      </button>
      <mat-menu #userMenu>
        <a mat-menu-item routerLink="/reset-password">Reset password</a>
      </mat-menu>

      <!-- admin menu -->
      <button
        type="button"
        *ngIf="user && user.roles.includes('admin')"
        mat-button
        [matMenuTriggerFor]="adminMenu"
      >
        Admin
      </button>
      <mat-menu #adminMenu>
        <a mat-menu-item routerLink="/manage-users">Manage users</a>
        <a mat-menu-item routerLink="/register-user">Register user</a>
      </mat-menu>
    </div>

    <!-- login -->
    <button type="button" *ngIf="!logged" mat-icon-button routerLink="/login">
      <mat-icon>login</mat-icon>
    </button>
    <!-- logout -->
    <button type="button" *ngIf="logged" mat-icon-button (click)="logout()">
      <mat-icon>logout</mat-icon>
    </button>
  </mat-toolbar>
</header>

<main>
  <router-outlet></router-outlet>
</main>

<footer>
  <div layout="row" layout-align="center center">
    <p>
      Cadmus __PRJ__ by
      <a rel="noopener" href="http://www.fusisoft.it" target="_blank"
        >Daniele Fusi</a
      >
      - version {{ version }}
    </p>
  </div>
</footer>
```

## Configure Routes

Setup your routes in `app.routes.ts`, e.g.:

```ts
// app.routes.ts

import { Routes } from '@angular/router';

import {
  AuthJwtGuardService,
  AuthJwtAdminGuardService,
} from '@myrmidon/auth-jwt-login';
import { EditorGuardService } from '@myrmidon/cadmus-api';
import { PendingChangesGuard } from '@myrmidon/cadmus-core';

import { HomeComponent } from './home/home.component';
import { LoginPageComponent } from './login-page/login-page.component';
import { ManageUsersPageComponent } from './manage-users-page/manage-users-page.component';
import { RegisterUserPageComponent } from './register-user-page/register-user-page.component';
import { ResetPasswordComponent } from './reset-password/reset-password.component';

export const routes: Routes = [
  // local home
  { path: '', redirectTo: 'home', pathMatch: 'full' },
  { path: 'home', component: HomeComponent },
  // local auth
  { path: 'login', component: LoginPageComponent },
  {
    path: 'reset-password',
    component: ResetPasswordComponent,
    canActivate: [AuthJwtGuardService],
  },
  {
    path: 'register-user',
    component: RegisterUserPageComponent,
    canActivate: [AuthJwtAdminGuardService],
  },
  {
    path: 'manage-users',
    component: ManageUsersPageComponent,
    canActivate: [AuthJwtAdminGuardService],
  },
  // cadmus - items
  {
    path: 'items',
    loadChildren: () =>
      import('@myrmidon/cadmus-item-list').then(
        (module) => module.CadmusItemListModule
      ),
    canActivate: [AuthJwtGuardService],
  },
  {
    path: 'items/:id',
    loadChildren: () =>
      import('@myrmidon/cadmus-item-editor').then(
        (module) => module.CadmusItemEditorModule
      ),
    canActivate: [AuthJwtGuardService],
    canDeactivate: [PendingChangesGuard],
  },
  {
    path: 'search',
    loadChildren: () =>
      import('@myrmidon/cadmus-item-search').then(
        (module) => module.CadmusItemSearchModule
      ),
    canActivate: [AuthJwtGuardService],
  },
  // cadmus - thesauri
  {
    path: 'thesauri',
    loadChildren: () =>
      import('@myrmidon/cadmus-thesaurus-list').then(
        (module) => module.CadmusThesaurusListModule
      ),
    canActivate: [EditorGuardService],
  },
  {
    path: 'thesauri/:id',
    loadChildren: () =>
      import('@myrmidon/cadmus-thesaurus-editor').then(
        (module) => module.CadmusThesaurusEditorModule
      ),
    canActivate: [EditorGuardService],
  },
  // cadmus - parts
  {
    path: 'items/:iid/general',
    loadChildren: () =>
      import('@myrmidon/cadmus-part-general-pg').then(
        (module) => module.CadmusPartGeneralPgModule
      ),
    canActivate: [AuthJwtGuardService],
  },
  {
    path: 'items/:iid/philology',
    loadChildren: () =>
      import('@myrmidon/cadmus-part-philology-pg').then(
        (module) => module.CadmusPartPhilologyPgModule
      ),
    canActivate: [AuthJwtGuardService],
  },
  // cadmus - graph
  {
    path: 'graph',
    loadChildren: () =>
      import('@myrmidon/cadmus-graph-pg').then(
        (module) => module.CadmusGraphPgModule
      ),
    canActivate: [AuthJwtGuardService],
  },
  // cadmus - preview
  {
    path: 'preview',
    loadChildren: () =>
      import('@myrmidon/cadmus-preview-pg').then(
        (module) => module.CadmusPreviewPgModule
      ),
    canActivate: [AuthJwtGuardService],
  },
  // cadmus - flags
  {
    path: 'flags',
    loadChildren: () =>
      import('@myrmidon/cadmus-flags-pg').then(
        (module) => module.CadmusFlagsPgModule
      ),
  },
  // geography
  {
    path: 'items/:iid/geography',
    loadChildren: () =>
      import('@myrmidon/cadmus-part-geo-pg').then(
        (module) => module.CadmusPartGeoPgModule
      ),
    canActivate: [AuthJwtGuardService],
  },
  // epigraphy
  {
    path: 'items/:iid/epigraphy',
    loadChildren: () =>
      import('@myrmidon/cadmus-part-epigraphy-pg').then(
        (module) => module.CadmusPartEpigraphyPgModule
      ),
    canActivate: [AuthJwtGuardService],
  },

  // fallback
  { path: '**', component: HomeComponent },
];
```

## Configure App

Finally, add to `app.config.ts` the required services, e.g.:

```ts
// app.config.ts

import { ApplicationConfig, importProvidersFrom, provideZoneChangeDetection } from '@angular/core';
import { provideRouter } from '@angular/router';
import {
  provideHttpClient,
  withInterceptors,
  withJsonpSupport,
} from '@angular/common/http';
import { provideAnimationsAsync } from '@angular/platform-browser/animations/async';

// material
import { provideNativeDateAdapter } from '@angular/material/core';

// vendor
import { NgeMonacoModule } from '@cisstech/nge/monaco';
import { NgeMarkdownModule } from '@cisstech/nge/markdown';

// myrmidon
import { authJwtInterceptor } from '@myrmidon/auth-jwt-login';
import { EnvServiceProvider } from '@myrmidon/ng-tools';
import { CadmusApiModule } from '@myrmidon/cadmus-api';
import {
  CADMUS_TEXT_ED_BINDINGS_TOKEN,
  CADMUS_TEXT_ED_SERVICE_OPTIONS_TOKEN,
} from '@myrmidon/cadmus-text-ed';
import {
  MdBoldCtePlugin,
  MdItalicCtePlugin,
  MdLinkCtePlugin,
} from '@myrmidon/cadmus-text-ed-md';
import { TxtEmojiCtePlugin } from '@myrmidon/cadmus-text-ed-txt';
import { GEONAMES_USERNAME_TOKEN } from '@myrmidon/cadmus-refs-geonames-lookup';
import { PROXY_INTERCEPTOR_OPTIONS } from '@myrmidon/cadmus-refs-lookup';

// local
import { INDEX_LOOKUP_DEFINITIONS } from './index-lookup-definitions';
import { ITEM_BROWSER_KEYS } from './item-browser-keys';
import { PART_EDITOR_KEYS } from './part-editor-keys';
import { routes } from './app.routes';

export const appConfig: ApplicationConfig = {
  providers: [
    provideZoneChangeDetection({ eventCoalescing: true }),
    provideRouter(routes),
    provideAnimationsAsync(),
    provideHttpClient(
      withJsonpSupport(),
      withInterceptors([authJwtInterceptor])
    ),
    provideNativeDateAdapter(),
    importProvidersFrom(NgeMonacoModule.forRoot({})),
    importProvidersFrom(NgeMarkdownModule),
    EnvServiceProvider,
    importProvidersFrom(CadmusApiModule),
    // parts and fragments type IDs to editor group keys mappings
    // https://github.com/nrwl/nx/issues/208#issuecomment-384102058
    // inject like: @Inject('partEditorKeys') partEditorKeys: PartEditorKeys
    {
      provide: 'partEditorKeys',
      useValue: PART_EDITOR_KEYS,
    },
    // index lookup definitions
    {
      provide: 'indexLookupDefinitions',
      useValue: INDEX_LOOKUP_DEFINITIONS,
    },
    // item browsers IDs to editor keys mappings
    // inject like: @Inject('itemBrowserKeys') itemBrowserKeys: { [key: string]: string }
    {
      provide: 'itemBrowserKeys',
      useValue: ITEM_BROWSER_KEYS,
    },
    // text editing plugins
    MdBoldCtePlugin,
    MdItalicCtePlugin,
    TxtEmojiCtePlugin,
    MdLinkCtePlugin,
    // provide a factory so that plugins can be instantiated via DI
    {
      provide: CADMUS_TEXT_ED_SERVICE_OPTIONS_TOKEN,
      useFactory: (
        mdBoldCtePlugin: MdBoldCtePlugin,
        mdItalicCtePlugin: MdItalicCtePlugin,
        txtEmojiCtePlugin: TxtEmojiCtePlugin,
        mdLinkCtePlugin: MdLinkCtePlugin
      ) => {
        return {
          plugins: [
            mdBoldCtePlugin,
            mdItalicCtePlugin,
            txtEmojiCtePlugin,
            mdLinkCtePlugin,
          ],
        };
      },
      deps: [
        MdBoldCtePlugin,
        MdItalicCtePlugin,
        TxtEmojiCtePlugin,
        MdLinkCtePlugin,
      ],
    },
    // monaco bindings for plugins
    // 2080 = monaco.KeyMod.CtrlCmd | monaco.KeyCode.KeyB;
    // 2087 = monaco.KeyMod.CtrlCmd | monaco.KeyCode.KeyI;
    // 2083 = monaco.KeyMod.CtrlCmd | monaco.KeyCode.KeyE;
    // 2090 = monaco.KeyMod.CtrlCmd | monaco.KeyCode.KeyL;
    {
      provide: CADMUS_TEXT_ED_BINDINGS_TOKEN,
      useValue: {
        2080: 'md.bold', // Ctrl+B
        2087: 'md.italic', // Ctrl+I
        2083: 'txt.emoji', // Ctrl+E
        2090: 'md.link', // Ctrl+L
      },
    },
    // GeoNames lookup (see environment.prod.ts for the username)
    {
      provide: GEONAMES_USERNAME_TOKEN,
      useValue: 'myrmex',
    },
    // proxy
    {
      provide: PROXY_INTERCEPTOR_OPTIONS,
      useValue: {
        proxyUrl: (window as any).__env?.apiUrl + 'proxy',
        urls: [
          'http://lookup.dbpedia.org/api/search',
          'http://lookup.dbpedia.org/api/prefix',
        ],
      },
    },
  ],
};
```

## Add Supplementary Styles

If using preview, add the corresponding styles in `preview-styles.css` and import them. For instance (under `src/app`):

```css
/*
Preview styles. By convention they all start with prefix "pv-".
*/

.pv-muted {
  color: silver;
}
.pv-flex-row {
  display: flex;
  gap: 8px;
  align-items: center;
  flex-wrap: wrap;
}
.pv-flex-row * {
  flex: 0 0 auto;
}
.cadmus-text-block-view-col {
  border: 1px solid transparent;
  padding: 1px;
}
.cadmus-text-block-view-col:hover {
  border: 1px solid #ffdd26;
  padding: 1px;
}

/* layer ID-based styles */

div[class^="it.vedph.token-text-layer|fr.it.vedph.apparatus"],
div[class*=" it.vedph.token-text-layer|fr.it.vedph.apparatus"] {
  background-color: lightsalmon;
}

div[class^="it.vedph.token-text-layer|fr.it.vedph.chronology"],
div[class*=" it.vedph.token-text-layer|fr.it.vedph.chronology"] {
  background-color: palegoldenrod;
}

div[class^="it.vedph.token-text-layer|fr.it.vedph.comment"],
div[class*=" it.vedph.token-text-layer|fr.it.vedph.comment"] {
  background-color: palegreen;
}

div[class^="it.vedph.token-text-layer|fr.it.vedph.orthography"],
div[class*=" it.vedph.token-text-layer|fr.it.vedph.orthography"] {
  background-color: palevioletred;
}

div[class^="it.vedph.token-text-layer|fr.it.vedph.quotations"],
div[class*=" it.vedph.token-text-layer|fr.it.vedph.quotations"] {
  background-color: paleturquoise;
}

div[class^="it.vedph.token-text-layer|fr.it.vedph.witnesses"],
div[class*=" it.vedph.token-text-layer|fr.it.vedph.witnesses"] {
  background-color: peachpuff;
}

/* note */

.note-text {
  margin: 8px;
  column-count: 4;
  column-width: 400px;
}

/* apparatus layer */

.apparatus-lemma {
  padding: 2px 4px;
  border: 1px solid silver;
  border-radius: 4px;
  margin-right: 4px;
  color: #065e1d;
}

.apparatus-w-value {
  font-weight: bold;
}

.apparatus-w-note {
  font-style: italic;
}

.apparatus-a-value {
  font-style: italic;
}

.apparatus-a-note {
  font-style: italic;
}

.apparatus-sep {
  margin-left: 0.75em;
}

.apparatus-tag {
  font-style: italic;
}

.apparatus-subrange {
  color: silver;
}

.apparatus-value {
  color: #b8690f;
}

.apparatus-type {
  font-style: italic;
}

.apparatus-note {
  font-style: italic;
}

/* comment layer */

.comment a {
  text-decoration: none;
}
.comment a:hover {
  text-decoration: underline;
}
.comment-tag {
  color: silver;
  font-weight: bold;
  padding: 6px;
  border: 1px solid silver;
  border-radius: 6px;
}
.comment-text {
  margin: 8px;
  column-count: 4;
  column-width: 400px;
}
.comment-categories {
    margin: 6px 0;
}
.comment-category {
    background-color: #afd3ff;
    border: 1px solid #afd3ff;
    border-radius: 4px;
    padding: 4px;
}
.comment-keywords {
  line-height: 200%;
}
.comment-kw-x {
  background-color: #34eb98;
  color: white;
  border-radius: 4px;
  padding: 4px;
  margin: 0 4px;
}
.comment-kw-l {
  background-color: #bdb03e;
  color: white;
  border-radius: 4px;
  padding: 4px;
  margin: 0 4px;
}
.comment-kw-v {
  color: #827609;
}
.comment-hdr {
  color: royalblue;
  border-bottom: 1px solid royalblue;
  margin: 8px 0;
  font-variant: small-caps;
}
.comment-references {
  line-height: 200%;
}
.comment-ref-y {
  background-color: #35c6ea;
  color: white;
  border-radius: 4px;
  padding: 4px;
  margin: 0 4px;
}
.comment-ref-t {
  background-color: #34eb98;
  color: white;
  border-radius: 4px;
  padding: 4px;
  margin: 0 4px;
}
.comment-ref-c {
}
.comment-ref-n {
  font-style: italic;
}
.comment-ids {
  line-height: 200%;
}
.comment-id-t {
  background-color: #34eb98;
  color: white;
  border-radius: 4px;
  padding: 4px;
  margin: 0 4px;
}
.comment-id-r {
  font-weight: bold;
  color: orange;
}
.comment-id-n {
  font-style: italic;
}
.comment-id-s {
  border: 1px solid orange;
  border-radius: 4px;
  padding: 4px;
  margin: 0 4px;
}
.comment-assertion {
  border: 1px solid orange;
  border-radius: 6px;
  padding: 6px;
  margin: 4px;
  background-color: #fefefe;
}
.comment-assertion-refs {
}
```

Import this file in your app's `styles.scss`:

```scss
@import "preview-styles.css";
```

## Add Docker Support

You can add Docker support to create an image of your frontend app. Use as templates the files in the reference shell app:

- `Dockerfile` using NGINX to serve the Angular app:

```dockerfile
FROM nginx:alpine

COPY nginx.conf /etc/nginx/nginx.conf
RUN rm /etc/nginx/conf.d/default.conf

WORKDIR /usr/share/nginx/html
COPY dist/cadmus-__PRJ__-app/browser .

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
      - 5080:8080
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

>⚠️ If using bibliography API, add its service to the compose stack too.

- `dockerignore`:

```txt
e2e
node_modules
src
```

>💡 In a true [host environment](../deploy/hosting.md), you would also want to specify a `restart` policy for each container.

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
