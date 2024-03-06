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

```angular2html
<input type="number" matInput [(ngModel)]="unit.conversionRatio" name="conversion"
  placeholder="Inserisci un numero" pattern="^[0-9]+(,[0-9]+)?$">
<mat-hint>Separatore: ,</mat-hint>
@if (unitForm.form.controls['conversion'].hasError('pattern') ||
unitForm.form.controls['conversion'].hasError('number')) {
<mat-error>Il fattore di conversione deve essere un <strong>numero</strong></mat-error>
}
```

### Adding cross fiels validation

https://angular.io/guide/form-validation#:~:text=do%20not%20match.-,Adding%20cross%2Dvalidation%20to%20reactive%20forms,-link

A cross-field validator is a custom validator that compares the values of different fields in a form and accepts or rejects them in combination.

```typescript
@Directive({
  selector: '[baseUnitFormValidator]',
  providers: [
    {
      provide: NG_VALIDATORS,
      useExisting: BaseUnitFormValidatorDirective,
      multi: true,
    },
  ],
  standalone: true
})
export class BaseUnitFormValidatorDirective implements Validator {

  constructor(private appStateService: AppStateService) { }


  validate(control: AbstractControl<any, any>): ValidationErrors | null {
    return this.baseUnitFormValidatorFunction(control);
  }

  //this is the method to implement the Validator interface
  private baseUnitFormValidatorFunction: ValidatorFn =
    (control: AbstractControl): ValidationErrors | null => {
      
      this.appStateService.logIfDebug("BaseUnitFormValidatorDirective called");  
      const unitIsBase = control.get('unitIsBase')
      const baseUnit = control.get('baseUnit');

      validationisValid ? null  
              : { baseUnitInvalid: true }; //validation fails (ValidationErrors)
    };

}
```

The usage requires the import in the using component and the directive being added to the form element

```angular2html
<form #unitForm="ngForm" (ngSubmit)="onSubmit(unitForm)" baseUnitFormValidator>
```

## Check validation state

```angular2html
<pre>Valid ? {{unitForm.form.controls['unitName'].valid}}</pre>
<pre>Value unitName: {{unitForm.form.controls['unitName'].value}}</pre>
<pre>Form valid: {{unitForm.form.valid}}</pre>
<pre>Form Errors: {{unitForm.errors | json}}</pre>
<!-- unitForm.controls returns a map, the keyvalue pipe returns 
  the iterator of key value -->
@for(controlkv of unitForm.controls | keyvalue; track controlkv) {
  @if(controlkv.value.errors != null) {
    <pre>{{controlkv.key}} errors</pre>
    <pre>{{controlkv.value.errors | json}}</pre>
  }
}
```

## Async validator

Asynchronous validators implement the `AsyncValidatorFn` and `AsyncValidator` interfaces. 
These are very similar to their synchronous counterparts, with the following differences.

* The `validate()` functions must return a Promise or an observable,
* The observable returned must be **finite**, meaning **it must complete at some point**. 
  * To convert an infinite observable into a finite one, pipe the observable through a filtering operator such as `first`, `last`, `take`, or `takeUntil`.

### Implement the AsyncValidator

```typescript
@Injectable({ providedIn: 'root' })
export class UnitNameUniqueValidator implements AsyncValidator {
    
    constructor(private unitService: UnitService, private appStateService: AppStateService) {}
  
    validate(control: AbstractControl<any, any>): Promise<ValidationErrors | null> | Observable<ValidationErrors | null> {
        const unitName: string = control.value;
      if(unitName.length == 0) {
        return of(null);
      }
      return this.unitService.getByName(unitName.toUpperCase()).pipe(
        first(), //this makes the returned observable finite.
        tap(existingUnit => this.appStateService.logIfDebug("UnitNameUniqueValidatorDirective is check valid", existingUnit == undefined)),
        map(existingUnit => existingUnit == undefined ? null : { unitNameUnique: true }), 
        catchError(()=>of(null))
      );
    }
}
```
Where the `unitService.getByName(unitName.toUpperCase())` returns `Observable<Unit>`; we make this finite with 
the `first()` pipe operator. 

### Adding async validators to reactive forms

First, the validator has to be injected into the form class.

```typescript
constructor(private uniqueNameValidator: UnitNameUniqueValidator) { }
```

Then the validator can be added to the specific control

```typescript
const unitNameControl = new FormControl('', {
  asyncValidators: [
    this.unitNameControl.validate.bind(this.uniqueNameValidator),
  ],
  updateOn: 'blur',
});
```

### Adding async validators to template-driven forms

To use an async validator in template-driven forms, create a new directive 
and register the `NG_ASYNC_VALIDATORS` provider on it.

```typescript
@Directive({
  selector: '[unitNameUnique]',
  providers: [
    {
      provide: NG_ASYNC_VALIDATORS,
      useExisting: forwardRef(() => UnitNameUniqueValidatorDirective),
      multi: true,
    },
  ],
  standalone: true,
})
export class UnitNameUniqueValidatorDirective implements AsyncValidator {

  constructor(private uniqueNameValidator: UnitNameUniqueValidator) { }
  
  validate(control: AbstractControl<any, any>): Promise<ValidationErrors | null> | Observable<ValidationErrors | null> {
      return this.uniqueNameValidator.validate(control);
  }
}
```
And then add the validator to the control

```angular2html
<input matInput placeholder="es. L per 'litri'" appAllCaps [(ngModel)]="unit.name" name="unitName" [ngModelOptions]="{updateOn: 'blur'}" required unitNameUnique>
```

### Performance optimization

When the async validator is backed by an **HTTP call**, the default behavior based on onchange of the value, it 
would **trigger a new call each keystroke**, with the ``[ngModelOptions]="{updateOn: 'blur'}"`` we trigger the validation on _blur_ event.