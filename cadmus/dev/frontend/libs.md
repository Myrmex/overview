---
layout: page
title: Creating Libraries
subtitle: "Cadmus Frontend Development"
---

- [Adding Libraries](#adding-libraries)
  - [Single-Component Libraries](#single-component-libraries)
  - [Multiple-Components Libraries](#multiple-components-libraries)
  - [Setup](#setup)

📌 Create frontend libraries for custom Cadmus models (parts/fragments).

1. [app](app.md)
2. **libraries**
3. [parts](parts.md) (optional)
4. [fragments](fragments.md) (optional)

## Adding Libraries

When creating libraries parts and fragments, you can use different approaches according to the desired level of granularity.

When you plan for reuse, creating a [single library](#single-component-libraries) for each component (part/fragment editor) is the best choice, unless your parts/fragments can be considered so related among themselves that they can be implemented in the same library.

If instead you are implementing project-specific editors that will probably never be reused outside of your project, you can go with a [multiple-components](#multiple-components-libraries) approach and implement all of them in a single library.

Of course, you can also adopt a mixed strategy, reserving single-editor libraries for single reusable editors, and multiple-editors libraries for project-specific editors.

### Single-Component Libraries

In the single-component approach you create a single library for each editor, containing both the editor UI and its page wrapper. Also, a `-pg` library will be created for all the single-components libraries which go together in a project.

- the library is added like `ng generate library @myrmidon/cadmus-part-<PRJ>-<NAME> --prefix cadmus`, e.g. `cadmus-part-itinera-cod-loci` (use `-fr-` instead of `-part-` for fragments).
- in each library you will have:
  - the part/fragment model (e.g. a file `cod-loci-part`).
  - the part/fragment editor UI (e.g. a folder with the `cod-loci-part` editor component).
  - the part/fragment editor wrapper (e.g. a folder with the `cod-loci-part-feature` editor wrapper).
- also, a library for all of your components in the project will provide routes to the editor wrappers (e.g. `cadmus-part-itinera-pg`). This will include only the library module, which imports all the single component libraries, and exports routes like e.g.:

```ts
import { CommonModule } from '@angular/common';
import { NgModule } from '@angular/core';
import { FormsModule, ReactiveFormsModule } from '@angular/forms';
import { RouterModule } from '@angular/router';
import { CadmusStateModule } from '@myrmidon/cadmus-state';
import { CadmusUiModule } from '@myrmidon/cadmus-ui';
import { CadmusUiPgModule } from '@myrmidon/cadmus-ui-pg';

// cadmus
import { CadmusCoreModule, PendingChangesGuard } from '@myrmidon/cadmus-core';
import {
  COD_LOCI_PART_TYPEID,
  CodLociPartFeatureComponent,
  CadmusPartItineraCodLociModule,
} from '@myrmidon/cadmus-part-itinera-cod-loci';
// ... etc.

export const RouterModuleForChild = RouterModule.forChild([
  {
    path: `${COD_LOCI_PART_TYPEID}/:pid`,
    pathMatch: 'full',
    component: CodLociPartFeatureComponent,
    canDeactivate: [PendingChangesGuard],
  },
  // ... etc.
});

@NgModule({
  declarations: [],
  imports: [
    CommonModule,
    FormsModule,
    ReactiveFormsModule,
    // Cadmus
    RouterModuleForChild,
    CadmusCoreModule,
    CadmusStateModule,
    CadmusUiModule,
    CadmusUiPgModule,
    // Itinera
    CadmusPartItineraCodLociModule,
    // ... etc.
  ],
  exports: [],
})
export class CadmusPartItineraPgModule {}
```

### Multiple-Components Libraries

In a multiple-components approach, you create two libraries: one with the editors UI, and another for their page wrappers. Conventionally, these libraries are suffixed with `-ui` and `-pg` respectively.

The UI library is just a standard Angular library with a bunch of components in it. Once you have added all of your imports, you just have to include your part/fragment editors there, and export their components.

The PG library is a standard Angular library with some added routes, one for each part included in the library. In its module add code like this:

```ts
// ... TODO: other imports ...

// cadmus
import { CadmusCoreModule, PendingChangesGuard } from '@myrmidon/cadmus-core';
import { CadmusMatPhysicalSizeModule } from '@myrmidon/cadmus-mat-physical-size';
import { CadmusRefsDecoratedCountsModule } from '@myrmidon/cadmus-refs-decorated-counts';
import { CadmusStateModule } from '@myrmidon/cadmus-state';
import { CadmusUiModule } from '@myrmidon/cadmus-ui';

export const RouterModuleForChild = RouterModule.forChild([
  {
    path: `${YOUR_PART_TYPEID}/:pid`,
    pathMatch: 'full',
    component: YourPartFeatureComponent,
    canDeactivate: [PendingChangesGuard],
  },
  // ... other parts ...
]);

@NgModule({
  declarations: [
    // ...
  ],
  imports: [
    CommonModule,
    FormsModule,
    ReactiveFormsModule,
    RouterModuleForChild,
    // material
    // ...
    // cadmus
    CadmusCoreModule,
    CadmusStateModule,
    CadmusUiModule,
    CadmusUiPgModule,
    CadmusUiPgModule,
    // ...
  ],
  exports: [],
})
export class Cadmus__PRJ__PartPgModule {}
```

### Setup

Once you create a library, whatever its type:

(1) remove the stub code files added by Angular CLI: the sample component and its service. Also, take the time for adding more metadata to its `package.json` file, e.g. (replace `__PRJ__` with your project's ID):

```json
{
  "name": "@myrmidon/cadmus-itinera-part-lt-ui",
  "version": "0.0.1",
  "description": "Cadmus - general parts UI components.",
  "keywords": [
    "Cadmus",
    "__PRJ__"
  ],
  "homepage": "https://github.com/vedph/cadmus_shell",
  "repository": {
    "type": "git",
    "url": "https://github.com/vedph/cadmus_shell"
  },
  "author": {
    "name": "Daniele Fusi"
  },
  "peerDependencies": {
    "@angular/common": "^15.0.0",
    "@angular/core": "^15.0.0",
  },
  "dependencies": {
    "tslib": "^2.0.0"
  }
}
```

(2) In each library module you should import all the required Angular, Angular Material, and Cadmus modules. You can refer to the `app.module` of your app to define the subset of modules required. If your library requires additional modules, remember to add them to the `app.module` file of the app, too.

(3) in your app's `package.json` file, to speed up builds, for each added library you can add the corresponding **build commands** to `package.json` scripts (to be run like `npm run <SCRIPTNAME>`), e.g.:

```json
{
  "build-ui": "ng build @myrmidon/cadmus-__PRJ__-part-ui",
  "build-pg": "ng build @myrmidon/cadmus-__PRJ__-part-pg",
  "build-lib": "npm run-script build-ui && npm run-script build-pg"
}
```

The `build-lib` command is used to build all the libraries in the workspace. Be sure to enumerate them in their order of dependencies when writing the command.

🏠 [developer's home](../toc.md)

▶️ next: [parts](parts.md)
