# 03-Http Interceptors 

https://angular.io/guide/http-intercept-requests-and-responses
With interception, you declare interceptors that **inspect and transform HTTP requests from or to your application to a server**.  
Multiple interceptors form a **forward-and-backward chain** of request/response handlers.

Interceptors can perform a variety of implicit tasks, 
    - **authentication** to **logging**, in a routine, standard way, for every HTTP request/response.

## Write an interceptor

```typescript
@Injectable()
export class LoggingHttpInterceptor implements HttpInterceptor {

    constructor(private appStateService: AppStateService) {}

    intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
        this.appStateService.logIfDebug("Intercepted HTTP request", req);
        return next.handle(req); //triggers the next interceptor in the chain
    }
}
```
The `intercept` method transforms a request into an `Observable` that eventually returns the HTTP response. 
In this sense, **each interceptor is fully capable of handling the request entirely by itself**.

### The ``next`` object

```typescript
export abstract class HttpHandler {
  abstract handle(req: HttpRequest<any>): Observable<HttpEvent<any>>;
}
```

The next object represents the **next interceptor in the chain of interceptors**. 
The final next in the chain is the **HttpClient backend handler** that sends the request to the server and receives the server's response.

Most interceptors call `next.handle()` so that the request flows through to the next interceptor and, eventually, the backend handler. 
An interceptor could **skip calling `next.handle()`, short-circuit the chain**, and return its own `Observable` with an artificial server response.

## Provide the interceptor

The interceptor will be injected by Angular's DI as eny other service, for that we
need a provider. The declaration order will be the one used for intercepting requests;
**responses will pass intercptors in reverse order**.

```typescript
/** Array of Http interceptor providers in outside-in order */
export const httpInterceptorProviders = [ 
  { provide: HTTP_INTERCEPTORS, useClass: LoggingHttpInterceptor, multi: true }
  //other interceptors called in order for requests
];
```

![interceptor_stack.png](images%2Finterceptor_stack.png)

The left arrows represent the request handling.

### Add the provider in a standalone application

The last step is to add the provider in the ``main.ts`` file

```typescript
bootstrapApplication(AppComponent, {
  providers: [
      ...
    provideHttpClient(withInterceptorsFromDi()),
    httpInterceptorProviders,
  ]
}).catch((err) => console.error(err));
```

## Handle interceptor events

https://angular.io/guide/http-intercept-requests-and-responses#handle-interceptor-events

Request and response passed to the interceptor are immutable (`read-only`) so we need to clone 
the instances if the intercptors have to change parts of them, by using `clone()`.

```typescript
// clone request and replace 'http://' with 'https://' at the same time
const secureReq = req.clone({
  url: req.url.replace('http://', 'https://')
});
// send the cloned, "secure" request to the next handler.
return next.handle(secureReq);
```

```typescript
// copy the body and trim whitespace from the name property
const newBody = { ...body, name: body.name.trim() };
// clone request and set its body
const newReq = req.clone({ body: newBody });
// send the cloned request to the next handler.
return next.handle(newReq);
```




