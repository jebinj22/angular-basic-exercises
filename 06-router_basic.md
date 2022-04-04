# Exercise 6: Router setup

## Create movie-list-page component

```bash
ng g c movie/movie-list-page --module=movie
```

Move data-fetching from `AppComponent` to `MovieListPageComponent`

```ts
// movie-list-page.component.ts

movies$: Observable<{ results: MovieModel[]}>;

constructor(
    private httpClient: HttpClient
) { }

ngOnInit() {
    // destruct variables
    const { tmdbBaseUrl, tmdbApiReadAccessKey } = environment;
    this.movies$ = this.httpClient.get<{ results: MovieModel[]}>(
        `${tmdbBaseUrl}/3/movie/popular`,
        {
            headers: {
                Authorization: `Bearer ${tmdbApiReadAccessKey}`,
            },
        }
    );
}

```

```html
<!-- movie-list-page.component.html -->

<movie-list
  [movies]="movieResponse.results"
  *ngIf="(movies$ | async) as movieResponse; else: loading"></movie-list>
<ng-template #loading>
  <div class="loader"></div>
</ng-template>

```

## Setup routing for movie-page components

Create a const for the routes and add the import for the `RouterModule` to the `import` section
of the `MovieModule`

```ts
// movie.module.ts

const routes: Routes = [
    {
        path: 'list/popular',
        component: MovieListPageComponent
    }
];

imports: [
    CommonModule,
    RouterModule.forChild(routes)
]
```

Now add the `router-outlet` to the `AppComponent`'s template. you can also remove any typescript code from the `AppComponent`

```html
<!-- app.component.html -->

<app-shell>
    <router-outlet></router-outlet>
</app-shell>
```

Serve the application. you should see no output in the content section of the `AppComponent`. 
Change the url in the address bar to `http://localhost:4200/list/popular`. You should see movie-list-page being rendered now.

## Router Configuration

We want to make sure users don't land on an empty page when navigating to our application. Lets configure a default route by
configuring a `redirectTo` rule for our router in the `AppModule`

```ts
// app.module.ts

RouterModule.forRoot([
    {
        path: '',
        redirectTo: '/list/popular',
        pathMatch: 'full'
    }
])

```

Serve the application and navigate to `/`. You should get redirected to the list/popular route now.

## 404 page

Please try to enter any invalid route into the address-bar of your browser (e.g. `list/populardawdaw`), you will see the application navigates back to the default
route and throw an error in the console

`Error: Uncaught (in promise): Error: Cannot match any routes. URL Segment: 'list/populardawdaw'`

Let's build a 404 page in case of a user entering an invalid url

`ng g c not-found`

```html
<!-- not-found.component.html -->

<div class="not-found-container">
    <svg-icon size="350px" name="error"></svg-icon>
    <h1 class="title">Sorry, page not found</h1>
</div>
```

```scss
/* not-found.component.scss */

:host {
  width: 100%;
  height: 100%;
  display: block;
}

.not-found-container {
  display: flex;
  align-items: center;
  justify-content: center;
  flex-direction: column;
}

.title {
  text-align: center;
  font-size: 4rem;
  font-weight: 700;
  margin: 3rem 1rem;
}
```


With our new component in place, we can configure a wildcard fallback to display it on any invalid url

```ts
// app.module.ts

RouterModule.forRoot([
    // other configuration
    {
        path: '**',
        component: NotFoundComponent
    }
])
```

Serve the application and again try enter an invalid url. the application now should display the 404 page component instead

## `routerLink`

Now we have a beautiful 404 page showing the user he did something wrong, but there is yet no chance for him to recover to
a proper state of the application without touching the address bar.  
We want to give the user the possibility to navigate back to the popular list when he is on a wrong path.

Go to `NotFoundComponent` and insert a button with a `routerLink` Directive to setup routing to the popular list.

```html
<!-- not-found.component.html -->

<div class="not-found-container">
    <!-- .... -->
    <a class="btn" routerLink="/list/popular">See popular</a>
</div>

```

Serve the application, enter an invalid url and hit see if the navigation works

## Bonus

### Router Tracing

If you ever encounter issues related to routing, you can use the `enableTracing` configuration param to get verbose debugging information
about what the router does during runtime.

```ts
// app.module.ts

RouterModule.forRoot([
    // route configuration
], {
    enableTracing: true
})
```

Serve the application and start different routing processes while observing the console output in your devtools

### `useHash`

You can also try out different routing styles, e.g. by adding the `useHash` configuration param to your router config

```ts
// app.module.ts

RouterModule.forRoot([
    // route configuration
], {
    useHash: true
})
```

Serve the application and observe the different route styles in the address bar while navigating around the application
