# Exercise 5: structural directives, NgModule & HttpClient

## *ngIf to display missing-movie state

display a loading state when the movie is missing

```html
<!-- app.component.html -->
<movie-card
        *ngIf="movie; else: loading"
        (movieClicked)="onMovieClick($event)"
        [movie]="movie"></movie-card>

<ng-template #loading>
    <div class="loader"></div>
</ng-template>
```

unset the `movie` property, try if it works

## show movie grid

go to `projects/movies/src/app/app.component.ts`

prepare array of Movies `MovieModel[]`

```ts
// app.component.ts

movies: MovieModel[] = [
    {
        title: 'Turning Red',
        poster_path: '/qsdjk9oAKSQMWs0Vt5Pyfh6O4GZ.jpg',
        vote_average: 5
    }
]

```

go to `projects/movies/src/app/app.component.html`

use `*ngFor` to display a list of movies

```html
<!-- app.component.html -->

<div class="movie-list" *ngIf="movies?.length; else: loading">
    
    <movie-card
            *ngFor="let movie of movies"
            (movieClicked)="onMovieClick($event)"
            [movie]="movie"></movie-card>
    
</div>
<ng-template #loading>
    
    <div class="loader"></div>
    
</ng-template>

```

apply stylings

go to `projects/movies/src/app/app.component.scss`

```scss
/* app.component.scss */

.movie-list {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(10rem, 35rem));
  gap: 4rem 2rem;
  place-content: space-between space-evenly;
  align-items: start;
  margin: 4rem 0;
  position: relative;
}

```

## refactor to NgModule

create `MovieModule`

```bash
ng generate module movie

OR

ng g m movie
```

move the following components to the `MovieModule` by adding them to the `declarations` and `exports`
property of the `MovieModule`.
Also move their files into the new `movie` folder

* `MovieCardComponent`
* `MovieImagePipe`

> note that our *Directive is not working any longer, please ignore it for now or move it as well to the movie module

```ts
// movie/movie.module.ts

@NgModule({
    imports: [],
    exports: [MovieCardComponent, MovieImagePipe],
    declarations: [MovieCardComponent, MovieImagePipe]
})
export class MovieModule {}
```

## create MovieListComponent

```bash
ng g c movie/movie-list --module=movie
```

go to `projects/movies/src/app/movie/movie-list/movie-list.component.ts`

* setup inputs
* move methods from `app.component.ts` to react to click events

```ts
// movie-list.component.ts

@Input() movies: MovieModel[];

onMovieClick(event) {
    console.log(event, 'movie clicked');
}
```


go to `projects/movies/src/app/movie/movie-list/movie-list.component.html`

move list-template from `app.component.html` to `movie-list.component.html

```html
<!-- movie-list.component.html -->

<div class="movie-list" *ngIf="movies?.length;">
    <movie-card
            *ngFor="let movie of movies"
            (movieClicked)="onMovieClick($event)"
            [movie]="movie"></movie-card>
</div>

```

apply stylings

go to `projects/movies/src/app/movie/movie-list/movie-list.component.scss`

move list styles from `app.component.scss` to `movie-list.component.scss

```scss
/* movie-list.component.scss */

.movie-list {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(10rem, 35rem));
  gap: 4rem 2rem;
  place-content: space-between space-evenly;
  align-items: start;
  margin: 4rem 0;
  position: relative;
}

```

## use actual data

import `HttpClientModule`

```ts
// app.module.ts

@NgModule({
    imports: [HttpClientModule]
})
export class AppModule {}
```

inject `HttpClient` in `AppComponent` in order to setup an http call

```ts
// app.component.ts

export class AppComponent {

    constructor(
        private httpClient: HttpClient
    ) {}
}
```

setup http call and log data

```ts
// app.component.ts

constructor(
    private httpClient: HttpClient
) {
}

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
    ).subscribe(console.log);
}
```

store data in local variable and use in template

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


## use async pipe

get rid of the subscription in the component and instead subscribe in the template
by using the built-in `AsyncPipe`.

head to `AppComponent` and prepare the variable `movie$` of type `Observable<{ results: MovieModel[] }>`

we want to assign the http request to that variable for later usage in the template

```ts
// app.component.ts

// observable values are typically postfixed with `$` sign
movies$: Observable<{ results: MovieModel [] }>;

constructor(
    private httpClient: HttpClient
) {
}

ngOnInit() {
    // destruct variables
    const { tmdbBaseUrl, tmdbApiReadAccessKey } = environment;
    // assign observable value here
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

in the template we now use the `async` pipe in order to retrieve the values from the `movies$` observable

```html
<!-- movie-list.component.html -->
<app-shell>
    <movie-list
            [movies]="(movies$ | async).results"
            *ngIf="(movies$ | async); else: loading"></movie-list>
    <ng-template #loading>
        <div class="loader"></div>
    </ng-template>
</app-shell>

```

Run the application and make sure it's showing the movie list.

> ⚠️ warning ⚠️ with this setup in place, we are sending two network requests instead of only one

Open the network tab in your `devtools` and watch out for the network request to the `popular` endpoint. You
will notice it fires twice.

To avoid this, we can make use of the `async ngIf hack`.

```html
<!-- app.component.html -->

<movie-list
        [movies]="movieResponse.results"
        *ngIf="(movies$ | async) as movieResponse; else: loading"></movie-list>
<ng-template #loading>
    <div class="loader"></div>
</ng-template>
```


run the application and check if the second http request is gone
