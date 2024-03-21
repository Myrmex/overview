---
layout: page
title: Frontend App - Components (Modules)
subtitle: "Cadmus Frontend Development"
---

- [Add Global Providers](#add-global-providers)
- [Add Wrapper Modules](#add-wrapper-modules)
- [Root Component](#root-component)
- [Add Routes](#add-routes)

📌 Task: add components to a Cadmus frontend web app. This page refers to a module-less Angular application with standalone components. Alternatively, you can use the [legacy module-based templates](app-components-m.md).

## Add Global Providers

Add to `app.config.ts` global providers as in this code:

```ts
import { ApplicationConfig, importProvidersFrom } from '@angular/core';
import { HTTP_INTERCEPTORS, HttpClientModule } from '@angular/common/http';
import { provideRouter } from '@angular/router';
import { provideAnimationsAsync } from '@angular/platform-browser/animations/async';

import { AuthJwtInterceptor } from '@myrmidon/auth-jwt-login';
import { EnvServiceProvider } from '@myrmidon/ng-tools';
import { CadmusApiModule } from '@myrmidon/cadmus-api';

import { provideMarkdown } from 'ngx-markdown';

import { routes } from './app.routes';

import { INDEX_LOOKUP_DEFINITIONS } from './index-lookup-definitions';
import { ITEM_BROWSER_KEYS } from './item-browser-keys';
import { PART_EDITOR_KEYS } from './part-editor-keys';

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes),
    provideAnimationsAsync(),
    provideHttpClient(),
    provideMarkdown(),
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
    // HTTP interceptor
    // https://medium.com/@ryanchenkie_40935/angular-authentication-using-the-http-client-and-http-interceptors-2f9d1540eb8
    {
      provide: HTTP_INTERCEPTORS,
      useClass: AuthJwtInterceptor,
      multi: true,
    },
  ],
};
```

## Add Wrapper Modules

If you are using modules requiring a configuration when importing them, like Monaco Editor, Markdown, or MapGL, do this configuration in a wrapper module, and then import this module in your root `AppComponent`.

Here are some typical wrappers:

**Monaco**:

```ts
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { MonacoEditorModule } from 'ngx-monaco-editor-v2';

@NgModule({
  declarations: [],
  imports: [CommonModule, MonacoEditorModule.forRoot()],
})
export class MonacoWrapperModule {}
```

***Markdown***: this module is no more needed since now this library provides a `provideMarkdown` function.

```ts
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { MarkdownModule } from 'ngx-markdown';

@NgModule({
  declarations: [],
  imports: [CommonModule, MarkdownModule.forRoot()],
})
export class MarkdownWrapperModule {}
```

Using the `provideMarkdown` function in your `app.config.ts`:

```ts
import { provideMarkdown } from 'ngx-markdown';

export const appConfig: ApplicationConfig = {
  providers: [
    // ... other providers ...
    provideMarkdown(),
  ],
};
```

**MapGL**:

```ts
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { NgxMapboxGLModule } from 'ngx-mapbox-gl';

@NgModule({
  declarations: [],
  imports: [
    CommonModule,
    NgxMapboxGLModule.withConfig({
      accessToken: (window as any).__env.mapboxToken,
    }),
  ],
})
export class MapglWrapperModule {}
```

## Root Component

Import app-wide modules in `app.component.ts` and add typical code to it:

```ts
import { Component, Inject, OnDestroy, OnInit } from '@angular/core';
import { CommonModule } from '@angular/common';
import { Router, RouterModule, RouterOutlet } from '@angular/router';
import { Subscription } from 'rxjs';

import { MatButtonModule } from '@angular/material/button';
import { MatIconModule } from '@angular/material/icon';
import { MatMenuModule } from '@angular/material/menu';
import { MatToolbarModule } from '@angular/material/toolbar';

import { EnvService, EnvServiceProvider } from '@myrmidon/ng-tools';
import {
  User,
  AuthJwtService,
  GravatarService,
} from '@myrmidon/auth-jwt-login';

// Cadmus bricks
import { CadmusRefsAssertedIdsModule } from '@myrmidon/cadmus-refs-asserted-ids';
import { CadmusRefsDocReferencesModule } from '@myrmidon/cadmus-refs-doc-references';
import { CadmusRefsHistoricalDateModule } from '@myrmidon/cadmus-refs-historical-date';
// Cadmus
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
import { CadmusUiFlagsPickerModule } from '@myrmidon/cadmus-ui-flags-picker';

import { MonacoWrapperModule } from './monaco-wrapper.module';
import { MarkdownWrapperModule } from './markdown-wrapper.module';

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
    MarkdownWrapperModule,
    MonacoWrapperModule,
    // Cadmus
    CadmusRefsDocReferencesModule,
    CadmusRefsHistoricalDateModule,
    CadmusRefsAssertedIdsModule,
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
    CadmusUiFlagsPickerModule,
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
    env: EnvService
  ) {
    this.version = env.get('version') || '';
    this._subs = [];
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

The corresponding HTML template:

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
        >{{ entry.value }}</a
      >
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
        title="You must verify your email address! Please check your mailbox {{
          user.email
        }}"
        >feedback</mat-icon
      >
      <!-- <button mat-icon-button [mat-menu-trigger-for]="menu">
        <mat-icon>more_vert</mat-icon>
      </button> -->

      <!-- user menu -->
      <button type="button" mat-button [matMenuTriggerFor]="userMenu">User</button>
      <mat-menu #userMenu>
        <a mat-menu-item routerLink="/reset-password">Reset password</a>
      </mat-menu>

      <!-- admin menu -->
      <button type="button"
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
      Cadmus by
      <a rel="noopener" href="http://www.fusisoft.it" target="_blank"
        >Daniele Fusi</a
      >
      - version {{ version }}
    </p>
  </div>
</footer>
```

The corresponding CSS styles:

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

#logo {
  flex: 0 0 60px;
}
```

## Add Routes

In `app.routes.ts`, define the typical routes, for instance (remove the modules you are not using):

```ts
import { Routes } from '@angular/router';

import { AuthJwtGuardService, AuthJwtAdminGuardService } from '@myrmidon/auth-jwt-login';
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

🏠 [developer's home](../toc.md)

◀️ previous: [app setup](app-setup.md)
▶️ next: [libraries](libs.md)
