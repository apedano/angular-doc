# 06 - Pipes

## ``async``

```typescript
export class RecipesComponent {

  public recipes$!: Observable<Recipe[]>
  ...  
}
```

```angular2html
@for (recipe of recipes$ | async; track recipe) {
        <div>{{recipe.id}}</div>
        <div>{{recipe.name}}</div>
}
```

```angular2html
<ng-container *ngIf="recipes$ | async">
    <div>Recipes loaded</div>
</ng-container>
```