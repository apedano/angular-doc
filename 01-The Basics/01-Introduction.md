# 01-Introduction

https://blog.angular.io/introducing-angular-v17-4d7033312e4b


## Create the first application

After Angular CLI installation, we can install the first app

```bash
npm install -g @angular/cli
```

Create the application

```bash
ng new angular-recipes
```

Run the application

```bash
cd anugular-recipes
ng serve
```

# 02-Components

## Create a new component

```bash
ng generate component home
```

It will generate a new `home` component in the subfolder.

## Add the component to the `app-component` layout

The start of the application is indicated in the `main.ts`

```js
import {bootstrapApplication, provideProtractorTestingSupport} from '@angular/platform-browser';
import {AppComponent} from './app/app.component';

bootstrapApplication(AppComponent, {providers: [provideProtractorTestingSupport()]}).catch((err) =>
  console.error(err),
);
```
It is shown that the application starts with the `AppComponent` as root component.

We can change the layout of the root component in the `app.component.ts` to add the selector of the new component `app-home`.

We need to import the component:

```js
import {HomeComponent} from './home/home.component';
```

and add it to the list of imports of the root component:

```js
imports: [HomeComponent],
```
So we can use the selector in the template:

```html
template: `
    <header class="brand-name">
        <img class="brand-logo" src="/assets/logo.svg" alt="logo" aria-hidden="true" />
    </header>
    <section class="content">
        <app-home></app-home>
    </section>
  `,
```
Since Angular 17 **components are standalone by default**, meaning that they don't have to be attached to
a `ngModule`.

```
 standalone: true,
```
### Add features to ``HomeComponent``

We can then add a template to the new component and component specific style rules  to the CSS file.

# 03-Interfaces

**Interfaces** are custom data types for your app.

Angular uses _TypeScript_ to take advantage of working in a strongly typed programming environment.

## Create a new interface
```bash
ng generate interface housinglocation
```
## Instanciate the new interface

In a typescript class we can first import the interface:

```js
import {HousingLocation} from '../housinglocation';
...
export class HousingLocationComponent {

    readonly baseUrl = 'https://angular.dev/assets/tutorials/common';

    housingLocation: HousingLocation = {
        id: 9999,
        name: 'Test Home',
        ...
        photo: `${this.baseUrl}/example-house.jpg`,
        ...
    }
}
```
## Input decorator

`@Input()` decorator, allows components to share data. The direction of the data sharing is **from parent component to child component**.

Learn more in the [Accepting data with input properties](https://angular.dev/guide/components/inputs) and [Custom events with outputs guides](https://angular.dev/guide/components/outputs).

We create the input in the `HouseLocationComponent` passed by the `HouseComponent`.

We first create the input by importing the classes and adding the decorated attribute in
`HouseLocationComponent` class

```js
import {Input} from '@angular/core';
import {HousingLocation} from '../housinglocation';
```

```typescript
@Input() housingLocation!: HousingLocation;
```

You have to add the ! because the input is expecting the value to be passed.
The initial value will be passed in from the input of the parent component.

We can now pass the input from the parent to the child addressing the input in the child component

```typescript
<section class="results">
  <app-housing-location [housingLocation]="housingLocation"></app-housing-location>
</section>
```

## Input binding

When adding a property binding to a component tag, we use the `[attribute] = "value"` syntax to notify 
Angular that the assigned value **should be treated as a property from the component class and not a string value**.

The value on the right handside is the **name of the property from the `HomeComponent`**.

## Interpolation

https://angular.dev/guide/templates/interpolation

Using the `{{ expression }}` in Angular templates, you can render values from properties, 
`Inputs` and valid JavaScript expressions.

In the `HouseLocationComponent` template we can refer to the input attribute

```html
<section class="listing">
  <img
    class="listing-photo"
    [src]="housingLocation.photo"
    alt="Exterior photo of {{ housingLocation.name }}"
    crossorigin
  />
  <h2 class="listing-heading">{{ housingLocation.name }}</h2>
  <p class="listing-location">{{ housingLocation.city }}, {{ housingLocation.state }}</p>
</section>
```
In this updated template code you have used property binding to bind the `housingLocation.photo` 
to the `src` attribute. The `alt` attribute uses interpolation to give more context to the alt text of the image.

You use interpolation to include the values for name, city and state of the housingLocation property.

## Control flow directives

### `*ngFor`

Add the `CommonModule` to the host component.

```typescript
import {CommonModule} from '@angular/common';
...
@Component({
    selector: 'app-home',
    standalone: true,
    imports: [CommonModule, HousingLocationComponent],
    templateUrl: './home.component.html',
    styleUrl: './home.component.css'
})
```

```typescript
<section class="results">
    <app-housing-location
        *ngFor="let housingLocation of housingLocationList"
        [housingLocation]="housingLocation"
      ></app-housing-location>
</section>
```

## Angular services

Angular services provide a way for you to separate Angular app data and functions that can be used by multiple components in your app. 
To be used by multiple components, a service must be made **injectable**. 
Services that are injectable and used by a component **become dependencies of that component**. 
The component depends on those services and can't function without them.

### Dependency injection

https://angular.dev/guide/di
Dependency injection is the mechanism that manages the dependencies of an app's components and the services that other components can use.

### Service creation

```bash
ng generate service housing --skip-tests
```
The command will generate an `HousingService` class in the project, where we can pass the static
list of houses originally in the `HomeCompoment` and then add the methods

```typescript
getAllHousingLocations(): HousingLocation[] {
    return this.housingLocationList;
  }
  getHousingLocationById(id: number): HousingLocation | undefined {
    return this.housingLocationList.find((housingLocation) => housingLocation.id === id);
  }
```

### Service injection

In the `HomeComponent`

```typescript
import {HousingService} from '../housing.service';
```
```typescript
export class HomeComponent {

    housingLocationList: HousingLocation[] = [];

    constructor(private housingService: HousingService) {
        this.housingLocationList = this.housingService.getAllHousingLocations();
    }

}

```

## Routing 
https://angular.dev/tutorials/first-app/routing

Routing is the ability to navigate from one component in the application to another. 
In _Single Page Applications_ (**SPA**), only parts of the page are updated to represent the requested view for the user.

The [Angular Router](https://angular.dev/guide/routing) enables users to declare routes and specify which component 
should be displayed on the screen if that route is requested by the application.

### Adding routes

In the `src/app` directory, create a file called `routes.ts`. 
This file is where we will define the routes in the application.

In `main.ts`, make the following updates to enable routing in the application:

```typescript
import {provideRouter} from '@angular/router';
import routeConfig from './app/routes';
```

Update the call to `bootstrapApplication` to include the routing configuration:

```typescript
bootstrapApplication(AppComponent, {
  providers: [provideProtractorTestingSupport(), provideRouter(routeConfig)],
}).catch((err) => console.error(err));
```

In `src/app/app.component.ts`, update the component to use routing

```typescript
import {RouterModule} from '@angular/router';

...
@Component({
    imports: [HomeComponent, RouterModule],
    template: `
          <main>
          <a [routerLink]="['/']">
            <header class="brand-name">
              <img class="brand-logo" src="/assets/logo.svg" alt="logo" aria-hidden="true" />
            </header>
          </a>
          <section class="content">
            <router-outlet></router-outlet>
          </section>
        </main>
    `,
    
})

```

The `<router-outlet>` tag is the container for the rendered component linked to the route

### Add specific routes

In the `routes.ts` file

```typescript
import {Routes} from '@angular/router';
import {HomeComponent} from './home/home.component';
import {DetailsComponent} from './details/details.component';

const routeConfig: Routes = [
    {
        path: '',
        component: HomeComponent,
        title: 'Home page',
    },
    {
        path: 'details/:id',
        component: DetailsComponent,
        title: 'Home details',
    },
];
export default routeConfig;
```

That is what we import in the `main.ts` file

### Routing with route parameters

Route parameters enable you to include dynamic information as a part of your route URL. 
To identify which housing location a user has clicked on you will use the id property of the HousingLocation type.

We can add a `routerLink` to the `HousingLocationComponent` in order to create the link from the list to the specific house

We first add the imports:

```typescript
import {RouterModule} from '@angular/router';
...
import {RouterModule} from '@angular/router';

@Component({
    ...
    imports: [RouterModule],
    ...
})
export class HousingLocationComponent {
    ...
}
```
Then we can add the link to the template

```angular2html
<a [routerLink]="['/details', housingLocation.id]">Learn More</a>
```

### Get the route parameter

We now need to get the `id` parameter from the route to show the details of the specific house.

```typescript
import {ActivatedRoute} from '@angular/router';

export class DetailsComponent {
  housingLocationId = -1;
  constructor(private route: ActivatedRoute) {
      this.housingLocationId = Number(this.route.snapshot.params['id']);
  }
}
```

## Forms

### What approaches to choose

| Forms                 | Details                                                                                                                                                                                                                                                                                                                                                                                                             |
|-----------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Reactive forms**    | Provide direct, explicit access to the underlying form's object model. Compared to template-driven forms, they are more robust: they're more scalable, reusable, and testable. If forms are a key part of your application, or you're already using reactive patterns for building your application, use reactive forms.                                                                                            |
| **Template-driven forms** | Rely on directives in the template to create and manipulate the underlying object model. They are useful for adding a simple form to an app, such as an email list signup form. They're straightforward to add to an app, but they don't scale as well as reactive forms. If you have very basic form requirements and logic that can be managed solely in the template, template-driven forms could be a good fit. |

We will use **Reactive forms**

### Add form imports 

To the `DetailsComponent` 

```typescript
import {FormControl, FormGroup, ReactiveFormsModule} from '@angular/forms';
...
imports: [ ReactiveFormsModule]
...
export class DetailsComponent {
    
    applyForm = new FormGroup({
        firstName: new FormControl(''),
        lastName: new FormControl(''),
        email: new FormControl(''),
    });
    ...
    submitApplication() {
        this.housingService.submitApplication(
            this.applyForm.value.firstName ?? '',
            this.applyForm.value.lastName ?? '',
            this.applyForm.value.email ?? '',
        );
    }
}
```

In Angular, `FormGroup` and `FormControl` are types that enable you to build forms. 
The `FormControl` type can provide a default value and shape the form data. 
In this example firstName is a string and the default value is empty string.

and the form temlpate

```angular2html

<form [formGroup]="applyForm" (submit)="submitApplication()">
    <label for="first-name">First Name</label>
    <input id="first-name" type="text" formControlName="firstName"/>
    <label for="last-name">Last Name</label>
    <input id="last-name" type="text" formControlName="lastName"/>
    <label for="email">Email</label>
    <input id="email" type="email" formControlName="email"/>
    <button type="submit" class="primary">Apply now</button>
</form>
```

## Search functionality

The app will enable users to search through the data provided by your app and display only the results that match the entered term.

We change the `HomeComponent` to add a searching functionality.

## HTTP service call

Up until this point your app has read data from a static array in an Angular service. 
The next step is to use a JSON server that your app will communicate with over HTTP. 
The HTTP request will simulate the experience of working with data from a server.

### JSON server configuration

```bash
npm install -g json-server
```

In the root directory of your project, create a file called `db.json` and copy the json list of 
housing location from the `HousingService`.
This is where you will store the data for the json-server.

To test the server

```bash
json-server --watch db.json
```

It should expose the file content on `http://localhost:3000/locations`

### Service call implementation

```typescript
async getAllHousingLocations(): Promise<HousingLocation[]> {
    const data = await fetch(this.externalServiceUrl);
    return (await data.json()) ?? [];
  }

  async getHousingLocationById(id: number): Promise<HousingLocation | undefined> {
    const data = await fetch(`${this.externalServiceUrl}/${id}`);
    return (await data.json()) ?? {};
  }
```

### Usage of the service methods

The methods return a `Promise<?>` therefore the caller must define the callback when the promise returns

```typescript
 this.housingService.getAllHousingLocations().then((housingLocationList: HousingLocation[]) => {
      this.housingLocationList = housingLocationList;
      this.filteredLocationList = housingLocationList;
    });
// Here we cannot define the loadedHousingLocation type because it can be HousingLocation | undefined
this.housingService.getHousingLocationById(this.housingLocationId)
    .then((loadedHousingLocation) => {this.housingLocation = loadedHousingLocation});
```



https://angular.io/tutorial/tour-of-heroes/toh-pt6
https://medium.com/front-end-weekly/handling-http-request-using-rxjs-in-angular-25060a70c6d9
https://timdeschryver.dev/blog/the-benefits-of-adding-rx-query-to-your-angular-project#update-methods

https://angular.dev/tutorials/first-app/search
https://blog.angular-university.io/how-to-build-angular2-apps-using-rxjs-observable-data-services-pitfalls-to-avoid/
https://github.com/apedano/angular-f1-app/blob/main/src/app/shared/generic.service.ts