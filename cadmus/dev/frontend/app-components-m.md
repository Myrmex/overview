---
layout: page
title: Frontend App - Components (Modules)
subtitle: "Cadmus Frontend Development"
---

- [App Module](#app-module)
- [App Routes](#app-routes)
- [App Component](#app-component)

📌 Task: add components to a Cadmus frontend web app. ⚠️ This page refers to the legacy, module-based templates. Please refer to [this page](app-components.md) if you want to use module-less, standalone components.

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

(2) (for apps using modules): add all the required modules in the `imports` array, which depend on the packages you installed; and add in the `providers` array the Cadmus providers. Here a typical example follows, adjust it as required:

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

// ngx-monaco
import { MonacoEditorModule } from 'ngx-monaco-editor';
// ngx-markdown
import { MarkdownModule } from 'ngx-markdown';

// myrmidon
import { NgxDirtyCheckModule } from '@myrmidon/ngx-dirty-check';
import { EnvServiceProvider, NgToolsModule } from '@myrmidon/ng-tools';
import { NgMatToolsModule } from '@myrmidon/ng-mat-tools';
import { PagedDataBrowsersModule } from '@myrmidon/paged-data-browsers';
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
    PagedDataBrowsersModule,
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
    <button type="button" mat-button routerLink="/graph" *ngIf="logged">
      Graph
    </button>
    <!-- thesauri menu -->
    <button
      type="button"
      mat-button
      routerLink="/thesauri"
      *ngIf="
        user && (user.roles.includes('admin') || user.roles.includes('editor'))
      "
    >
      Thesauri
    </button>

    <!-- demo menu -->
    <button type="button" mat-button [matMenuTriggerFor]="demoMenu">
      Demo
    </button>
    <mat-menu #demoMenu>
      <button type="button" mat-menu-item routerLink="/demo/layers">
        Text Layers
      </button>
    </mat-menu>

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

#logo {
  flex: 0 0 60px;
}
```

🏠 [developer's home](../toc.md)

◀️ previous: [app setup](app-setup.md)
▶️ next: [libraries](libs.md)
