---
layout: page
title: Creating Libraries
subtitle: "Cadmus Frontend Development"
---

- [Adding Libraries](#adding-libraries)
  - [UI Library](#ui-library)
  - [PG Library](#pg-library)

📌 Create frontend libraries for custom Cadmus models (parts/fragments).

1. [app](app.md)
2. **libraries**
3. [parts](parts.md) (optional)
4. [fragments](fragments.md) (optional)

## Adding Libraries

When creating libraries parts and fragments, you can use different approaches according to the desired level of granularity. When you plan for reuse, creating a single library for each part/fragment editor is the best choice, unless your parts/fragments can be considered so related among themselves that they can be implemented in the same library. If instead you are implementing project-specific editors that will probably never be reused outside of your project, you can go with a multiple-editors approach and implement all of them in a single library.

Of course, you can also adopt a mixed strategy, reserving single-editor libraries for single reusable editors, and multiple-editors libraries for project-specific editors.

In both cases, the procedure for adding a new library to the app's workspace is detailed in the [frontend section](frontend.md#create-angular-app). Here I assume that the library where you will store your editor(s) have already been created.

In a multiple-editors approach, you create two libraries: one with the editors UI, and another for their page wrappers. Conventionally, these libraries are suffixed with `-ui` and `-pg` respectively. In the single editor approach instead you create a single library for each editor, containing both the editor UI and its page wrapper.

In each library module you should import all the required Angular, Angular Material, and Cadmus modules. You can refer to the `app.module` to define the subset of modules required. If your library requires additional modules, remember to add them to the `app.module` file of the app, too.

### UI Library

The UI library is just a standard Angular library with a bunch of components in it. Once you have added all of your imports, you just have to include your part/fragment editors there, and export their components.

### PG Library

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

▶️ next: [parts](parts.md)
