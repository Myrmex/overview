---
layout: page
title: Creating Parts
subtitle: "Cadmus Frontend Development"
---

- [Adding Part](#adding-part)
- [Adding UI Editor](#adding-ui-editor)
  - [Generic Part Editor Template](#generic-part-editor-template)
  - [List Part Editor Template](#list-part-editor-template)
- [Adding PG Editor Wrapper](#adding-pg-editor-wrapper)

📌 Add to a frontend library a custom Cadmus model (part) with its editor.

1. [app](frontend/app.md)
2. [libraries](frontend/libs.md)
3. **parts**
4. [fragments](frontend/fragments.md) (optional)

## Adding Part

(1) in your library, under `src/lib`, add the **part model** file, named `<PARTNAME>.ts` (e.g. `cod-bindings-part.ts`), like this:

```ts
import { Part } from "@myrmidon/cadmus-core";

/**
 * The __NAME__ part model.
 */
export interface __NAME__Part extends Part {
  // TODO: add properties
}

/**
 * The type ID used to identify the __NAME__Part type.
 */
export const __NAME___PART_TYPEID = "it.vedph.__PRJ__.__NAME__";

/**
 * JSON schema for the __NAME__ part.
 * You can use the JSON schema tool at https://jsonschema.net/.
 */
export const __NAME___PART_SCHEMA = {
  $schema: "http://json-schema.org/draft-07/schema#",
  $id:
    "www.vedph.it/cadmus/parts/__PRJ__/__LIB__/" +
    __NAME___PART_TYPEID +
    ".json",
  type: "object",
  title: "__NAME__Part",
  required: [
    "id",
    "itemId",
    "typeId",
    "timeCreated",
    "creatorId",
    "timeModified",
    "userId",
    // TODO: add other required properties here...
  ],
  properties: {
    timeCreated: {
      type: "string",
      pattern: "^\\d{4}-\\d{2}-\\d{2}T\\d{2}:\\d{2}:\\d{2}.\\d+Z$",
    },
    creatorId: {
      type: "string",
    },
    timeModified: {
      type: "string",
      pattern: "^\\d{4}-\\d{2}-\\d{2}T\\d{2}:\\d{2}:\\d{2}.\\d+Z$",
    },
    userId: {
      type: "string",
    },
    id: {
      type: "string",
      pattern: "^[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}$",
    },
    itemId: {
      type: "string",
      pattern: "^[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}$",
    },
    typeId: {
      type: "string",
      pattern: "^[a-z][-0-9a-z._]*$",
    },
    roleId: {
      type: ["string", "null"],
      pattern: "^([a-z][-0-9a-z._]*)?$",
    },

    // TODO: add properties and fill the "required" array as needed
  },
};
```

If you want to infer a schema in the [JSON schema tool](https://jsonschema.net/), which is usually the quickest way of writing the schema, you can use this JSON template, adding your model's properties to it:

```json
{
  "id": "009dcbd9-b1f1-4dc2-845d-1d9c88c83269",
  "itemId": "2c2eadb7-1972-4415-9a43-b8036b6fa685",
  "typeId": "it.vedph.thetype",
  "roleId": null,
  "timeCreated": "2019-11-29T16:48:49.694Z",
  "creatorId": "zeus",
  "timeModified": "2019-11-29T16:48:49.694Z",
  "userId": "zeus",
  "TODO": "add properties here"
}
```

(2) add the new file to the exports of the "barrel" `public-api.ts` file in the module, like `export * from './lib/<NAME>-part';`.

## Adding UI Editor

This is the part editor UI, a dumb component which essentially uses a form to represent the data of a part's model. These data are adapted to the form when loading them, and converted back to the part's model when saving.

(1) in `src/lib`, add a **part editor dumb component** named after the part (e.g. `ng g component note-part` for `NotePartComponent` after `NotePart`), and extending `ModelEditorComponentBase<T>` where `T` is the part's type. Here we usually have two cases: a generic part, or a part consisting only of a list of entities. Two different templates are provided here.

### Generic Part Editor Template

(1) write code and HTML template:

```ts
import { Component, OnInit } from '@angular/core';
import {
  FormControl,
  FormBuilder,
  Validators,
  FormGroup,
  UntypedFormGroup,
} from '@angular/forms';

import { AuthJwtService } from '@myrmidon/auth-jwt-login';
import { ThesauriSet, ThesaurusEntry } from '@myrmidon/cadmus-core';
import { EditedObject, ModelEditorComponentBase } from '@myrmidon/cadmus-ui';

import { __NAME__Part, __NAME___PART_TYPEID } from '../__NAME__-part';

/**
 * __NAME__ part editor component.
 * Thesauri: ...TODO list of thesauri IDs...
 */
@Component({
  selector: 'cadmus-__NAME__-part',
  templateUrl: './__NAME__-part.component.html',
  styleUrls: ['./__NAME__-part.component.css'],
})
export class __NAME__PartComponent
  extends ModelEditorComponentBase<__NAME__Part>
  implements OnInit
{
  // TODO: add your form controls here, e.g.:
  // public tag: FormControl<string | null>;
  // public text: FormControl<string | null>;

  // TODO: add your thesauri entries here, e.g.:
  // public tagEntries?: ThesaurusEntry[];

  constructor(authService: AuthJwtService, formBuilder: FormBuilder) {
    super(authService, formBuilder);
    // form
    // TODO: create your form controls (but NOT the form itself), e.g.:
    // this.tag = formBuilder.control(null, Validators.maxLength(100));
    // this.text = formBuilder.control(null, Validators.required);
  }

  public override ngOnInit(): void {
    super.ngOnInit();
  }

  protected buildForm(formBuilder: FormBuilder): FormGroup | UntypedFormGroup {
    return formBuilder.group({
      // TODO: assign your created form controls to the form returned here, e.g.:
      // tag: this.tag,
      // text: this.text,
    });
  }

  private updateThesauri(thesauri: ThesauriSet): void {
    // TODO: setup thesauri entries here, e.g.:
    // const key = 'note-tags';
    // if (this.hasThesaurus(key)) {
    //  this.tagEntries = thesauri[key].entries;
    // } else {
    //  this.tagEntries = undefined;
    // }
  }

  private updateForm(part?: __NAME__Part | null): void {
    if (!part) {
      this.form.reset();
      return;
    }
    this.tag.setValue(part.tag || null);
    this.text.setValue(part.text);
    this.form.markAsPristine();
  }

  protected override onDataSet(data?: EditedObject<__NAME__Part>): void {
    // thesauri
    if (data?.thesauri) {
      this.updateThesauri(data.thesauri);
    }

    // form
    this.updateForm(data?.value);
  }

  protected getValue(): __NAME__Part {
    let part = this.getEditedPart(__NAME___PART_TYPEID) as __NAME__Part;
    // TODO: assign values to your part properties from form controls, e.g.:
    // part.tag = this.tag.value || undefined;
    // part.text = this.text.value?.trim() || '';
    return part;
  }
}
```

HTML template:

```html
<form [formGroup]="form" (submit)="save()">
  <mat-card>
    <mat-card-header>
      <div mat-card-avatar>
        <mat-icon>picture_in_picture</mat-icon>
      </div>
      <mat-card-title>__NAME__ Part</mat-card-title>
    </mat-card-header>
    <mat-card-content> TODO: your template here... </mat-card-content>
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

(2) ensure the component has been added to its module's `declarations` and `exports`, and to the `public-api.ts` barrel file.

### List Part Editor Template

(1) write code and HTML template:

```ts
import { Component, OnInit } from '@angular/core';
import {
  FormControl,
  FormBuilder,
  FormGroup,
  UntypedFormGroup,
} from '@angular/forms';
import { take } from 'rxjs/operators';

import { NgToolsValidators } from '@myrmidon/ng-tools';
import { DialogService } from '@myrmidon/ng-mat-tools';
import { AuthJwtService } from '@myrmidon/auth-jwt-login';
import { EditedObject, ModelEditorComponentBase } from '@myrmidon/cadmus-ui';
import { ThesauriSet, ThesaurusEntry } from '@myrmidon/cadmus-core';

import {
  __NAME__,
  __NAME__sPart,
  __NAME__S_PART_TYPEID,
} from '../__NAME__s-part';

/**
 * __NAME__sPart editor component.
 * Thesauri: ...TODO list of thesauri IDs...
 */
@Component({
  selector: 'cadmus-__NAME__s-part',
  templateUrl: './__NAME__s-part.component.html',
  styleUrls: ['./__NAME__s-part.component.css'],
})
export class __NAME__sPartComponent
  extends ModelEditorComponentBase<__NAME__sPart>
  implements OnInit
{
  private _editedIndex: number;

  public tabIndex: number;
  public edited: __NAME__ | undefined;

  // TODO: add your thesauri entries here, e.g.:
  // cod-binding-tags
  // public tagEntries: ThesaurusEntry[] | undefined;

  public entries: FormControl<__NAME__[]>;

  constructor(
    authService: AuthJwtService,
    formBuilder: FormBuilder,
    private _dialogService: DialogService
  ) {
    super(authService, formBuilder);
    this._editedIndex = -1;
    this.tabIndex = 0;
    // form
    this.entries = formBuilder.control([], {
      // at least 1 entry
      validators: NgToolsValidators.strictMinLengthValidator(1),
      nonNullable: true,
    });
  }

  public override ngOnInit(): void {
    super.ngOnInit();
  }

  protected buildForm(formBuilder: FormBuilder): FormGroup | UntypedFormGroup {
    return formBuilder.group({
      entries: this.entries,
    });
  }

  private updateThesauri(thesauri: ThesauriSet): void {
    // TODO setup your thesauri entries here, e.g.:
    // let key = 'cod-binding-tags';
    // if (this.hasThesaurus(key)) {
    //   this.tagEntries = thesauri[key].entries;
    // } else {
    //   this.tagEntries = undefined;
    // }
  }

  private updateForm(part?: __NAME__sPart): void {
    if (!part) {
      this.form.reset();
      return;
    }
    this.entries.setValue(part.bindings || []);
    this.form.markAsPristine();
  }

  protected override onDataSet(data?: EditedObject<__NAME__sPart>): void {
    // thesauri
    if (data?.thesauri) {
      this.updateThesauri(data.thesauri);
    }

    // form
    this.updateForm(data?.value);
  }

  protected getValue(): __NAME__sPart {
    let part = this.getEditedPart(__NAME__S_PART_TYPEID) as __NAME__sPart;
    part.__NAME__s = this.entries.value || [];
    return part;
  }

  public add__NAME__(): void {
    const entry: __NAME__ = {
      // TODO: set your entry default properties...
    };
    this.entries.setValue([...this.entries.value, entry]);
    this.edit__NAME__(this.entries.value.length - 1);
  }

  public edit__NAME__(index: number): void {
    if (index < 0) {
      this._editedIndex = -1;
      this.tabIndex = 0;
      this.edited = undefined;
    } else {
      this._editedIndex = index;
      this.edited = this.entries.value[index];
      setTimeout(() => {
        this.tabIndex = 1;
      }, 300);
    }
  }

  public on__NAME__Save(entry: __NAME__): void {
    this.entries.setValue(
      this.entries.value.map((e: __NAME__, i: number) =>
        i === this._editedIndex ? entry : e
      )
    );
    this.edit__NAME__(-1);
  }

  public on__NAME__Close(): void {
    this.edit__NAME__(-1);
  }

  public delete__NAME__(index: number): void {
    this._dialogService
      .confirm('Confirmation', 'Delete __NAME__?')
      .pipe(take(1))
      .subscribe((yes) => {
        if (yes) {
          const entries = [...this.entries.value];
          entries.splice(index, 1);
          this.entries.setValue(entries);
        }
      });
  }

  public move__NAME__Up(index: number): void {
    if (index < 1) {
      return;
    }
    const entry = this.entries.value[index];
    const entries = [...this.entries.value];
    entries.splice(index, 1);
    entries.splice(index - 1, 0, entry);
    this.entries.setValue(entries);
  }

  public move__NAME__Down(index: number): void {
    if (index + 1 >= this.entries.value.length) {
      return;
    }
    const entry = this.entries.value[index];
    const entries = [...this.entries.value];
    entries.splice(index, 1);
    entries.splice(index + 1, 0, entry);
    this.entries.setValue(entries);
  }
}
```

HTML:

```html
<form [formGroup]="form" (submit)="save()">
  <mat-card>
    <mat-card-header>
      <div mat-card-avatar>
        <mat-icon>picture_in_picture</mat-icon>
      </div>
      <mat-card-title>__NAME__s Part</mat-card-title>
    </mat-card-header>
    <mat-card-content>
      <mat-tab-group [(selectedIndex)]="tabIndex">
        <mat-tab label="__NAME__s">
          <div>
            <button
              type="button"
              mat-icon-button
              color="primary"
              (click)="add__NAME__()"
            >
              <mat-icon>add_circle</mat-icon>
            </button>
          </div>
          <table *ngIf="entries.value?.length">
            <thead>
              <tr>
                <th></th>
                TODO: add model properties
              </tr>
            </thead>
            <tbody>
              <tr
                *ngFor="
                  let entry of entries.value;
                  let i = index;
                  let first = first;
                  let last = last
                "
              >
                <td>
                  <button
                    type="button"
                    mat-icon-button
                    color="primary"
                    matTooltip="Edit this __NAME__"
                    (click)="edit__NAME__(i)"
                  >
                    <mat-icon>edit</mat-icon>
                  </button>
                  <button
                    type="button"
                    mat-icon-button
                    matTooltip="Move this __NAME__ up"
                    [disabled]="first"
                    (click)="move__NAME__Up(i)"
                  >
                    <mat-icon>arrow_upward</mat-icon>
                  </button>
                  <button
                    type="button"
                    mat-icon-button
                    matTooltip="Move this __NAME__ down"
                    [disabled]="last"
                    (click)="move__NAME__Down(i)"
                  >
                    <mat-icon>arrow_downward</mat-icon>
                  </button>
                  <button
                    type="button"
                    mat-icon-button
                    color="warn"
                    matTooltip="Delete this __NAME__"
                    (click)="delete__NAME__(i)"
                  >
                    <mat-icon>remove_circle</mat-icon>
                  </button>
                </td>
                TODO: td's for properties
              </tr>
            </tbody>
          </table>
        </mat-tab>

        <mat-tab label="unit" *ngIf="edited__NAME__">
          TODO: editor control with: [model]="edited__NAME__"
          (modelChange)="on__NAME__Save($event)"
          (editorClose)="on__NAME__Close()"
        </mat-tab>
      </mat-tab-group>
    </mat-card-content>
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

Typically you should edit each single entry in a component (generated with `ng g component <NAME>-editor` where NAME is the model's name, e.g. `cod-binding-editor` for the `cod-bindings-part` component - remember to export it both from the library's module and from its barrel `public-api.ts` file), similar to the following template:

```ts
import { Component, EventEmitter, Input, OnInit, Output } from '@angular/core';
import {
  FormArray,
  FormBuilder,
  FormControl,
  FormGroup,
  Validators,
} from '@angular/forms';
import { ThesaurusEntry } from '@myrmidon/cadmus-core';

@Component({
  selector: 'tgr-...',
  templateUrl: './...component.html',
  styleUrls: ['./...component.css'],
})
export class __NAME__Component implements OnInit {
  private _model: __TYPE__ | undefined;

  @Input()
  public get model(): __TYPE__ | undefined {
    return this._model;
  }
  public set model(value: __TYPE__ | undefined) {
    this._model = value;
    this.updateForm(value);
  }

  @Output()
  public modelChange: EventEmitter<__TYPE__>;
  @Output()
  public editorClose: EventEmitter<any>;

  public form: FormGroup;
  // TODO: controls

  constructor(formBuilder: FormBuilder) {
    this.modelChange = new EventEmitter<__TYPE__>();
    this.editorClose = new EventEmitter<any>();
    // form
    // TODO: controls
    this.form = formBuilder.group({
      // TODO
    });
  }

  ngOnInit(): void {
    if (this._model) {
      this.updateForm(this._model);
    }
  }

  private updateForm(model: __TYPE__ | undefined): void {
    if (!model) {
      this.form.reset();
      return;
    }

    // TODO set controls values

    this.form.markAsPristine();
  }

  private getModel(): __TYPE__ | null {
    return {
      // TODO get values from controls
    };
  }

  public cancel(): void {
    this.editorClose.emit();
  }

  public save(): void {
    if (this.form.invalid) {
      return;
    }
    const model = this.getModel();
    if (!model) {
      return;
    }
    this.modelChange.emit(model);
  }
}
```

HTML template:

```html
<form [formGroup]="form" (submit)="save()">
  TODO
  <!-- buttons -->
  <div>
    <button
      type="button"
      color="warn"
      mat-icon-button
      matTooltip="Discard changes"
      (click)="cancel()"
    >
      <mat-icon>clear</mat-icon>
    </button>
    <button
      type="submit"
      color="primary"
      mat-icon-button
      matTooltip="Accept changes"
      [disabled]="form.invalid || form.pristine"
    >
      <mat-icon>check_circle</mat-icon>
    </button>
  </div>
</form>
```

(2) ensure the component has been added to its module's `declarations` and `exports`, and to the `public-api.ts` barrel file.

## Adding PG Editor Wrapper

Once you have the part editor, you need its wrapper page, which in turn is linked to a specific route in the context of the project. Such route is defined in the part's library when using a granular approach, or in the general PG library when using a multiple-editors approach.

(1) under your library's `src/lib` folder, add a **part editor feature component** named after the part (e.g. `ng g component note-part-feature` for `NotePartFeatureComponent` after `NotePart`).

(2) ensure that this component is both under the module `declarations` and `exports`, and in the `public-api.ts` barrel file.

(3) add the corresponding **route** in the PG library's module of the project's PG library, e.g.:

```ts
{
  path: `${__NAME___PART_TYPEID}/:pid`,
  pathMatch: 'full',
  component: __NAME__PartFeatureComponent,
  canDeactivate: [PendingChangesGuard]
},
```

(4) implement the feature **editor component** by making it extend `EditPartFeatureBase`, like in this code template:

```ts
import { Component, OnInit } from '@angular/core';
import { Router, ActivatedRoute } from '@angular/router';
import { MatSnackBar } from '@angular/material/snack-bar';

import { EditPartFeatureBase, PartEditorService } from '@myrmidon/cadmus-state';
import { ItemService, ThesaurusService } from '@myrmidon/cadmus-api';

@Component({
  selector: 'cadmus-__NAME__-part-feature',
  templateUrl: './note-__NAME__-feature.component.html',
  styleUrls: ['./note-__NAME__-feature.component.css'],
})
export class __NAME__PartFeatureComponent
  extends EditPartFeatureBase
  implements OnInit
{
  constructor(
    router: Router,
    route: ActivatedRoute,
    snackbar: MatSnackBar,
    itemService: ItemService,
    thesaurusService: ThesaurusService,
    editorService: PartEditorService
  ) {
    super(
      router,
      route,
      snackbar,
      itemService,
      thesaurusService,
      editorService
    );
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
<cadmus-__NAME__-part
  [identity]="identity"
  [data]="$any(data)"
  (dataChange)="save($event)"
  (editorClose)="close()"
  (dirtyChange)="onDirtyChange($event)"
></cadmus-__NAME__-part>
```

(5) in your app's project `part-editor-keys.ts`, add the mapping for the part just created, like e.g.:

```ts
  // itinera parts
  [PERSON_PART_TYPEID]: {
    part: ITINERA_LT
  },
```

Here, the type ID of the part (from its model in the "ui" library) is mapped to the route prefix constant `ITINERA_LT` = `itinera-lt`, which is the root route to the "pg" library module for the app.

▶️ next: [fragments](fragments.md)
