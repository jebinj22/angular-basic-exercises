# Exercise: structural directives

In this exercise we meet our first `structural directives`.
We want to display a loading state when there is no data available as well as be able to
display a list of movies instead of a single card.

# Advanced

## Simple Loading State

start by getting to know the `*ngIf` syntax in the `AppComponent`.

try to hide the `movie-card` when there is no data and instead (`;else:`) show another template.

```html
<!-- hint: use can use ng-template or ngIf twice :) -->
<div class="loader"></div>
```

## Movie List

now introduce the new `MovieListComponent` in the `MovieModule`.

It should receive a `Movie[]` as input and display it as list in its template.

```ts
@Input() movies: Movie[]
```

Iterate over the `Movie[]` in the template with `*ngFor`

```html
<!-- movie-list.component.html -->

<div class="movie-list">

    <!-- iterate here over MovieCardComponent -->
    
</div>
```

Also, introduce proper stylings for the `MovieListComponent`. You find the styles in the helper section, or you 
can introduce your very own styling if you like :)


Finally, use the `MovieListComponent` in the `AppComponent`s template instead of the old `movie-card`.

For this you probably need to mock `Movie[]` data. Feel free to just duplicate the one movie that's already there
and just some values accordingly.

## Helper

[structural directives](https://angular.io/guide/structural-directives#structural-directive-shorthand)

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

# Step by Step

## *ngIf to display missing-movie state

display a loading state when the movie is missing.

loader template:

```html
<ng-template #loading>
    <div class="loader"></div>
</ng-template>
```

now use `*ngIf` on the `movie-card` to display it only when `movie` is truthy

```html
<movie-card
        *ngIf="movie"
        (movieClicked)="onMovieClick($event)"
        [movie]="movie"></movie-card>
```

Finally, apply the `loader template` as `else` for the `*ngIf`

unset the `movie` property, try if it works

## show movie grid

go to `projects/movies/src/app/app.component.ts`

prepare array of Movies `MovieModel[]`

<details>
    <summary>show result</summary>

```ts
// app.component.ts

movies: MovieModel[] = [
    {
        title: 'Turning Red',
        poster_path: '/qsdjk9oAKSQMWs0Vt5Pyfh6O4GZ.jpg',
        vote_average: 5
    } // duplicate if you like :)
]

```

</details>

we now want to display a list of `movie-cards`.

start by adding `div.movie-list` and move the `*ngIf` logic to it

```html
<div class="movie-list" *ngIf="movies?.length; else...."></div>
```

Now introduce `*ngFor` to iterate over the `movies` array and display a list
of `movie-card`s.

```html
<movie-card *ngFor="let movie of movies"></movie-card>
```

<details>
    <summary>Final result</summary>

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
</details>

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

## create MovieListComponent

```bash
ng g c movie/movie-list --module=movie
```

* setup inputs
* move methods from `app.component.ts` to react to click events

```ts
// movie-list.component.ts

@Input() movies: MovieModel[];

onMovieClick(event) {
    console.log(event, 'movie clicked');
}
```


move list-template from `app.component.html` to `movie-list.component.html`

```html
<!-- movie-list.component.html -->

<!-- please don't move the *ngIf in here :) -->
<div class="movie-list">
    <!-- movie-card iteration -->
</div>

```

move list styles from `app.component.scss` to `movie-list.component.scss`


