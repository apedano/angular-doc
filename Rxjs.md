# Rxjs

## Pipe and map an

```typescript
protected fetchAll(): Observable<T[]> {
    return this.httpClient.get<{ [key: string]: T }>(this.getApiFullUrl() + '.json')
        .pipe(
            map((originalResponseData: { [key: string]: any }) => {
                // console.log('originalResponseData from fethAll call', originalResponseData);
                const valuesArray: T[] = [];
                for (const idKey in originalResponseData) {
                    console.log('response data T', originalResponseData[idKey], "for Idkey", idKey);
                    valuesArray.push(this.mapToEntity(idKey, originalResponseData[idKey]))
                }
                return valuesArray;
            }),
            catchError(errorRes => {
                //error handling code goes here
                //here we return the observable sending the error 
                //to the subscribers
                return throwError(() => errorRes);
            })
        );
}
```

The `httpClient.get()` function returns `Observable<{ [key: string]: any }>`, the `pipe` 
operator transforms the type returned by the http call into `T[]`, implementing error handling
with the `catchError` function. 

### Subscribing

A potential subscriber could be: 

```typescript
 fetchAllSubscription!: Subscription;
    private fetchAllObserver: Partial<Observer<T[]>> = { //Partial means that non all properties of the object will be set
        next: allValues => {
            this.allValuesSubject.next(allValues);
            this.isAllValueSubjectCalledAlready = true;
        },
        error: err => {
            const errorInstance = <HttpErrorResponse>err;
            console.log(errorInstance);
        }
    };


    protected refreshVallValues() {
        this.fetchAllSubscription?.unsubscribe();
        this.fetchAllSubscription = this.fetchAll().subscribe(this.fetchAllObserver);
    }
```
With ``Partial``

```typescript
/**
 * Make all properties in T optional
 */
type Partial<T> = {
    [P in keyof T]?: T[P];
};

```

### Error handling

```typescript
public getById(id: string): Observable<T> {
        return this.getByFilter((t: T) => t.id === id)
            .pipe(
                map(array => {
                    if(array.length == 0 || !array) {
                        //this returns an Observable which unsubscribes when the error is thrown
                        // return throwError(() => new IdEntityNotFoundError(`No element found with ID=${id}`, id)); 
                        throw new IdEntityNotFoundError(`No element found with ID=${id}`, id);
                    }
                    return array[0]
                    }
                ),
            );
    }
```

A possible error from the called observable is just propagated.

```typescript
this.recipeService.getById(id).subscribe(
  {
    next: entity => {
      this.recipe = entity;
      this.onEntityLoaded();
    },
    error: (err: IdEntityNotFoundError) => {
      console.error(err);
      this.hasError = true;
      this.loaded = true;
      this.error = err;
    }
  }
);
```

