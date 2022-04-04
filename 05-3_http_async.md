# Exercise: HttpClient & async pipe

In this exercise we get to know the `HttpClient`, set up our first get request in order to fetch a list of `Movies` we
can properly proceed in our view.

By throttling our network we can also check if our loading spinner is actually doing its job or not :).

## setup modules

import `HttpClientModule` into `AppModule` and inject `HttpClient` into `AppComponent`s constructor

<details>
    <summary>show result</summary>

```ts
// app.module.ts

@NgModule({
    imports: [HttpClientModule]
})
export class AppModule {}
```

```ts
// app.component.ts

export class AppComponent {

    constructor(
        private httpClient: HttpClient
    ) {}
}
```
</details>

## first http network request

use the `HttpClient` in the `ngOnInit` lifecycle hook in order to send our first get request.

the url will be `/3/movie/popular` + `environment.tmbdBaseUrl`

```ts
this.httpClient.get(url, options);
```

We also need to configure headers in order to communicate with our API

```ts
// tmbdApiReadAccessKey comes from environment as well

headers: {
    Authorization: `Bearer ${tmdbApiReadAccessKey}`
}
```

use the headers as options for network request

now `subscribe` to the request and log the output

```ts
this.httpClient.get(url, options)
    .subscribe(console.log);
```

<details>
    <summary>full solution</summary>

```ts
// app.component.ts

constructor(
    private httpClient: HttpClient
) {
}

ngOnInit() {
    // destruct environment variables
    const { tmdbBaseUrl, tmdbApiReadAccessKey } = environment;
    this.httpClient.get(
        `${tmdbBaseUrl}/3/movie/popular`,
        {
            headers: {
                Authorization: `Bearer ${tmdbApiReadAccessKey}`,
            },
        }
    ).subscribe(console.log);
}
```
</details>

run the application and see if you get a result :)

## don't forget to unsubscribe!!!

store the `subscription` into a private field in the `AppComponent`

```ts
private readonly sub = new Subscription();
```

add the `subscription` to the local `sub` when you set it up

```ts
ngOnInit() {
    
    this.sub.add(
        this.http.get()...
    )
}
```

implement the `OnDestroy` interface and make sure to `unsubscribe` from the `subscription`

```ts
implements OnDestroy
```

```ts

ngOnDestroy() {
    this.sub.unsubscribe();
}
```


## display real values

store data in local variable and use in template

instead of calling `console.log` in the `subscribe` method we want to actually set our `movie: Movie[]` field to
the new value coming as response from the request.

if you have already inspected the outcome of the request, you may have noticed the result is something like:

```ts
{ results: MovieModel[], /* other properties */ }
```

let's type our get request and properly react to the subscription value

```ts
this.httpClient.get<{ results: MovieModel[]}>()
    .subscribe(response => {
        this.movies = response.results;
    })

// or with object destructing

this.httpClient.get<{ results: MovieModel[]}>()
    .subscribe(({ results }) => {
        this.movies = results;
    })
```


<details>
    <summary>full solution</summary>

```ts
// app.component.ts

constructor(
    private httpClient: HttpClient
) {}

ngOnInit() {
    // destruct environment variables
    const { tmdbBaseUrl, tmdbApiReadAccessKey } = environment;
    this.httpClient.get<{ results: MovieModel[]}>(
        `${tmdbBaseUrl}/3/movie/popular`,
        {
            headers: {
                Authorization: `Bearer ${tmdbApiReadAccessKey}`,
            },
        }
    ).subscribe(response => {
        this.movies = response.results;
    });
}
```
</details>

serve the application and see the result! You should now see a beautiful list of movies.

## throttle and watch loading spinner

serve the application

go to the network tab of your dev tools and configure network throttling to something very slow (slow 3g).

just refresh the page and see if the loading spinner appears for a while until the result is finally there.

## set up Observable movie$ field

head to `AppComponent` and prepare a variable `movie$` of type `Observable<{ results: MovieModel[] }>`

we want to assign the http request to that variable for later usage in the template

```ts
this.movies$ = this.httpClient.get()...
```

## use async pipe

get rid of the subscription in the component and instead subscribe in the template
by using the built-in `AsyncPipe`.

also remove the local `movies` field from `AppComponent`. The application should not build any longer.

in the template we now use the `| async` pipe in order to retrieve the values from the `movies$` observable.

replace the usages of `movie` to `movies$` and use the `async` pipe instead

<details>
    <summary>show solution</summary>

```html
<!-- movie-list.component.html -->

[movies]="(movies$ | async).results"
*ngIf="(movies$ | async); else: loading"
```

</details>

Run the application and make sure it's showing the movie list.

> ⚠️ warning ⚠️ with this setup in place, we are sending two network requests instead of only one

Open the network tab in your `devtools` and watch out for the network request to the `popular` endpoint. You
will notice it fires twice.

To avoid this, we can make use of the `async ngIf hack`.

```html
*ngIf="(movies$ | async) as movieResponse
```

<details>
    <summary>show solution</summary>

```html
<!-- app.component.html -->

<movie-list
        [movies]="movieResponse.results"
        *ngIf="(movies$ | async) as movieResponse; else: loading"></movie-list>
<ng-template #loading>
    <div class="loader"></div>
</ng-template>
```

</details>

run the application and check if the second http request is gone
