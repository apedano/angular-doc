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


https://angular.dev/tutorials/first-app/ngFor