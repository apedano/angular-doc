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



