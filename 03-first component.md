# Exercise 3: create your first component

In this exercise we want to create our first component (ofc leveraging the angular cli).
For this we will first transform our code we created before into a new component.
Afterwards we will make it a re-usable component by introducing `@Input` and `@Output` bindings.

## create component

```bash
ng generate component movie-card

OR

ng g c movie-card
```

move parts from `app.component.ts`, `app.component.html` & `app.component.scss`
to

`movie-card.component.ts`
`movie-card.component.html`
`movie-card.component.scss`

```html
<!-- movie-card.component.html -->

<div class="movie-card" (click)="onMovieClick($event)">
    <img class="movie-image"
         [src]="'https://image.tmdb.org/t/p/w300' + movie.poster_path">
    <div class="movie-card-content">
        <div class="movie-card-title">
            {{ movie.title }}
        </div>
        <div class="movie-card-rating">
            {{ movie.vote_average }}
        </div>
    </div>
</div>
```

```scss
/* movie-card.component.scss */

.movie-card {
  transition: transform .15s cubic-bezier(.4,0,.2,1) 0s;
  max-width: 300px;
}

.movie-card:hover {
  transform: scale(1.03);
}

.movie-image {
  display: block;
  width: 100%;
  height: auto;
}

.movie-card-content {
  text-align: center;
  padding: 1.5rem 3rem;
  font-size: 1.5rem;
}

.movie-card-title {
  font-size: 2rem;
}

```

```ts
// movie-card.component.ts

@Component({
  selector: 'movie-card',
  templateUrl: './movie-card.component.html',
  styleUrls: ['./movie-card.component.scss']
})
export class MovieCardComponent {

  movie;
  
  onMovieClick(event) {
      console.log(event);
  }
}

```

### Bonus:

try different schematics

```bash
ng g c movie-card --inline

ng g c movie-card --skipTests

```

## input bindings

introduce input binding for `movie`

```ts
// movie-card.component.ts

@Input() movie;

```

## output bindings

```ts
// movie-card.component.ts

@Output() movieClicked = new EventEmitter();

onMovieClick(event) {
    this.movieClicked.emit(event);
}

```

## use movie-card component

```html
<!-- app.component.html -->

<movie-card [movie]="movie"
            (movieClicked)="onMovieClick($event)">
</movie-card>
```

## introduce proper typings

introduce an interface `MovieModel` in `projects/movies/src/app/movie.model.ts`

```ts
// movie.model.ts

export interface MovieModel {
    id: string;
    title: string;
    poster_path: string;
    vote_average: number;
}
```

use the interface to type all movie related usages

```ts
// app.component.ts

movie: MovieModel = {
    // ...
}
```

```ts
// movie-card.component.ts

@Input() movie: MovieModel;
```

go to the `AppComponent`, remove the `poster_path` property and watch how the IDE notifies you about the error.
try to serve the application with `ng serve`.

restore the application to a working state afterwards


## make everything type safe

go to `AppComponent` & `MovieCardComponent` and type all event bindings with their correct types

```ts

@Output() movieClicked = new EventEmitter<MouseEvent>();

onMovieClick(event: MouseEvent) {
    
}
```
