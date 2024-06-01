# 05 - Custom form control creation

https://blog.angular-university.io/angular-custom-form-controls/

How to make a standard component a form control that can be used in either template driven or reactive
forms.

* if we are using **template driven forms**, we will be able to plug the custom component to the form simply by
  using `ngModel`
* in case of **reactive forms**, we will be able to add the custom component to the form using `formControlName` (
  or `formControl`)

## Standard form controls

```angular2html

<input placeholder="Course title" formControlName="title">

<input type="checkbox" placeholder="Free course" formControlName="free">

<textarea placeholder="Description" formControlName="longDescription"></textarea>
```

Form controls are associated with a `formControlName`; this is how the HTML element gets binded to the form.

## The `ControlValueAccessor` interface

At every interaction with the control, Angular checks value and validity of the control and of the form.
The **Angular forms module** is applying to each native HTML element, called **control value accessor**, a **built-in
Angular directive**,
which will be **responsible for tracking the value of the field, and communicate it back to the parent form**.

The control value accessor exist for each HTML native element, here one example for the checkbox input:

```typescript
@Directive({
    selector:
        'input[type=checkbox][formControlName],
    input[type = checkbox][formControl],
    input[type = checkbox][ngModel]',

})
export class CheckboxControlValueAccessor implements ControlValueAccessor {
....
}
```

The directive targets HTML inputs of type checkbox, but only if the `ngModel`, `formControl` or
`formControlName` attributes are applied to it.

If this directive only targets checkboxes, then what about all the other types of form controls, like text inputs or
text areas?

For a custom form control, the same `ControlValueAccessor` implementing directive must be built.

### Methods in `ControlValueAccessor`

The interface defines a set of methods that are not meant to be called by the control user in pruduction code, but
rather
by the `FormModule` during the value and validity check cycles, to facilitate communication between our form control and
the parent form.

| **Method**          | **Description**                                                                                                                                                                                                                                                    |
|---------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `writeValue`        | this method is called by the Forms module to write a value into a form control                                                                                                                                                                                     |
| `registerOnChange`  | When a form value changes due to user input, we need to report the value back to the parent form. <br/>This is done by calling a callback, that was initially registered with the control using this method                                                        |
| `registerOnTouched` | When the user first interacts with the form control, the control is considered to have the status `touched`, which is useful for styling.<br/> In order to report to the parent form that the control was touched, we need to use a callback registered using this |
| `setDisabledState`  | Form controls can be enabled and disabled using the Forms API. This state can be transmitted to the form control via the setDisabledState method                                                                                                                   |

## Building a custom form control

The example is a numeric ranged value input, which gets status invalid if the current value is outside the given range.

```typescript
@Component({
    selector: 'choose-quantity',
    template: `
     <div class="choose-quantity">
        <mat-icon (click)="onRemove()" color="primary">minus_box</mat-icon>
        <div class="quantity">{{quantity}}</div>
        <mat-icon (click)="onAdd()" color="primary">add_box</mat-icon>
     </div>
`,
    styleUrls: ["choose-quantity.component.scss"]
})
export class ChooseQuantityComponent {

    quantity = 0;

    @Input()
    increment: number;

    onAdd() {
        this.quantity += this.increment;
    }

    onRemove() {
        this.quantity -= this.increment;
    }
}
```

Its usage within a form

```angular2html

<div [formGroup]="form">
    <choose-quantity [increment]="10" formControlName="totalQuantity"></choose-quantity>
    ...
</div>
```

```typescript
@Component(...)
export class ExampleForms implements OnInit {

    form = this.fb.group({
        ...other fields
        totalQuantity: [60, [Validators.required, Validators.max(100)]]
    });

    constructor(private fb: FormBuilder) {
    }
}
```

With this version of the control, Angular will return the following error:

```
ERROR Error: Must supply a value for form control with name: 'totalQuantity'.
at forms.js:2692
...
```

We need to make the component implement the `ControlValueAccessor` interface.

### Implementing the `ControlValueAccessor` interface

```typescript
@Component({
    selector: 'choose-quantity',
    templateUrl: "choose-quantity.component.html",
    styleUrls: ["choose-quantity.component.scss"],
    providers: [
        { //provider for the form component
            provide: NG_VALUE_ACCESSOR,
            multi: true,
            useExisting: ChooseQuantityComponent
        }
    ]
})
export class ChooseQuantityComponent implements ControlValueAccessor {

    quantity = 0;

    @Input()
    increment: number;

    onAdd() {
        this.markAsTouched(); //the interaction with the controls changes the touched status
        if (!this.disabled) {
            this.quantity += this.increment;
            //the callback function which communicates the updated value is called
            this.onChange(this.quantity);
        }
    }

    onRemove() {
        this.markAsTouched();
        if (!this.disabled) { //applied only if not disabled
            this.quantity -= this.increment;
            //the callback function which communicates the updated value is called
            this.onChange(this.quantity);
        }
    }


    //called when the parent forms sets a value
    @Override
    writeValue(quantity: number) {
        this.quantity = quantity;
    }


    //callback function for the OnChange event of the control. Initialized with empty body 
    //(before the first call to the registerOnChange is done by the form module to avoid side effects before the register
    // method is called)
    onChange = (quantity) => {
    };

    // when called by the form module, the input callback function (used to return the value to the form) 
    // is passed as parameter. When the control needs to communitcate its value to the parent form after a user interaction
    // the callback function can be called with the updated value
    @Override
    registerOnChange(onChange: any) {
        this.onChange = onChange; //we set the callback function to communicate the value to the parent form
    }


    //callback function for the OnTouched event of the control. Initialized with empty body 
    //(before the first call to the registerOnTouched is done by the form module to avoid side effects before the register
    // method is called)
    onTouched = () => {
    };


    // used to change the initial css class `ng-untouched` to `ng-touched`  when the control gets touched by the user, 
    // meaning the user has tried to interact with it at least once
    // here we set the input callback function as the class attribute onTouched to propagate the event to the parent form.
    @Override
    registerOnTouched(onTouched: any) {
        this.onTouched = onTouched;
    }

    touched = false; //initial value for touched

    markAsTouched() {
        if (!this.touched) {
            this.onTouched(); //called only at the first interaction (touched is still false)
            this.touched = true;
        }
    }

    disabled = false; //initial value for disabled

    //used to change the disable state on the control from the parent form
    @Override
    setDisabledState(disabled: boolean) {
        this.disabled = disabled;
    }
}
```

### Dependency injection configuration

```typescript
@Component({
    ...
        providers:
[
    {
        provide: NG_VALUE_ACCESSOR,
        multi: true,
        useExisting: ChooseQuantityComponent
    }
]
})

export class ChooseQuantityComponent implements ControlValueAccessor {
....
}
```

This configuration **registers the custom form control as a known value accessor in the dependency injection system**:
it adds the component to the ``NG_VALUE_ACCESSOR`` unique dependency injection key (also known as an **injection token
**).

Notice the `multi:true` flag, this means that this dependency provides a list of values, and not only one value (we have
multiple components under the same injection token).

This is normal because there are many value accessors registered with Angular Forms besides our own.

For example, all the built-in value accessors for standard text inputs, checkboxes, etc. are also registered under
NG_VALUE_ACCESSOR.

This way, whenever the Angular forms module needs the full list of all the value accessors available, all it has to do
is to inject NG_VALUE_ACCESSOR.

## Custom control validation

The component implementing the `ControlValueAccessor` interface is now **capable of participating in the form validation
process** and is already fully compatible with the **built-in validators** (e.g. `required` and `max`).

### The `Validator` interface

In case a custom internal validation is needed, the component can implement the `Validator` interface, which defines

| **Method**                  | **Description**                                                                                                                                                                                                                                                                                                                             |
|-----------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `validate`                  | This method is used to validate the current value of the form control. This method will be **called whenever a new value is reported to the parent form**. The method needs to return `null` if no errors are found, or an **error object** containing all the details needed to correctly display a meaningful error message to the user.  |
| `registerOnValidatorChange` | This will **register a callback** that will allow us to **trigger the validation of the custom control on demand**. We **don't need to do this when new values are emitted, as validation is already triggered in that case**. We only need to call this method if some other input that also affects the behavior of validate has changed. |

This is the implementation in the custom control

```typescript
@Component({
    selector: 'choose-quantity',
    templateUrl: "choose-quantity.component.html",
    styleUrls: ["choose-quantity.component.scss"],
    providers: [
        { //provider for the form component
            provide: NG_VALUE_ACCESSOR,
            multi: true,
            useExisting: ChooseQuantityComponent
        },
        {
            //register our custom component with the NG_VALIDATORS injection token
            provide: NG_VALIDATORS,
            multi: true,
            useExisting: ChooseQuantityComponent
        }

    ]
})
export class ChooseQuantityComponent implements ControlValueAccessor, Validator {

    quantity = 0;

    @Input()
    increment: number;

    @Override
    validate(control: AbstractControl): ValidationErrors | null {
        const quantity = control.value;
        if (quantity <= 0) {
            return {  //object with error info
                mustBePositive: {
                    quantity
                }
            };
        }
        return null; //no error return
    }

...
}
```








