# Exercise 6: Services, Dependency Injection & Interceptors

## introduce MovieService

create a new service `MovieService` in the `movie` folder

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

Now start introducing methods for re-usage in our components for fetching a list of movies and a single movie by id

```ts
// movie.service.ts

const { tmdbBaseUrl, tmdbApiReadAccessKey } = environment;

getMovies(): Observable<{ results: MovieModel[] }> {
    // destruct variables
    return this.httpClient.get<{ results: MovieModel[]}>(
        `${tmdbBaseUrl}/3/movie/popular`,
        {
            headers: {
                Authorization: `Bearer ${tmdbApiReadAccessKey}`,
            },
        }
    );
}

```

Go to the `MovieListPageComponent`, inject the `MovieService` and replace it with the `HttpClient`

```ts
// movie-list-page.component.ts

movies$ = this.movieService.getMovies();

constructor(
    private movieService: MovieService
) {
}
```

serve the application, the movie list should still be visible

## introduce TMDBReadAccessInterceptor

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

Now that our interceptor is in place, we can remove the `header` configuration from our http call in the `MovieService`

```ts
// movie.service.ts

getMovies(): Observable<{ results: MovieModel[] }> {
    // destruct variables
    return this.httpClient.get<{ results: MovieModel[]}>(
        `${tmdbBaseUrl}/3/movie/popular`
    );
}

```


serve the application and check in the network tab if the header is still set and the result is still a valid movie list response
