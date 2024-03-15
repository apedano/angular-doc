# 01B - Reactive forms

Reactive forms provide a model-driven approach to handling form inputs whose values change over time.

It requires 

```typescript
import { ReactiveFormsModule, FormControl, FormGroup, Validators } from '@angular/forms';

@Component({
    selector: 'app-ingredient-form',
    standalone: true,
    imports: [ReactiveFormsModule, ...],
    templateUrl: './ingredient-form.component.html',
    styleUrl: './ingredient-form.component.css'
})
```

## Form template declaration


```angular2html
<form (ngSubmit)="onSubmit()" [formGroup]="entityForm">
   <span class="p-float-label">
                        <input pInputText formControlName="name">
                        <label for="name">Name</label>
                    </span>
  <p-dropdown formControlName="season" [options]="seasons" [showClear]="true"
              placeholder="Select a Season">
  </p-dropdown>
  <small id="name-invalid" class="p-error block"
         *ngIf="season?.touched && season?.errors?.['required']">Season
    is mandatory</small>
  <button pButton type="submit" [disabled]="!entityForm.valid" label="Save"></button>
</form>
```

The `FormGroup` directive listens for the **submit event** emitted by the form element and emits an `ngSubmit` event that you can bind to a callback function. 
Add an `ngSubmit` event listener to the form tag with the `onSubmit()` callback method.

```typescript
  private createAndStoreObserver: Partial<Observer<HttpResponse<{ name: string }>>> = {
    next: (httpResponse) => {
      alert("Ingredient saved");
      this.appStateService.logIfDebug("Ingredient response data", httpResponse)
    },
    error: err => {
      console.log(err);
    }
  }
  
  onSubmit() {
    this.ingredient.name = this.name?.value;
    this.ingredientService
            .createOrUpdate(this.ingredient)
            .subscribe(this.createAndStoreObserver);
  }
```

## Form model declaration

Normally we can define the `FormGroup` in a simple way.

```typescript

ingredientForm!: FormGroup;

constructor(...) {
    const nameControl = new FormControl('');

    this.ingredientForm = new FormGroup({nameControl}); //one control only
    
  }

```


### Reactive form model declaration
It is useful when we need to preload data via abservables, for instance, when we need to prepopulate select controls
with options coming from an external HTTP call. 


```typescript
export abstract class FormComponent<T extends IdEntity> {

    protected entity!: T;
    protected entityForm!: FormGroup; //as in the template  

    constructor(private genericService: GenericService<T>, private router: Router) {
    }

    initEntity(currentRoute: ActivatedRoute) {
        currentRoute.params
            .subscribe(
                (updatedParams: Params) => {
                    if (updatedParams['id']) {
                        let id = updatedParams['id'];
                        this.genericService.getById(id).subscribe(entity => {
                            this.entity = entity;
                            this.onEntityLoaded();
                        });
                    } else {
                        this.entity = this.emptyEntity();
                        this.onEntityLoaded();
                    }
                }
            );
    }

    private onEntityLoaded(): void {
      this.initEntityForm()
              .pipe(first()) //unsibsribe automatically after the first observed value
              .subscribe((form: FormGroup) => {
                console.log('form received', form);
                this.entityForm = form;
                this.loading = false;
              });
  
    }

    protected abstract emptyEntity(): T;
  
    protected abstract initEntityForm(): Observable<FormGroup>;

    protected onSubmit() { //bind with the onSubimt event on the FormGroup submit
      this.mapFormToEntity();
      console.log('entityForm', this.entityForm);
      this.genericService.createOrUpdate(this.entity).subscribe(this.createAndStoreObserver);
    }
  
    protected abstract mapFormToEntity(): void;
  
    protected abstract getRedirectUrlAfterSave(): any[];
}
```
In the form implementing class

```typescript
@Component({
  selector: 'app-car-form',
  templateUrl: './car-form.component.html',
  styleUrls: ['./car-form.component.css']
})
export class CarFormComponent extends GenericFormComponent<Car> implements OnInit {

  teams!: Team[];
  seasons: LabelAndValue[] = Season.allToLabelAndValue();


  constructor(private teamService: TeamsService, carService: CarService, router: Router, private currentRoute: ActivatedRoute) {
    super(carService, router);
  }

  protected emptyEntity(): Car {
    return new Car(Season.SEASONS[0], '', '', '', '');
  }


  protected initEntityForm(): Observable<FormGroup<any>> {
    return this.teamService.getAll().
      pipe(
        map((tArray: Team[]) => {
          console.log('Season.toLabelAndValue(this.entity.season)', Season.toLabelAndValue(this.entity.season));
          this.teams = tArray;
          return new FormGroup({
            season: new FormControl<Number>(this.entity.season,
              [Validators.required]),
            name: new FormControl(this.entity.name!,
              [Validators.required, Validators.minLength(3)]),
            powerUnit: new FormControl(this.entity.powerUnit!,
              [Validators.required, Validators.minLength(3)]),
            team: new FormControl<Team | null>(this.getTeamByLogo(tArray, this.entity.teamId),
              [Validators.required, Validators.minLength(3)]),
            imageName: new FormControl(this.entity.imageName!, [Validators.required]),
          });
        })
      );
  }

  private getTeamByLogo(tA: Team[], logoName: string): Team | null {
    let filtered: Team[] = tA.filter((t: Team) => t.logoName == logoName);
    return filtered.length == 0 ? null : filtered[0];
  }
  
  protected mapFormToEntity(): void {
    this.entity.name = this.name?.value;
    this.entity.season = this.season?.value;
    this.entity.teamId = this.team?.value.logoName;
    this.entity.imageName = this.imageName?.value;
    this.entity.powerUnit = this.powerUnit?.value;
    console.log(this.entityForm);
  }
  
  //These methods allow the call `season?.touched && season?.errors` on the template
  get season() { return this.entityForm.get('season') }
  get name() { return this.entityForm.get('name') }
  get team() { return this.entityForm.get('team') }
  get imageName() { return this.entityForm.get('imageName') }
  get powerUnit() { return this.entityForm.get('powerUnit') }
  // get foundation() { return this.entityForm.get('foundation') }


  protected getRedirectUrlAfterSave(): any[] {
    return ['/cars'];
  }

  ngOnInit(): void {
    this.initEntity(this.currentRoute);
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
export class NameUniqueValidator<T extends NameEntity> implements AsyncValidator {

  constructor(private appStateService: AppStateService, private modelService: GenericNameBasedService<T>, private validateUppercase: boolean) { }

  validate(control: AbstractControl<any, any>): Promise<ValidationErrors | null> | Observable<ValidationErrors | null> {
    const name: string = control.value;
    this.appStateService.logIfDebug("Called NameUniqueValidator with", name)
    if (name.length == 0) {
      return of(null);
    }
    let nameToCheck = this.validateUppercase ? name.toUpperCase() : name;
    return this.modelService.getByName(nameToCheck).pipe(
            first(), //this makes the returned observable finite.
            tap(existingUnit => this.appStateService.logIfDebug("NameUniqueValidatorDirective check: ", existingUnit == undefined)),
            map(existingUnit => existingUnit == undefined ? null : { nameUnique: true }),
            catchError(() => of(null))
    );
  }
}
```
Where the `unitService.getByName(unitName.toUpperCase())` returns `Observable<Unit>`; we make this finite with
the `first()` pipe operator.

### Adding async validators to reactive forms

First, the validator has to be injected into the form class.

```typescript
constructor(private uniqueNameValidator: UnitNameUniqueValidator) {
  this.nameUniqueValidator = new NameUniqueValidator(appStateService, ingredientService, false);
  ...
}
```

Then the validator can be added to the specific control

```typescript
const nameControl = new FormControl('', {
  validators: [Validators.required, Validators.minLength(3)],
  asyncValidators: [
    this.uniqueNameValidator.validate.bind(this.uniqueNameValidator), //creates an AsyncValidatorFn out of the AsyncValidator
  ],
  updateOn: 'blur', //apply validation at the blur event
});
```
We need to pass ``asyncValidators?: AsyncValidatorFn | AsyncValidatorFn[] | null;`` but ``UnitNameUniqueValidator implements AsyncValidator``,
so we create the ``AsyncValidatorFn`` from the `AsyncValidator` with the `bind`; for the given `validate` function,
creates a **bound function** that has the same body as the original function.
The argument is an object to which the `this` keyword can refer inside the new function (which is the validator itself).
So, if inside the new function, we call `this.`, the bound function will refer to the argument object.

## Update or set value

```angular2html
<button type="button" (click)="updateName()">Update Name</button>
```

```typescript
updateName() {
  this.ingredientForm.patchValue({
    name: 'MyIngredient',
    address: {
      street: '123 Drew Street', //sub control in form group
    },
  });
}
```


