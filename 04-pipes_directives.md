# Exercise 4: Pipes and Directives

## implement movie-image pipe

create a `Pipe` to deliver the formatted image url for our movie-card component

```bash
ng generate pipe movie-image

OR

ng g p movie-image
```

go to `projects/movies/src/app/movie-image.pipe.ts`

```ts
@Pipe({
    selector: 'movieImage'
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

## use pipe in movie-card

go to `projects/movies/src/app/movie-card/movie-card.component.html`

```html
<!-- movie-card.component.html -->

<img class="movie-image" [src]="movie.poster_path | movieImage">
```

try the pipe by setting the `poster_path` property in the `AppComponent` to an empty string `''`

The poster should now be displaying the fallback image

## implement tilt directive

now we want to add some funk to our component by letting it animate when we enter it with the mouse.

generate the directive

```bash
ng generate directive tilt

OR

ng g d tilt
```

go to `projects/movies/src/app/tilt.directive`

```ts

@Directive({
    selector: '[tilt]'
})
export class TiltDirective {
    
    constructor() {}
}
```

implement the `OnInit` `Interface` and the `ngOninit` Lifecycle hook and set up a private variable
for the `ElementRef` in the constructor.

> Tip: type it with `HTMLElement`, you will have an easier life

```ts

@Directive({
    selector: '[tilt]'
})
export class TiltDirective implements OnInit {
    
    constructor(private el: ElementRef) {}
    
    ngOnInit() {}
    
}
```

setup the eventListeners


```ts

nativeElement.addEventListener('mouseleave', () => {
  // we want to reset the styles here
});

nativeElement.addEventListener('mouseenter', (event) => {
  // 
});
```


implement the reset and a simple animation


```ts

nativeElement.addEventListener('mouseleave', () => {
    nativeElement.style.transform = 'rotate(0deg)';
});

nativeElement.addEventListener('mouseenter', () => {
    nativeElement.style.transform = 'rotate(30deg)';
});
```


## use directive to adjust behavior of movie-card

go to `projects/movies/src/app/app.component.html`
or to `projects/movies/src/movie-card/movie-card.component.html`

```html
<!-- app.component.html -->
<movie-card tilt></movie-card>

<!-- or -->

<!-- movie-card.component.html -->
<div class="movie-card" tilt>
    <!-- template -->
</div>

```

serve the application and test your result

```bash
ng serve
```

## implement the funk :-D

now we want to add a more complex animation and tilt the movie-card according to the mouseposition on enter


```ts

  const pos = this.determineDirection(event.pageX);
  this.el.nativeElement.style.transform = `rotate(${pos === 0 ? '22deg' : '-22deg'})`;

  /**
   *
   * returns 0 if entered from left, 1 if entered from right
   */
  determineDirection(pos: number): 0 | 1 {
    const width = this.el.nativeElement.clientWidth;
    const middle = this.el.nativeElement.getBoundingClientRect().left + width / 2;
    return (pos > middle ? 1 : 0);
  }
```

serve the application and test your result

```bash
ng serve
```

## use HostBindings and HostListeners

now we want to refactor our directive to the `angular way`, yaay!

initialize the HostListeners and HostBindings needed for to replace our manual eventListeners and move the code accordingly.

```ts

@HostListener('mouseenter')
onMouseenter(event: MouseEvent) {
    
}

@HostListener('mouseleave')
onMouseleave() {
    
}

@HostBinding('style.transform')
rotation;
```


in the end, you can get rid of the `OnInit` implementation.

serve the application and test your result

```bash
ng serve
```
