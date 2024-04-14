# 02-Dialog windows

https://material.angular.io/components/dialog/api

## Dialog component creation

```typescript
import { MAT_DIALOG_DATA, MatDialogModule, MatDialogRef } from '@angular/material/dialog';
...

@Component({
  selector: 'app-new-ingredient-dialog',
  standalone: true,
  templateUrl: './new-ingredient-dialog.component.html',
  styleUrl: './new-ingredient-dialog.component.css',
  imports: [MatDialogModule, IngredientFormComponent]
})
export class NewIngredientDialogComponent {

  constructor(private dialogRef: MatDialogRef<NewIngredientDialogComponent>,
    @Inject(MAT_DIALOG_DATA) data: any, private appStateService: AppStateService) {}

  closeAndResult($event: Ingredient) {
    this.appStateService.logIfDebug("Closing NewIngredientDialogComponent dialog with result:", $event);
    this.dialogRef.close($event);
  }

}
```

and the template

```angular2html
<h2 mat-dialog-title>Inserisci nuovo ingrediente</h2>
<mat-dialog-content><app-ingredient-form (newIngredient)="closeAndResult($event)" ></app-ingredient-form></mat-dialog-content>
<!-- <mat-dialog-actions> -->
  <!-- <button mat-button mat-dialog-close>Cancel</button> -->
  <!-- The mat-dialog-close directive optionally accepts a value as a result for the dialog. -->
  <!-- <button mat-button [mat-dialog-close]="true">Delete</button> -->
<!-- </mat-dialog-actions> -->
```

We see that the dialog component content is a form component, which has an output which emits the submitted new ingredient:

```typescript
@Output('newIngredient') ingredientEmitter: EventEmitter<Ingredient> = new EventEmitter();
```
This output is bounded to the internal method ``closeAndResult``

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


### Passing data to the dialog

```typescript
let dialogRef = dialog.open(YourDialog, {
data: { name: 'austin' },
});
```

```typescript
@Component({
  selector: 'your-dialog',
  template: 'passed in {{ data.name }}',
})
export class YourDialog {
  constructor(@Inject(MAT_DIALOG_DATA) public data: {name: string}) { }
}
```
And the template is 

```angular2html
<ng-template let-data>
  Hello, {{data.name}}
</ng-template>
```

## Using the `NewIngredientDialogComponent`


```typescript
openNewIngredientDialog() {
    let newIngredientDialogRef = 
        this.dialog.open(NewIngredientDialogComponent);
    // this.dialog.open(NewIngredientDialogComponent, {
    //   data: {
    //     name: 'panda',
    //   },
    // });
}
```

Then we can subscibe to one of the dialog events to get the result

```typescript
newIngredientDialogRef.afterClosed().subscribe(result => {
  console.log("Dialog rsult", result); // new ingredient | undefined
});
```

    
}

 