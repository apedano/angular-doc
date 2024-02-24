# 01A - Template driven form

## Import module

For `standalone` components

```typescript

import {FormsModule, Validators} from '@angular/forms';

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
  onSubmit(unitForm): NgForm {
        console.log("form submitted", unitForm, unitForm.form.controls['unitName']);
    }

    @ViewChild('myForm') form: any;  
```

The form tag defines also the `ngSubmit` function.

* Binds itself to the `<Form>` directive
* Creates a top-level `FormGroup` instance
* Creates `FormControl` https://angular.io/api/forms/FormControl instance for each of child control, which has ngModel
  directive.
* Creates `FormGroup` instance for each of the `NgModelGroup` directive.

## Validation

### Build in validators

Built in validators in HTML 5

| Name                | Example                                                                   |
|:--------------------|:--------------------------------------------------------------------------|
| Required Validation | `<input type="text" id="name" name="userName" required>`                  |  
| Minlenght           | `<input type="text" id="name" name="userName" minlength="20">`            |
| Maxlength           | `<input type="text" id="name" name="userName" required maxlength="20">`   |
| Pattern             | `<input type="text" id="userName" name="username" pattern="^[a-zA-Z]+$">` |
| Email               | `<input type="text" id="email" name="email" email>`                       |

### Custom validators

ou can also create custom validators by creating a function that returns a ValidatorFn or an array of ValidatorF

### FormControl states

| Name     | Description                                                                                                       |
|:---------|:------------------------------------------------------------------------------------------------------------------|
| Dirty    | It becomes `true` when the user modifies the value of the control.                                                |  
| Touched  | It becomes true when the user interacts with the control (e.g., clicks inside the input and then clicks outside). |
| Pristine | It becomes `true` if the control value has not been changed (it's in its initial state).                          |

```typescript
<pre>Valid ? {{unitForm.form.controls.name?.valid}}</pre> 
<pre> Dirty ? {{unitForm.form.controls.name?.dirty}</pre>
<pre> Touched ? {{unitForm.form.controls.name?.touched}} </pre>
```

