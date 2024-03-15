# Tpypescript

## Interfaces

```typescript
export interface HousingLocation {
    id: number;
    name: string;
    city: string;
    state: string;
    photo: string;
    availableUnits: number;
    wifi: boolean;
    laundry: boolean;
  }
```

instance creation

```typescript
export class HomeComponent {

  readonly baseUrl = 'https://angular.dev/assets/tutorials/common';

  housingLocation: HousingLocation = {
    id: 9999,
    name: 'Test Home',
    city: 'Test city',
    state: 'ST',
    photo: `${this.baseUrl}/example-house.jpg`,
    availableUnits: 99,
    wifi: true,
    laundry: false
  }

}
```

## Assignments

## LET vs VAR

Variables declared by `let` are only available **inside the block where they’re defined**.
Variables declared by `var` are available **throughout the function in which they’re declared**.

```typescript
function varScoping() {
    var x = 1;
    if (true) {
        var x = 2;
        console.log(x); // will print 2
    }
    console.log(x); // will print 2
}

function letScoping() {
    let x = 1;
    if (true) {
        let x = 2;
        console.log(x); // will print 2
    }
    console.log(x); // will print 1
}
```

### Union types
A union type describes a value that can be one of several types.

```typescript
let something: number | string | boolean;
```


## Assertions

### Nullish coalescing operator

It can be used to assign a **default value to a variable if it is _null_ or _undefined_**

```typescript
const firstName = null;
const lastName = undefined;
const fullName = firstName ?? lastName ?? 'John Doe';
console.log(fullName); // Output: "John Doe"
```

```typescript

null || undefined ?? "foo"; // raises a SyntaxError
true && undefined ?? "foo"; // raises a SyntaxError

(null || undefined) ?? "foo"; // returns "foo"
```

### Nullish coalescing assignment

Nullish coalescing assignment short-circuits, meaning that `x ??= y` is equivalent to `x ?? (x = y)`, 
except that the expression x is only evaluated once.

```typescript
const x = 1;
x ??= 2; //does not throw an error, despite x being const

```

### Non null assertion

```typescript
housingLocation!: HousingLocation;
```

### Elvis operator

Call the property on the right of `?` if the property on the left of `?` is not `null`

```typescript
f.form.controls.email?.valid
```




