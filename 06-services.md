# Exercise: Services, Dependency Injection, Interceptors, wrap-up

In this exercise we will get to know the dependency injection system of angular.
We will introduce a new `MovieService` which for now will serve as our abstraction layer for HttpAccess.

Furthermore, we will implement our first `HttpIntercepter` to finally get rid of the `Bearer {token}` in our http calls.

To wrap it all up, we will introduce the `MovieDetailPageComponent`.

## introduce MovieService

create a new service `MovieService` in the `movie` folder. It should be providedIn `root`.

<details>
    <summary>show solution</summary>

`ng g s movie/movie`

you should end up having the following `MovieService`

```ts
@Injectable({
  providedIn: 'root'
})
export class MovieService {

  constructor() { }
}
```

</details>

Inject, the `HttpService`.

Now start introducing methods for re-usage in our components:
* method to fetch movies by `getMovies(category: string)` (return value is same as the current `movies$` in `MovieListPageComponent`)
* method to fetch movie by `getMovieById(id: string)`
* method to fetch recommendations `getMovieRecommendations(id: string)`

Information for the byId request:
* [`/movie/${movieId}`](https://developers.themoviedb.org/3/movies/get-movie-details)
* returns `TMDBMovieDetailsModel` (`shared/model/movie-details.model.ts`)

Information for the recommendations request:
* [`/movie/${movieId}/recommendations`](https://developers.themoviedb.org/3/movies/get-movie-recommendations)
* returns `{ results: MovieModel[] }` (`shared/model/movie-details.model.ts`)
  


<details>
    <summary>show solution</summary>

```ts
// movie.service.ts

getMovieRecommendations(id: string): Observable<{ results: MovieModel[] }> {
    return this.httpClient.get<TMDBMovieDetailsModel>(
        `${tmdbBaseUrl}/3/movie/${id}/recommendations`,
        {
            headers: {
                Authorization: `Bearer ${tmdbApiReadAccessKey}`,
            },
        }
    );
}

getMovieById(id: string): Observable<TMDBMovieDetailsModel> {
    return this.httpClient.get<TMDBMovieDetailsModel>(
        `${tmdbBaseUrl}/3/movie/${id}`,
        {
            headers: {
                Authorization: `Bearer ${tmdbApiReadAccessKey}`,
            },
        }
    );
}

getMovies(category: string): Observable<{ results: MovieModel[] }> {
    return this.httpClient.get<{ results: MovieModel[]}>(
        `${tmdbBaseUrl}/3/movie/${category}`,
        {
            headers: {
                Authorization: `Bearer ${tmdbApiReadAccessKey}`,
            },
        }
    );
}
```
</details>

Now we want to make use of our `MovieService` in the `MovieListPageComponent`.

<details>
    <summary>show solution</summary>

Go to the `MovieListPageComponent`, inject the `MovieService` and replace it with the `HttpClient`

```ts
// movie-list-page.component.ts

constructor(
    private movieService: MovieService
) {
}

// onInit
this.activatedRoute.params.subscribe((params) => {
    this.movies$ = this.movieService.getMovies(params.category);
});
```

</details>

serve the application, the movie list should still be visible

## introduce TMDBReadAccessInterceptor

generate a new `ReadAccessInterceptor`.

The `Interceptor` should add `Authorization: `Bearer ${environment.tmdbApiReadAccessKey}`` token statement
automatically on each request.

If you don't read the solution, you may want to [read here](https://angular.io/guide/http#intercepting-requests-and-responses).

<details>
    <summary>show solution</summary>

`ng g interceptor read-access`

```ts
// read-access.interceptor.ts

intercept(request: HttpRequest<unknown>, next: HttpHandler): Observable<HttpEvent<unknown>> {
    return next.handle(
        request.clone({
            setHeaders: {
                Authorization: `Bearer ${environment.tmdbApiReadAccessKey}`
            },
        })
    );
}
```

</details>

After finishing the implementation, we need to provide the `ReadAccessInterceptor` to our application.

create a [`StaticProvider`](https://angular.io/api/core/StaticProvider) which provides the `ReadAccessInterceptor`
as `HTTP_INTERCEPTORS`.

<details>
    <summary>show solution</summary>

provide the `ReadAccessInterceptor` as `HTTP_INTERCEPTORS` in the `AppModule`

```ts
// app.module.ts
providers: [
    {
        provide: HTTP_INTERCEPTORS,
        useClass: ReadAccessInterceptor,
        multi: true
    }
]
```
</details>

Now that our interceptor is in place, we can remove the `header` configuration from our http call in the `MovieService`.

<details>
    <summary>show solution</summary>

```ts
// movie.service.ts
// do the same for the other request
return this.httpClient.get<{ results: MovieModel[]}>(
    `${tmdbBaseUrl}/3/movie/${category}`
);
```

</details>

serve the application and check in the network tab if the header is still set and the result is still a valid movie list response

### Bonus: ErrorInterceptor

Implement an interceptor which listens to the response of a request instead.
In case of an error, it should at least `console.error` a message.
If you like, you can also send an `alert` to the user.

When u are done implementing it, try to produce an error. You can do it either with `rxjs`, or you can think about a way
dealing with it with the `devtools` :-)

useful resources:
* [blog post](https://dev.to/this-is-angular/angular-error-interceptor-12bg)
* [docs](https://angular.io/guide/http#intercepting-requests-and-responses)
* [alert](https://developer.mozilla.org/de/docs/Web/API/Window/alert)
* [throwError](https://rxjs.dev/api/index/function/throwError)
* [catchError](https://rxjs.dev/api/operators/catchError)

## implement MovieDetailPage

It's time to wrap it all up, the detail page will let us implement and recap the following features:
* lazy loaded routing
* router params
* structural directives
* http client / service usage
* multiple http calls
* loading state
* routing

### Set things up

generate `MovieDetailPageComponent` and `RoutedModule` for `MovieDetailPage`.

<details>
    <summary> show solution </summary>

```bash
ng g m movie/movie-detail-page

ng g c movie/movie-detail-page
```

```ts
// movie-detail-page.module.ts

const routes: Routes = [{
    path: '',
    component: MovieDetailPageComponent
}];

RouterModule.forChild(routes)

```

</details>

Now we can configure our lazy-loaded route in the `AppRoutingModule`.
The route should have a parameter for the `:id`. 

If you don't stick to the solution, you can assign any custom route you want, it doesn't matter. Just be sure to add
a parameter for the id :)

<details>
    <summary> show solution </summary>

```ts
// app-routing.module.ts

{
    path: 'movie/:id',
        loadChildren: () => import('./movie/movie-detail-page/movie-detail-page.module')
    .then(m => m.MovieDetailPageModule)
},

```

</details>


### Implement MovieDetailPageComponent

Now it's time to implement our actual `MovieDetailPageComponent`.

The pattern is very similar to the one we use `MovieListPageComponent`.
We need to make sure to use the `ActivatedRoute` in order to `subscribe` to the `params`.

With the `id` from our params we now can make the request to the `MovieDetail` and the `MovieRecommendations` endpoints.

make sure to provide two `Observable` values for the template:
* `movie$: Observable<TMBD...>`
* `recommendations$: Observable<{ results: MovieModel[] }>`

<details>
    <summary> show solution </summary>

```ts
// movie-detail-page.component.ts

movie$: Observable<TMDBMovieDetailsModel>;
recommendations$: Observable<{ results: MovieModel[] }>;

constructor(
    private movieService: MovieService,
    private activatedRoute: ActivatedRoute
) { }

ngOnInit(): void {
    this.activatedRoute.params.subscribe(params => {
        if (params.id) {
            this.movie$ = this.movieService.getMovie(params.id);
            this.recommendations$ = this.movieService.getMovieRecommendations(params.id);
        }
    })
}
```

</details>

Now we should be ready to go to implement our template.
