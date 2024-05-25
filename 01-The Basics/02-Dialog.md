# 02-Dialog windows

https://material.angular.io/components/dialog/api

## Dialog component creation

```typescript
import { MAT_DIALOG_DATA, MatDialogModule, MatDialogRef } from '@angular/material/dialog';
...

@Component({
    selector: 'app-recipe-ingredient-dialog',
    standalone: true,
    imports: [MatDialogModule, RecipeIngredientComponent],
    templateUrl: './recipe-ingredient-dialog.component.html',
    styleUrl: './recipe-ingredient-dialog.component.css'
})
export class RecipeIngredientDialogComponent {

    constructor(private dialogRef: MatDialogRef<RecipeIngredientDialogComponent>,
                @Inject(MAT_DIALOG_DATA) public data: {recipeIngredient: RecipeIngredient}, private appStateService: AppStateService) {}

    closeAndResult($event: RecipeIngredient) {
        this.appStateService.logIfDebug("Closing RecipeIngredientComponent dialog with result:", $event);
        this.dialogRef.close($event);
    }
}

```

and the template

```angular2html
<h2 mat-dialog-title>Modifica ingrediente nella ricetta</h2>
<mat-dialog-content>
    <app-recipe-ingredient 
            [recipeIngredient]="data.recipeIngredient" 
            (value)="closeAndResult($event)" 
    ></app-recipe-ingredient>
</mat-dialog-content>
```

### Binding with the wrapped component

The dialog component can bind to the wrapped component input and binding

```
[recipeIngredient]="data.recipeIngredient"  -> @Input() recipeIngredient: RecipeIngredient = new RecipeIngredient();
(value)="closeAndResult($event) -> @Output() value: EventEmitter<RecipeIngredient> = new EventEmitter();
```

### Passing data to the dialog

The dialog compoment has an injected `MAT_DIALOG_DATA` with an expected object format `data: {recipeIngredient: RecipeIngredient}`

The component creating the component can pass the expected data to the dialog via the data object

```typescript
let recipeIngredientDialogRef = this.dialog.open(RecipeIngredientDialogComponent, {
      width: '600px',
      data: { 'recipeIngredient': recipeIngredient} //pass data to the dialog
});
```

### Referencing the dialog internally and return a dialog result

The ``NewIngredientDialogComponent`` received an injected reference to the dialog:

```typescript
private dialogRef: MatDialogRef<NewIngredientDialogComponent>
```
With this in place we can return the result of the dialog (from the submitted item from the form component output binding)

```typescript
closeAndResult($event: Ingredient) {
    this.dialogRef.close($event);
  }
```


## Using the `NewIngredientDialogComponent`

```typescript
openEditDialog(recipeIngredient: RecipeIngredient) {
    this.appStateService.logIfDebug('Edit dialog open with', recipeIngredient);

    let recipeIngredientDialogRef = this.dialog.open(RecipeIngredientDialogComponent, {
        width: '600px',
        data: { 'recipeIngredient': recipeIngredient} //pass data to the dialog
    });
    //we can subscribe to one of the dialog events to get the result
    recipeIngredientDialogRef.afterClosed().subscribe(result => {
        this.appStateService.logIfDebug("Result from dialog", result);
        this.recipeIngredientsOutput.emit(this.recipeIngredients);
    });
}
```
    
}

 