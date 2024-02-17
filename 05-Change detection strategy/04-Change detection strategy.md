# 04-Change detection strategy

JavaScript, by default, uses **mutable data structures** that you can reference from multiple different components. 

Angular runs **change detection** over your entire component tree to **make sure that the most up-to-date state of your data structures is reflected in the DOM**.

Change detection is sufficiently fast for most applications. However, when an application has an **especially large component tree**, 

running change detection across the whole application **can cause performance issues**. 
You can address this by configuring **change detection to only run on a subset of the component tree**.

If you are **confident that a part of the application is not affected by a state change**, you can use `OnPush` 
to skip change detection in an entire component subtree.

## The default strategies

Angular relyses on zone.js which detects a change in the document tree (either an input binding
or an event handling), but **it doesn't identify where it happens** in the tree.

Therefore, the default strategy is to run change detection **from top to bottom for all components in the tree**, as seen below.

![default_strategy.png](img%2Fdefault_strategy.png)

## Using `OnPush`

`OnPush` change detection instructs Angular to run change detection for a **component subtree** `only` when:

1. The **root component of the subtree receives new inputs** as the result of a template binding. Angular compares the current and past value of the input with ==.
1. Angular **handles an event** (for example using event binding, output binding, or @HostListener ) **in the subtree's root component or any of its children** whether they are using `OnPush` change detection or not.

```typescript
import { ChangeDetectionStrategy, Component } from '@angular/core';
@Component({
  changeDetection: ChangeDetectionStrategy.OnPush,
})
export class MyComponent {}
```


## Common change detection scenarios

### Legenda

![legenda_01.png](img%2Flegenda_01.png)
Component tree node using the Default strategy (left) and the OnPush strategy (right)

![legenda_02.png](img%2Flegenda_02.png)

Component tree node dispatching an event (left), running change detection (middle), receiving an input (right)

### onPush receives a new input

With an input binding value change,  

![scnario_1.png](img%2Fscnario_1.png)

Change detection will run on all components except E, because:

* A, B, and D use the Default strategy

* C uses `OnPush`, but the rule (1) applies (it received a new input)

* E uses `OnPush`, but it didn’t receive any new inputs, hence it’s not checked

* F is part of the subtree with C as its root and uses the Default strategy, hence it is checked

### Event on an `onPush`

![scnario_2.png](img%2Fscnario_2.png)

Change detection will run on **all components except E**, because:

* A, B, and D use the Default strategy

* C uses OnPush, but rule (2) applies (the event is in its scope as root)

* E uses OnPush, but the event is out of its scope, hence it’s not checked

* F is part of the subtree with C as its root and uses the Default strategy, hence it is checked

### Event on child of `onPush`

![scnario_3.png](img%2Fscnario_3.png)

Change detection will run on all components, because:

* A, C, E, and F use the Default strategy

* B uses the `OnPush` strategy, but the event happened on D which is its child, hence rule (2) applies

* D uses the `OnPush` strategy, but the rule (2) applies (the event is in its scope)


### Event on a default strategy component

![scnario_4.png](img%2Fscnario_4.png)

Change detection will run only on A, B, and D, because:

* A, B, and D use the Default strategy

C uses `OnPush` and none of the rules holds (event outside its scope), hence the entire subtree is skipped

## Summary 

Keep in mind of following details of OnPush change detection, will make your app working transparently as default, but more efficient!

1. With `Onpush`, Angular only compare change of variable by reference. and only `@Input` changed and passed from parent, will auto trigger change detection.

2. Any event binding using `(foo)=”bar()”` or `@HostListener()` will trigger change detection automatically

3. Async pipe will trigger change detection automatically, so try to use more async pipe, and avoid manully do the subscription in component, otherwise a `changeDetectorRef.detectChanges()` is required.