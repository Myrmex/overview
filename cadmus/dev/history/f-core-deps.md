---
layout: page
title: Cadmus Development
subtitle: History - Core Frontend Dependencies
---

## Core Frontend Dependencies Update

📆 Date: 2024-12-03.

### Rationale

Code modernization for Angular (standalone components, signal-based properties, functional DI) starts with updating its core dependencies. These being the lowest layers of the frontend, they have been refactored as a first stage. Next stages will involve a progressive refactoring of the libraries in bricks, then in this shell, and finally of all the single Cadmus projects. The modernization refers to:

1. migrate to standalone: `ng generate @angular/core:standalone`. This usually requires to manually define the imports of each component. Also, most of the work is then spent in changing the consumer code, which needs to import components at a more granular level.
2. migrate dependency injection to function-based paradigm: `ng generate @angular/core:inject`. This usually is trivial.
3. migrate properties from decorators to signal-based: `ng generate @angular/core:signals --insert-todos`. This is the step requiring most work, because the schematics are applied only to a minor subset of trivial cases. All the others must be manually refactored into signals: `input` or `model` for properties, `output` for events; the corresponding code and template must be updated so that whenever it reads a signal it calls a function, and whenever it writes it either sets it via output if it's an input, or uses a `model`.

## Affected Products

⚠️ All the frontend libraries and apps. Currently, these libraries have been modernized:

- `ngx-tools` and `ngx-mat-tools`; the old libraries are ng-tools and ng-mat-tools. The new one has the correct name according to Angular conventions.
- `ngx-data-browser`;
- `authjwt`;

Consequently, bricks and Cadmus shell libraries have been updated to use these libraries rather than their legacy counterparts. Apart from these changes in dependencies, currently no other change affected bricks and Cadmus shell libraries.

### Upgrade Path

1. ensure to update the app to the latest version of Angular (19+). Also, if your app depends on other libraries which in turn depend on the modernized core libraries, be sure to update them first.
2. replace legacy libraries:

    ```bash
    npm uninstall @myrmidon/ng-mat-tools @myrmidon/ng-tools --force
    npm i @myrmidon/ngx-mat-tools @myrmidon/ngx-tools --force
    ```

3. update all the libraries in `packages.json` and do a `npm i --force`.
4. in your app's module or config, replace module-based imports for these libraries with their more granular counterparts, e.g.:

    ```ts
    // myrmidon
    EllipsisPipe,
    FlatLookupPipe,
    SafeHtmlPipe,
    AuthJwtLoginComponent,
    AuthJwtRegistrationComponent,
    UserListComponent,
    GravatarPipe,
    ```

5. in your code, replace all the references to `@myrmidon/ng-mat-tools` and `@myrmidon/ng-tools` with the new x-counterparts: `@myrmidon/ngx-mat-tools` and `@myrmidon/ngx-tools`. Where you imported modules, replace these with the required components.

6. optionally, improve the login page by adding progress and error checking:

```ts
// login-page.component.ts
import { Component } from '@angular/core';
import { MatSnackBar } from '@angular/material/snack-bar';
import { Router } from '@angular/router';
import { AuthJwtService, Credentials } from '@myrmidon/auth-jwt-login';

@Component({
  selector: 'app-login-page',
  templateUrl: './login-page.component.html',
  styleUrls: ['./login-page.component.css'],
  standalone: false,
})
export class LoginPageComponent {
  constructor(
    private _authService: AuthJwtService,
    private _router: Router,
    private _snackbar: MatSnackBar
  ) {}

  public busy = false;
  public error?: string;

  public onLoginRequest(credentials: Credentials): void {
    this.busy = true;

    this._authService.login(credentials.name, credentials.password).subscribe({
      next: (user) => {
        console.log('User logged in', user);
        this._router.navigate([credentials.returnUrl || '/home']);
      },
      error: (error) => {
        this.error = 'Login failed';
        console.error(this.error, error);
        this._snackbar.open(this.error, 'Dismiss', {
          duration: 5000,
        });
      },
      complete: () => {
        this.busy = false;
      },
    });
  }

  public onResetRequest(): void {
    this._router.navigate(['/reset-password']);
  }
}
```

Template:

```html
<mat-card appearance="outlined">
  <mat-card-title>
    <mat-card-header>
      <div mat-card-avatar>
        <mat-icon>lock_open</mat-icon>
      </div>
      Login
    </mat-card-header>
  </mat-card-title>
  <mat-card-content>
    <auth-jwt-login
      [forgot]="true"
      [validating]="busy"
      [error]="error"
      (loginRequest)="onLoginRequest($event)"
      (resetRequest)="onResetRequest()"
    />
  </mat-card-content>
</mat-card>
```
