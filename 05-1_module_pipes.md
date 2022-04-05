# Exercise: NgModule and Pipes

In this exercise we want to implement our first `Pipe` as well as finally introduce a simple `NgModule` architecture.

`MovieModule` will be our first `DomainModule`

`TiltModule` will be our first `SCAM`


in the end we will have the following **folder structure**

```
src/
  app/
    movie/
      movie-card/
        movie-card.component
      movie-image.pipe.ts
      movie.module.ts
      
    tilt/
      tilt.directive.ts
      tilt.module.ts
```

# Advanced way

implement the pipe `MovieImagePipe`. The pipe should take a `string` input variable.  
It returns a concatenated string of `https://image.tmdb.org/t/p/w300` + the input value.  
If the input value is empty, it should return a placeholder url instead.

After you've finished the implementation of the `MovieImagePipe` and made sure it's working, continue by introducing
the `MovieModule` and `TiltModule`.


Create a new `NgModule`, `MovieModule`. It should `declare` the following components, directives, ...
* `MovieImagePipe`
* `MovieCardComponent`

It should `export` the following components:
* `MovieCardComponent`

Create a new `NgModule`, `TiltModule`. It should `declare` and `export` the `TiltDirective`.

make sure to move all involved components, pipes, directives into their correct folders.

## helper for advanced way

Placeholder image: `assets/images/no_poster_available.jpg`  

```html
<img [src]="movie.poster_path | movieImage">
```

```bash
ng g p movie-image

ng g m movie

ng g m tilt
```

# Step by Step

## implement movie-image pipe

create a `Pipe` to deliver the formatted image url for our movie-card component

It returns a concatenated string of `https://image.tmdb.org/t/p/w300` + the input value.  
If the input value is empty, it should return a placeholder url instead.

```bash
ng generate pipe movie-image

OR

ng g p movie-image
```

<details>
    <summary>show result</summary>

```ts
@Pipe({
    name: 'movieImage'
})
export class MovieImagePipe implements PipeTransform {
    // we keep the args for now, we may need them later
    transform(value: string, ...args: unknown[]): string {
        if (value) {
            return `https://image.tmdb.org/t/p/w300${value}`;
        }
        return `assets/images/no_poster_available.jpg`;
    }
}
```
</details>

## use pipe in movie-card

```html
<!-- movie-card.component.html -->

<img class="movie-image" [src]="movie.poster_path | movieImage">
```

try the pipe by setting the `poster_path` property in the `AppComponent` to an empty string `''`

The poster should now be displaying the fallback image

## introduce MovieModule

create the `DomainModule`, `MovieModule`.

```bash
ng generate module movie

OR

ng g m movie
```

move the following components to the `MovieModule` by adding them to the `declarations` and optionally to `exports`
property of the `MovieModule`.
Also move their files into the new `movie` folder

* `MovieCardComponent`
* `MovieImagePipe`

<details>
    <summary>show result</summary>

```ts
// movie/movie.module.ts

@NgModule({
    imports: [CommonModule],
    exports: [MovieCardComponent],
    declarations: [MovieCardComponent, MovieImagePipe]
})
export class MovieModule {}
```

</details>

## introduce TiltModule

now we want to create our first `SCAM`.

```bash
ng generate module tilt 

# or 

ng g m tilt 
```

move the `TiltDirective` to the `TiltModule` by adding them to the `declarations` and `exports`
property of the `TiltModule`.
Also move their files into the new `tilt/` folder

<details>
    <summary>show result</summary>

```ts
// tilt/tilt.module.ts

@NgModule({
    imports: [],
    exports: [TiltDirective],
    declarations: [TiltDirective]
})
export class TiltModule {}
```

</details>
you probably need to adjust some imports in the `MovieModule` as well as in `AppModule` now 

serve the application and see if the refactoring went well!
