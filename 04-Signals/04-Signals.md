# 04-Signals

https://angular.dev/guide/signals
Angular Signals is a system that granularly tracks how and where your state is used throughout an application, 
allowing the framework to optimize rendering updates.

A signal is a **wrapper around a value that notifies interested consumers when that value changes**. 
Signals can contain any value, from simple primitives to complex data structures.

You read a signal's value by calling its getter function, which allows Angular to track where the signal is used.

Signals may be either writable or **read-only**.

## Writable signals

Writable signals, of type `WritableSignal`, provide an **API for updating their values directly**. 
You create writable signals by calling the `signal` function with the signal's initial value:

```typescript
const count: WritableSignal = signal(0);
// Signals are getter functions - calling them reads their value.
console.log('The count is: ' + count());
```

To change the value of a writable signal, either `.set()` it directly:

```typescript
count.set(3);
```

or use the `.update()` operation to compute a new value from the previous one:

```typescript
// Increment the count by 1.
count.update(value => value + 1);

```

## Computed signals

Computed signal are **read-only signals that derive their value from other signals**. 
You define computed signals using the computed function and specifying a derivation:

```typescript
const count: WritableSignal<number> = signal(0);
const doubleCount: Signal<number> = computed(() => count() * 2);
```

The `doubleCount` signal depends on the `count` signal. 
Whenever `count` updates, Angular knows that `doubleCount` needs to update as well.
If we try to set the value of a computed signal, Angular will generate a **compilation error**.

### Computed signals are both lazily evaluated and memoized

`doubleCount`'s derivation function does not run to calculate its value until the first time you read `doubleCount`. The calculated value is then cached, and if you read doubleCount again, it will return the cached value without recalculating.

If you then change `count`, Angular knows that `doubleCount`'s cached value is no longer valid, and the **next time you read** `doubleCount` **its new value will be calculated**.

As a result, you can safely perform computationally expensive derivations in computed signals, such as filtering arrays.

### Computed signal dependencies are dynamic

**Only the signals actually read during the derivation are tracked**. 
For example, in this computed the count `signal` is only read if the `showCount` signal is true:

```typescript
const showCount = signal(false);
const count = signal(0);
const conditionalCount = computed(() => {
    if (showCount()) {
        return `The count is ${count()}.`;
    } else {
        return 'Nothing to see here!';
    }
});
```

When you read `conditionalCount`, if `showCount` is false the `Nothing to see here!` message is returned **without reading the `count` signal**. 
This means that if you later update count it will not result in a recomputation of conditionalCount.

If you set `showCount` to true and then read `conditionalCount` again, the **derivation will re-execute** and take the branch where `showCount` is true, returning the message which shows the value of `count`. 
Changing `count` will then invalidate `conditionalCount`'s cached value.

Note that dependencies can be removed during a derivation as well as added. 
If you later set `showCount` back to `false`, then `count` **will no longer be considered a dependency of `conditionalCount`**.

## Reading signals in `OnPush` components

When you read a signal within an `OnPush` component's template, Angular tracks the signal as a dependency of that component. 
When the value of that signal changes, Angular automatically marks the component to ensure it gets updated the next time change detection runs. 
Refer to the Skipping component subtrees guide for more information about OnPush components.



https://blog.angular-university.io/angular-signals/
https://medium.com/ngconf/keeping-state-with-a-service-using-signals-bee652158ecf

