# 01A - Template driven form

## Import module

For `standalone` components

```typescript

import { FormsModule, Validators } from '@angular/forms';

@Component({
  selector: 'app-unit-form',
  standalone: true,
  imports: [FormsModule],
  ...
})
export class UnitFormComponent implements OnInit {
    unit: Unit;
}
```

### Template reference variable

It's a reference variable built in the template

```angular2html
<form (ngSubmit)="onSubmit(unitForm)" #unitForm="ngForm">
    ...

    <input matInput placeholder="es. L" [(ngModel)]="unit.name" name="unitName"> <!-- FormControl -->
    
    <button type="submit" class="btn btn-success" [disabled]="!unitForm.form.valid">Submit</button>
    ...
</form>
```

```typescript
  onSubmit(unitForm: NgForm) {
    onSubmit(unitForm: NgForm) {
        console.log("form submitted", unitForm, unitForm.form.controls['unitName']);
    }
  }
```
The form tag defines also the the `ngSubmit` function.

* Binds itself to the `<Form>` directive
* Creates a top-level `FormGroup` instance
* Creates `FormControl` https://angular.io/api/forms/FormControl instance for each of child control, which has ngModel directive.
* Creates `FormGroup` instance for each of the `NgModelGroup` directive.


