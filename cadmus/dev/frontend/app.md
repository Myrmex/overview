---
layout: page
title: Frontend App
subtitle: "Cadmus Frontend Development"
---

- [Requirements](#requirements)
- [Create Angular App](#create-angular-app)
- [Setting Environment Variables](#setting-environment-variables)
- [Tuning Angular Settings](#tuning-angular-settings)
- [Assets](#assets)
- [Cadmus Infrastructure](#cadmus-infrastructure)
- [App Module](#app-module)
- [App Routes](#app-routes)
- [App Component](#app-component)
- [Adding Docker Support](#adding-docker-support)
- [Readme Template](#readme-template)

📌 Task: create a Cadmus frontend web app in a new Angular workspace.

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

>If you are creating an app for the only purpose of developing component libraries in it, the convention is naming it as `-shell` rather than `-app`.

(2) enter the newly created directory and **add Angular Material** via `ng add @angular/material` (choose the Indigo/Pink theme - or whatever you prefer -, setup typography styles=yes, include and enable animations=yes).

(3) install ELF with command `npx @ngneat/elf-cli install`. When prompted, install:

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

(4) install the typical Cadmus packages via NPM:

```bash
npm i @auth0/angular-jwt @myrmidon/auth-jwt-admin @myrmidon/auth-jwt-login
npm i @myrmidon/cadmus-api @myrmidon/cadmus-core @myrmidon/cadmus-graph-ui @myrmidon/cadmus-graph-pg
npm i @myrmidon/cadmus-item-editor @myrmidon/cadmus-item-list @myrmidon/cadmus-item-search
npm i @myrmidon/cadmus-part-general-pg @myrmidon/cadmus-part-general-ui
npm i @myrmidon/cadmus-part-philology-pg @myrmidon/cadmus-part-philology-ui
npm i @myrmidon/cadmus-preview-pg @myrmidon/cadmus-preview-ui @myrmidon/cadmus-profile-core
npm i @myrmidon/cadmus-refs-asserted-chronotope @myrmidon/cadmus-refs-asserted-ids @myrmidon/cadmus-refs-assertion @myrmidon/cadmus-refs-decorated-ids @myrmidon/cadmus-refs-doc-references @myrmidon/cadmus-refs-external-ids @myrmidon/cadmus-refs-historical-date @myrmidon/cadmus-refs-lookup @myrmidon/cadmus-refs-proper-name @myrmidon/cadmus-state @myrmidon/cadmus-text-block-view @myrmidon/cadmus-thesaurus-editor @myrmidon/cadmus-thesaurus-list @myrmidon/cadmus-thesaurus-ui @myrmidon/cadmus-ui @myrmidon/cadmus-ui-pg @myrmidon/ng-mat-tools @myrmidon/ng-tools @myrmidon/ngx-dirty-check @types/diff-match-patch diff-match-patch gravatar
npm i ngx-markdown ngx-monaco-editor rangy --force
```

The above packages are fairly typical, but you might well omit those you are not interested in, e.g. general parts or philology parts, or some of the bricks. Some of the legacy third party libraries like rangy may require `--force`.

Typically you will also need:

- [ngx-markdown](https://github.com/jfcere/ngx-markdown) if you have components _displaying_ Markdown.
- [ngx-monaco-editor](https://github.com/atularen/ngx-monaco-editor) if you components _using_ Markdown or other languages in a Monaco-based editor.

Please be sure to follow the directions provided by each library when installing it. For instance, [ngx-markdown](https://github.com/jfcere/ngx-markdown) requires these packages:

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
  window.__env.version = "0.0.1";
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

## Cadmus Infrastructure

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

(2) copy these folders (each corresponding to an app's page component) into your app's src folder from the [reference project](https://github.com/vedph/cadmus-shell-2):

- `home` (adjust the homepage contents according to your project)
- `login-page`
- `manage-users-page`
- `register-user-page`
- `reset-password`

## App Module

This is were all the Cadmus frontend pieces get assembled together; so, the details may vary according to the packages you installed.

Edit the `app.module.ts` file of your app as follows:

(1) manually add the components copied in the previous step to your `app.module.ts` file under `declarations`:

```ts
  declarations: [
    AppComponent,
    HomeComponent,
    LoginPageComponent,
    ManageUsersPageComponent,
    RegisterUserPageComponent,
    ResetPasswordComponent,
  ],
```

(2) add all the required modules in the `imports` array, which depend on the packages you installed; and add in the `providers` array Cadmus and ELF development tools providers. Here a typical example follows, adjust it as required:

```ts
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { BrowserAnimationsModule } from '@angular/platform-browser/animations';
import { HttpClientModule, HTTP_INTERCEPTORS } from '@angular/common/http';
import { FormsModule, ReactiveFormsModule } from '@angular/forms';
import { RouterModule } from '@angular/router';

// material
import { ClipboardModule } from '@angular/cdk/clipboard';
import { DragDropModule } from '@angular/cdk/drag-drop';
import { MatAutocompleteModule } from '@angular/material/autocomplete';
import { MatBadgeModule } from '@angular/material/badge';
import { MatButtonModule } from '@angular/material/button';
import { MatCardModule } from '@angular/material/card';
import { MatCheckboxModule } from '@angular/material/checkbox';
import { MatChipsModule } from '@angular/material/chips';
import { MatNativeDateModule } from '@angular/material/core';
import { MatDatepickerModule } from '@angular/material/datepicker';
import { MatDialogModule } from '@angular/material/dialog';
import { MatDividerModule } from '@angular/material/divider';
import { MatExpansionModule } from '@angular/material/expansion';
import { MatFormFieldModule } from '@angular/material/form-field';
import { MatIconModule } from '@angular/material/icon';
import { MatInputModule } from '@angular/material/input';
import { MatListModule } from '@angular/material/list';
import { MatMenuModule } from '@angular/material/menu';
import { MatPaginatorModule } from '@angular/material/paginator';
import { MatProgressBarModule } from '@angular/material/progress-bar';
import { MatProgressSpinnerModule } from '@angular/material/progress-spinner';
import { MatRadioModule } from '@angular/material/radio';
import { MatSelectModule } from '@angular/material/select';
import { MatSidenavModule } from '@angular/material/sidenav';
import { MatSliderModule } from '@angular/material/slider';
import { MatSlideToggleModule } from '@angular/material/slide-toggle';
import { MatSnackBarModule } from '@angular/material/snack-bar';
import { MatTableModule } from '@angular/material/table';
import { MatTabsModule } from '@angular/material/tabs';
import { MatToolbarModule } from '@angular/material/toolbar';
import { MatTooltipModule } from '@angular/material/tooltip';
import { MatTreeModule } from '@angular/material/tree';

// ELF
import { devTools } from '@ngneat/elf-devtools';
import { Actions } from '@ngneat/effects-ng';

// ngx-monaco
import { MonacoEditorModule } from 'ngx-monaco-editor';
// ngx-markdown
import { MarkdownModule } from 'ngx-markdown';

// myrmidon
import { NgxDirtyCheckModule } from '@myrmidon/ngx-dirty-check';
import { EnvServiceProvider, NgToolsModule } from '@myrmidon/ng-tools';
import { NgMatToolsModule } from '@myrmidon/ng-mat-tools';
import {
  AuthJwtInterceptor,
  AuthJwtLoginModule,
} from '@myrmidon/auth-jwt-login';
import { AuthJwtAdminModule } from '@myrmidon/auth-jwt-admin';

// cadmus bricks
import { CadmusRefsDocReferencesModule } from '@myrmidon/cadmus-refs-doc-references';
import { CadmusRefsHistoricalDateModule } from '@myrmidon/cadmus-refs-historical-date';
import { CadmusRefsAssertedIdsModule } from '@myrmidon/cadmus-refs-asserted-ids';

// cadmus
import { CadmusApiModule } from '@myrmidon/cadmus-api';
import { CadmusCoreModule } from '@myrmidon/cadmus-core';
import { CadmusGraphPgModule } from '@myrmidon/cadmus-graph-pg';
import { CadmusGraphUiModule } from '@myrmidon/cadmus-graph-ui';
import { CadmusProfileCoreModule } from '@myrmidon/cadmus-profile-core';
import { CadmusStateModule } from '@myrmidon/cadmus-state';
import { CadmusUiModule } from '@myrmidon/cadmus-ui';
import { CadmusUiPgModule } from '@myrmidon/cadmus-ui-pg';
import { CadmusItemEditorModule } from '@myrmidon/cadmus-item-editor';
import { CadmusItemListModule } from '@myrmidon/cadmus-item-list';
import { CadmusItemSearchModule } from '@myrmidon/cadmus-item-search';
import { CadmusThesaurusEditorModule } from '@myrmidon/cadmus-thesaurus-editor';
import { CadmusThesaurusListModule } from '@myrmidon/cadmus-thesaurus-list';
import { CadmusThesaurusUiModule } from '@myrmidon/cadmus-thesaurus-ui';

// local components
import { AppRoutingModule } from './app-routing.module';
import { AppComponent } from './app.component';
import { HomeComponent } from './home/home.component';
import { LoginPageComponent } from './login-page/login-page.component';
import { ManageUsersPageComponent } from './manage-users-page/manage-users-page.component';
import { RegisterUserPageComponent } from './register-user-page/register-user-page.component';
import { ResetPasswordComponent } from './reset-password/reset-password.component';
import { PART_EDITOR_KEYS } from './part-editor-keys';
import { INDEX_LOOKUP_DEFINITIONS } from './index-lookup-definitions';
import { ITEM_BROWSER_KEYS } from './item-browser-keys';

// https://ngneat.github.io/elf/docs/dev-tools/
export function initElfDevTools(actions: Actions) {
  return () => {
    devTools({
      name: 'Cadmus __PRJ__',
      actionsDispatcher: actions,
    });
  };
}

@NgModule({
  // ...
  imports: [
    BrowserAnimationsModule,
    BrowserModule,
    FormsModule,
    ReactiveFormsModule,
    RouterModule,
    HttpClientModule,
    // routing
    AppRoutingModule,
    // material
    ClipboardModule,
    DragDropModule,
    MatAutocompleteModule,
    MatBadgeModule,
    MatButtonModule,
    MatCardModule,
    MatCheckboxModule,
    MatChipsModule,
    MatDatepickerModule,
    MatDialogModule,
    MatDividerModule,
    MatExpansionModule,
    MatFormFieldModule,
    MatIconModule,
    MatInputModule,
    MatListModule,
    MatMenuModule,
    MatNativeDateModule,
    MatPaginatorModule,
    MatProgressBarModule,
    MatProgressSpinnerModule,
    MatRadioModule,
    MatSelectModule,
    MatSidenavModule,
    MatSlideToggleModule,
    MatSliderModule,
    MatSnackBarModule,
    MatTableModule,
    MatTabsModule,
    MatTooltipModule,
    MatToolbarModule,
    MatTreeModule,
    // vendors
    MonacoEditorModule.forRoot(),
    MarkdownModule.forRoot(),
    // myrmidon
    NgToolsModule,
    NgMatToolsModule,
    NgxDirtyCheckModule,
    AuthJwtLoginModule,
    AuthJwtAdminModule,
    // cadmus bricks
    CadmusRefsDocReferencesModule,
    CadmusRefsHistoricalDateModule,
    CadmusRefsAssertedIdsModule,
    // cadmus
    CadmusApiModule,
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
  providers: [
    // environment service
    EnvServiceProvider,
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
    // HTTP interceptor
    // https://medium.com/@ryanchenkie_40935/angular-authentication-using-the-http-client-and-http-interceptors-2f9d1540eb8
    {
      provide: HTTP_INTERCEPTORS,
      useClass: AuthJwtInterceptor,
      multi: true,
    },
    // ELF dev tools
    {
      provide: APP_INITIALIZER,
      multi: true,
      useFactory: initElfDevTools,
      deps: [Actions],
    },
  ],
  bootstrap: [AppComponent],
})
export class AppModule {}
```

## App Routes

In `app-routing.module.ts`:

(1) add routes as sampled in the [reference project](https://github.com/vedph/cadmus-shell-2). You should have routes for:

- _static pages_, like home;
- user authentication and _account_ management pages: login, reset password, register user, manage users;
- _Cadmus-specific pages_ like items list and editor, thesauri list and editor, the parts you want, and eventually the graph and preview pages.

The following is a generic sample:

```ts
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';

// myrmidon
import {
  AuthJwtAdminGuardService,
  AuthJwtGuardService,
} from '@myrmidon/auth-jwt-login';
import { PendingChangesGuard } from '@myrmidon/cadmus-core';
import { EditorGuardService } from '@myrmidon/cadmus-api';

// locals
import { HomeComponent } from './home/home.component';
import { ManageUsersPageComponent } from './manage-users-page/manage-users-page.component';
import { RegisterUserPageComponent } from './register-user-page/register-user-page.component';
import { ResetPasswordComponent } from './reset-password/reset-password.component';
import { LoginPageComponent } from './login-page/login-page.component';

const routes: Routes = [
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
  // TODO: project-specific parts here...
  // fallback
  { path: '**', component: HomeComponent },
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule],
})
export class AppRoutingModule {}
```

## App Component

Code your `app.component.ts` files like those in the [reference shell app](https://github.com/vedph/cadmus-shell-2), eventually customizing them as desired:

- `app.component.ts`: this exposes some data to the HTML template with reference to the currently logged in user and the optional custom item browsers eventually defined in the infrastructure.

```ts
import { Component, OnInit, Inject, OnDestroy } from '@angular/core';
import { Thesaurus, ThesaurusEntry } from '@myrmidon/cadmus-core';
import { Router } from '@angular/router';
import { take } from 'rxjs/operators';
import { Subscription } from 'rxjs';

import {
  AuthJwtService,
  GravatarService,
  User,
} from '@myrmidon/auth-jwt-login';
import { EnvService } from '@myrmidon/ng-tools';
import { AppRepository } from '@myrmidon/cadmus-state';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css'],
})
export class AppComponent implements OnInit, OnDestroy {
  private _authSub?: Subscription;
  private _brSub?: Subscription;

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
    env: EnvService
  ) {
    this.version = env.get('version') || '';
  }

  ngOnInit(): void {
    this.user = this._authService.currentUserValue || undefined;
    this.logged = this.user !== null;

    this._authSub = this._authService.currentUser$.subscribe(
      (user: User | null) => {
        this.logged = this._authService.isAuthenticated(true);
        this.user = user || undefined;
        if (user) {
          this._appRepository.load();
        }
      }
    );

    this._brSub = this._appRepository.itemBrowserThesaurus$.subscribe(
      (thesaurus: Thesaurus | undefined) => {
        this.itemBrowsers = thesaurus ? thesaurus.entries : undefined;
      }
    );
  }

  ngOnDestroy(): void {
    this._authSub?.unsubscribe();
    this._brSub?.unsubscribe();
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
    this._authService
      .logout()
      .pipe(take(1))
      .subscribe((_) => {
        this._router.navigate(['/home']);
      });
  }
}
```

- `app.component.html`: this is the corresponding template. Of course, you can customize it at will.

```html
<header>
  <mat-toolbar color="primary" fxLayout="row" fxLayoutAlign="start center">
    <span style="flex: 0 0 60px"
      ><img src="./assets/img/logo-white-40.png" alt="Fusisoft"
    /></span>
    <a mat-button routerLink="/home">Cadmus</a>

    <!-- items menu -->
    <button
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
        >{{ entry.value }}</a
      >
    </mat-menu>
    <!-- item menu -->
    <ng-container *ngIf="logged && !itemBrowsers">
      <button mat-button routerLink="/items">Items</button>
    </ng-container>

    <!-- search menu -->
    <button mat-button routerLink="/search" *ngIf="logged">Search</button>
    <!-- graph menu -->
    <button mat-button routerLink="/graph" *ngIf="logged">Graph</button>
    <!-- thesauri menu -->
    <button
      mat-button
      routerLink="/thesauri"
      *ngIf="
        user && (user.roles.includes('admin') || user.roles.includes('editor'))
      "
    >
      Thesauri
    </button>

    <!-- demo menu -->
    <button mat-button [matMenuTriggerFor]="demoMenu">Demo</button>
    <mat-menu #demoMenu>
      <button mat-menu-item routerLink="/demo/layers">Text Layers</button>
    </mat-menu>

    <span class="tb-fill-remaining-space"></span>

    <!-- user -->
    <div *ngIf="user" fxLayout="row" fxLayoutAlign="start center">
      <!-- indicators -->
      <img [src]="getGravatarUrl(user.email, 32)" [alt]="user.userName" />
      <mat-icon
        class="small-icon"
        *ngIf="user && user.roles.includes('admin')"
        title="admin"
        >build</mat-icon
      >
      <mat-icon
        class="small-icon"
        *ngIf="user && !user.emailConfirmed"
        title="You must verify your email address! Please check your mailbox {{
          user.email
        }}"
        >feedback</mat-icon
      >

      <!-- user menu -->
      <button mat-button [matMenuTriggerFor]="userMenu">User</button>
      <mat-menu #userMenu>
        <a mat-menu-item routerLink="/reset-password">Reset password</a>
      </mat-menu>

      <!-- admin menu -->
      <button
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
    <button *ngIf="!logged" mat-icon-button routerLink="/login">
      <mat-icon>login</mat-icon>
    </button>
    <!-- logout -->
    <button *ngIf="logged" mat-icon-button (click)="logout()">
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
      Cadmus by
      <a href="http://www.fusisoft.it" target="_blank">Daniele Fusi</a> -
      version {{ version }}
    </p>
  </div>
</footer>
```

- `app.component.css`: the corresponding styles:

```css
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
```

## Adding Docker Support

Finally, you can add Docker support to create an image of your frontend app. Use as templates the files in the reference shell app:

- `Dockerfile`
- `nginx.conf`: the NGINX configuration for serving the web app from the Docker container.
- `docker-compose.yml`: _customize this_ for the image names and versions.
- `docker-compose_linux-vol.yml`: a variation of the preceding, using Linux-hosted volumes for persistent data storage. _Customize this_ in the same way as the preceding one.
- `dockerignore`

## Readme Template

Finally you can use a README template like this:

```md
- [models](https://github.com/vedph/cadmus-__PRJ__)
- [API](https://github.com/vedph/cadmus-__PRJ__-api)

## Docker

Quick Docker image build:

1. `npm run build-lib`
2. update version in `env.js` and `ng build`
3. `docker build . -t vedph2020/cadmus-__PRJ__-app:0.0.1 -t vedph2020/cadmus-__PRJ__-app:latest` (replace with the current version).
```

🏠 [developer's home](../toc.md)

▶️ next: [libraries](libs.md)
