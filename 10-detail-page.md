# Exercise 10: Movie Detail Page

It's time to wrap it all up, the detail page will let us implement and recap the following features:
* lazy loaded routing
* router params
* structural directives
* http client / service usage
* multiple http calls
* loading state
* routing

## Implement Movie Detail Page

create module with basic lazy-loading router setup

`ng g m movie/movie-detail-page`

```ts
// movie-detail-page.module.ts

const routes: Routes = [{
    path: '',
    component: MovieDetailPageComponent
}];

RouterModule.forChild(routes)

```

create `MovieDetailPageComponent`

`ng g c movie/movie-detail-page`

```ts
// movie-detail-page.component.ts

```

start adding static values needed for our template

```ts
// movie-detail-page.component.ts

```

go to the template and implement with static values

```html
<!-- movie-detail-page.component.html -->

```

also add stylings

```scss
/* movie-detail-page.component.scss */


```

serve the application and navigate to `/movie` by typing it into the address bar

## Add dynamic routing

```ts
// app.module.ts

{
    path: 'movie/:id',
        loadChildren: () => import('./movie/movie-detail-page/movie-detail-page.module')
    .then(m => m.MovieDetailPageModule)
},

```

```ts
// movie-detail-page.component.ts

movie$: Observable<TMDBMovieDetailsModel>;

constructor(
    private movieService: MovieService,
    private activatedRoute: ActivatedRoute
) { }

ngOnInit(): void {
    this.activatedRoute.params.subscribe(params => {
        if (params.id) {
            this.movie$ = this.movieService.getMovie(params.id);
        }
    })
}
```

go to `MovieListComponent` and finally make use of the click binding we introduced in the beginning

```ts
// movie-list.component.ts

constructor(private router: Router) {}

onMovieClick(movie: MovieModel) {
    this.router.navigate(['/movie', movie.id]);
}

```

we also need to adjust the click binding slightly. We want to pass the `MovieModel` object instead of the `$event` variable.

```html

<movie-card
    *ngFor="let movie of movies"
    (movieClicked)="onMovieClick(movie)"
    [movie]="movie"></movie-card>
```

serve the application, try to navigate to the `MovieDetailPageComponent` when clicking on a movie card

### Bonus: use routerLink instead of click binding

implement the `routerLink` directive to navigate to the `MovieDetailPageComponent` instead of using the click binding 

## use data in template

```html
<!-- movie-detail-page.component.html -->
*ngIf="movie$ | async as movie;"

```

## loading state

```html
<!-- movie-detail-page.component.html -->

*ngIf="movie$ | async as movie; else: loader"

<ng-template #loader>
    <div class="loader"></div>
</ng-template>

```

## Back button, imdb link

> TBD

## recommended movies data

> TBD 

implement recommended movies:

* add method to `MovieService`
* use in `MovieDetailsPageComponent`
* display in template

## cast data

> TBD

implement cast list:

* add method to `MovieService`
* use in `MovieDetailsPageComponent`
* display in template
