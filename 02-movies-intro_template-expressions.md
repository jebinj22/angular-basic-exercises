# Exercise 2: angular movies intro & template expressions

In this Exercise, we want to install the angular movies workshop project, get to know the codebase a bit
and also create our first template expressions in the `AppComponent`.

## install angular movies project

```bash
git clone git@github.com:push-based/angular-basic-040422.git

# or if not work

git clone https://github.com/push-based/angular-basic-040422.git

cd angular-basic-040422

npm install
```

## serve application

```bash
ng serve -o
```

## implement movie-card template

go to `src/projects/movies/src/app.component.html`

implement movie-card with static html

```html
 <div class="movie-card">
    <img class="movie-image" src="https://image.tmdb.org/t/p/w300//qsdjk9oAKSQMWs0Vt5Pyfh6O4GZ.jpg">
    <div class="movie-card-content">
        <div class="movie-card-title">
            Turning Red
        </div>
        <div class="movie-card-rating">
            5
        </div>
    </div>
</div>
```

## apply stylings

Now it's time to make the movie-card a bit more beautiful

go to `src/projects/movies/src/app.component.scss`

```scss
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

## bring template to live

go to `src/projects/movies/src/app.component.ts`

```ts
// app.component.ts

// movie object for data binding
movie = {
    title: 'Turning Red',
    poster_path: '/qsdjk9oAKSQMWs0Vt5Pyfh6O4GZ.jpg',
    vote_average: 5
}

// event binding function
onMovieClick(event) {
    console.log(event, 'movie clicked');
}
```

go to `src/projects/movies/src/app.component.html`

```html
<!-- app.component.html -->

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

serve the application, see if the card gets displayed properly and the console output gets shown in your
devtools


