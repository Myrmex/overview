---
layout: page
title: Creating Fragments
subtitle: "Cadmus Frontend Development"
---

- [Adding Fragment](#adding-fragment)
- [Adding UI Editor](#adding-ui-editor)
- [Adding PG Editor Wrapper](#adding-pg-editor-wrapper)

📌 Add to a frontend library a custom Cadmus model for text layers (fragment) with its editor.

1. [app](app.md)
2. [libraries](libs.md)
3. [parts](parts.md) (optional)
4. **fragments**

Adding a fragment is very similar to [adding a part](frontend-part.md). Libraries may include both parts and fragments, and are created in the same way (see [adding libraries](frontend-part.md#adding-libraries) in the parts section).

## Adding Fragment

(1) add the fragment _model_ (derived from `Fragment`), its type ID constant, and its JSON schema constant to `<fragment>.ts` (e.g. `comment-fragment.ts`). You can use a template like this (replace `__NAME__` with your part's name, e.g. `Comment`, adjusting case where required):

```ts
import { Fragment } from "@myrmidon/cadmus-core";

/**
 * The __NAME__ layer fragment server model.
 */
export interface __NAME__Fragment extends Fragment {
  // TODO: add properties
}

export const __NAME___FRAGMENT_TYPEID = "fr.it.vedph.__PRJ__.__NAME__";

export const __NAME___FRAGMENT_SCHEMA = {
  definitions: {},
  $schema: "http://json-schema.org/draft-07/schema#",
  $id:
    "www.vedph.it/cadmus/fragments/<PRJ>/" +
    __NAME___FRAGMENT_TYPEID +
    ".json",
  type: "object",
  title: "__NAME__Fragment",
  // TODO: add which properties are required
  required: ["location"],
  properties: {
    location: {
      $id: "#/properties/location",
      type: "string",
    },
    baseText: {
      $id: "#/properties/baseText",
      type: "string",
    },
    // TODO: add properties
  },
};
```

If you want to infer a schema in the [JSON schema tool](https://jsonschema.net/), which is usually the quickest way of writing the schema, you can use this JSON template adding your model's properties to it:

```json
{
  "location": "1.2",
  "baseText": "abc",
  "TODO": "add properties here"
}
```

(2) add the export for the new file to the library's "barrel" file `public-api.ts`, e.g. `export * from './lib/<NAME>';`.

## Adding UI Editor

(1) add a _fragment editor dumb component_ named after the fragment (e.g. `ng g component comment-fragment` for `CommentFragmentComponent` after `CommentFragment`), and extending `ModelEditorComponentBase<T>` where `T` is the fragment's type.

Code template:

```ts
import { Component, OnInit } from '@angular/core';
import {
  FormBuilder,
  FormControl,
  FormGroup,
  UntypedFormGroup,
} from '@angular/forms';
import { BehaviorSubject } from 'rxjs';

import { AuthJwtService } from '@myrmidon/auth-jwt-login';
import { ThesauriSet, ThesaurusEntry } from '@myrmidon/cadmus-core';
import { EditedObject, ModelEditorComponentBase } from '@myrmidon/cadmus-ui';

import { __NAME__Fragment } from "../__NAME__-fragment";

/**
 * __NAME__ fragment editor component.
 * Thesauri: TODO...
 */
@Component({
  selector: "cadmus-__NAME__-fragment",
  templateUrl: "./__NAME__-fragment.component.html",
  styleUrls: ["./__NAME__-fragment.component.css"],
})
export class __NAME__FragmentComponent
  extends ModelEditorComponentBase<__NAME__Fragment>
  implements OnInit {
  // TODO: add form controls

  // TODO: add tag entries if required, e.g.:
  // public tagEntries: ThesaurusEntry[] | undefined;

  constructor(authService: AuthJwtService, formBuilder: FormBuilder) {
    super(authService, formBuilder);
    // form
    // TODO: instantiate your form's controls
  }

  public override ngOnInit(): void {
    super.ngOnInit();
  }

  protected buildForm(formBuilder: FormBuilder): FormGroup | UntypedFormGroup {
    return formBuilder.group({
      // TODO: add controls instantiated in ctor, e.g.:
      // tag: this.tag,      
    });
  }

  private updateThesauri(thesauri: ThesauriSet): void {
    // TODO: set thesaurus entries
    // const key = 'comment-tags';
    // if (this.hasThesaurus(key)) {
    //   this.tagEntries = thesauri[key].entries;
    // } else {
    //   this.tagEntries = undefined;
    // }
  }

  private updateForm(fr?: __NAME__Fragment | null): void {
    if (!fr) {
      this.form.reset();  
      return;
    }
    // TODO: set form controls from model, e.g.:
    // this.tag.setValue(fr.tag);
    this.form.markAsPristine();
  }

  protected override onDataSet(data?: EditedObject<__NAME__Fragment>): void {
    // thesauri
    if (data?.thesauri) {
      this.updateThesauri(data.thesauri);
    }

    // form
    this.updateForm(data?.value);
  }

  protected getValue(): __NAME__Fragment {
    const fr = this.getEditedFragment() as __NAME__Fragment;
    // TODO: set fragment's properties from form controls, e.g.:
    // fr.standard = this.standard.value.trim();
    return fr;
  }
}
```

Sample HTML template:

```html
<form [formGroup]="form" (submit)="save()">
  <mat-card>
    <mat-card-header>
      <div mat-card-avatar>
        <mat-icon>textsms</mat-icon>
      </div>
      <mat-card-title>__NAME__ Fragment {{ data?.value?.location }}</mat-card-title>
      <mat-card-subtitle> {{ data?.baseText }} </mat-card-subtitle>
    </mat-card-header>

    <mat-card-content> TODO: add controls </mat-card-content>

    <mat-card-actions>
      <cadmus-close-save-buttons
        [form]="form"
        [noSave]="userLevel < 2"
        (closeRequest)="close()"
      ></cadmus-close-save-buttons>
    </mat-card-actions>
  </mat-card>
</form>
```

(2) remember to add the component to the barrel `public-api.ts`, and to its module's `declarations` and `exports`.

## Adding PG Editor Wrapper

Once you have the fragment editor, you need its wrapper page, which in turn is linked to a specific route in the context of the project. Such route is defined in the fragment's library when using a granular approach, or in the general PG library when using a multiple-editors approach.

(1) under your library's `src/lib` folder, add a **fragment editor feature component** named after the part (e.g. `ng g component comment-fragment-feature` for `CommentFragmentFeatureComponent` after `CommentFragment`).

(2) ensure that this component is both under the module `declarations` and `exports`, and in the `public-api.ts` barrel file.

(3) add the corresponding **route** in the PG library's module of the project's PG library, e.g.:

```ts
{
  path: `fragment/:pid/${__NAME___FRAGMENT_TYPEID}/:loc`,
  pathMatch: "full",
  component: __NAME__FragmentFeatureComponent,
  canDeactivate: [PendingChangesGuard],
},
```

(4) implement the feature **editor component** by making it extend `EditFragmentFeatureBase`, like in this code template:

```ts
import { Component, OnInit } from '@angular/core';
import { Router, ActivatedRoute } from '@angular/router';
import { MatSnackBar } from '@angular/material/snack-bar';

import { LibraryRouteService } from '@myrmidon/cadmus-core';

@Component({
  selector: 'cadmus-__NAME__-part-feature',
  templateUrl: './note-__NAME__-feature.component.html',
  styleUrls: ['./note-__NAME__-feature.component.css'],
})
export class __NAME__PartFeatureComponent
  extends EditFragmentFeatureBase
  implements OnInit
{
  constructor(
    router: Router,
    route: ActivatedRoute,
    snackbar: MatSnackBar,
    editorService: FragmentEditorService,
    libraryRouteService: LibraryRouteService
  ) {
    super(router, route, snackbar, editorService, libraryRouteService);
  }

  protected override getReqThesauriIds(): string[] {
    // TODO: return the IDs of all the thesauri required by the wrapped editor, e.g.:
    return ['note-tags'];
    // or just avoid overriding the function if no thesaurus required
  }
}
```

The HTML template just wraps the UI editor preceded by a current-item bar:

```html
<cadmus-current-item-bar></cadmus-current-item-bar>
<div class="base-text">
  <cadmus-decorated-token-text
    [baseText]="data?.baseText || ''"
    [locations]="frLoc ? [frLoc] : []"
  ></cadmus-decorated-token-text>
</div>
<cadmus-__NAME__-fragment
  [data]="$any(data)"
  (dataChange)="save($event)"
  (editorClose)="close()"
  (dirtyChange)="onDirtyChange($event)"
></cadmus-__NAME__-fragment>
```

(5) in your app's project `part-editor-keys.ts`, add the mapping for the fragment just created under the layer part key, like e.g.:

```ts
// layer parts
[TOKEN_TEXT_LAYER_PART_TYPEID]: {
  part: GENERAL,
  fragments: {
    // each fragment type in the layer part is a property
    [COMMENT_FRAGMENT_TYPEID]: GENERAL,
    [APPARATUS_FRAGMENT_TYPEID]: PHILOLOGY,
    [LING_TAGS_FRAGMENT_TYPEID]: TGR_GR,
  },
},
```

Here, the type ID of the fragment (from its model in the "ui" library) is mapped to the route prefix constant TGR_GR = `tgr-gr`, which is the root route to the "pg" library module for the app.

🏠 [developer's home](../toc.md)
