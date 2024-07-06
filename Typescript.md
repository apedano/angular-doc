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

### Strict and non strict operator

The **equality operator** only compares the values of the operands, 
while the **strict equality operator** compares both the values and types of the operands. 

```typescript
let x = 10;
let y = 5;
let a = "apple";
let b = "banana";

console.log(x == y); // false
console.log(x == 10); // true
console.log(x === y); // false (same type but different value)
console.log(x === "10"); // false (different data types)
console.log(x === 10); // true (same data types and value)
console.log(a == b); // false
console.log(a == "apple"); // true
console.log(a === b); // false (same type but different value)
console.log(a === "apple"); // true (same data types and value)
console.log(a === 10); // false (different data types and value)
```


```typescript
let x: string | null = null;
let y: string | undefined = undefined;

//strict check
if (x !== null) {
    console.log(x.toUpperCase());
}
if (y !== undefined) {
    console.log(y.toUpperCase());
}

// non strict
if (x != null) {
    console.log(x.toUpperCase());
}
if (y != undefined) {
    console.log(y.toUpperCase());
}

```


### Elvis operator

Call the property on the right of `?` if the property on the left of `?` is not `null`

```typescript
f.form.controls.email?.valid
```


## Arrays

### Declaration 

```typescript
public recipeIngredients!: RecipeIngredient[] ;
```

### Add to array

```typescript
var numbers = new Array(1, 4, 9); 
var length = numbers.push(10); 
console.log("new numbers is : " + numbers );  
length = numbers.push(20); 
console.log("new numbers is : " + numbers );
```

`
new numbers is : 1,4,9,10
new numbers is : 1,4,9,10,20
`

### Remove elements

```typescript
let array: number[] = [0, 1, 2, 3, 4, 5, 6];

//Remove from the end
let removedElement = array.pop();  //[0, 1, 2, 3, 4, 5]

//Remove from the beginning
removedElement = array.shift();  //[1, 2, 3, 4]

//Remove from specified index
let index = array.indexOf(1);
let elementsToRemove = 2;
let removedElementsArray = array.splice(index, elementsToRemove);  //[1, 2]

//Remove from specified index
let itemIndex = array.indexOf(3);
let newArray = array.filter((e, i) => i !== itemIndex);  //[4, 5]

//Delete the value at specified index without resizing the array
delete array[1];   //[3, , 5]

```

