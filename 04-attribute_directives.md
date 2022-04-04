# Exercise: Attribute Directives

In this exercise we want to build our first `Attribute Directive`.

# Advanced way

implement the attribute directive `TiltDirective`. The directive should `rotate` its host element (hint: `ElementRef`)
when _entering_ it with the mouse and reset the rotation when the mouse _leaves_ the host element.

In addition to a simple rotation, the directive should rotate the element according to the position
the cursor entered the element.
If the cursor enters from **left** => rotate to the **right** and vice versa.

start by using the `ElementRef#nativeElement#addEventListener` methods and continue by implementing it
with the help of `@HostListener` and `@HostBinding` for the respective events and bindings.

## helper for advanced way

```bash
ng g directive tilt
```

```ts

transform = 'rotate()';

this.elementRef.nativeElement.addEventListener('event', callbackFn);

@HostListener('event', ['$event'])

@HostListener('prop.val')

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

# Step by Step

## implement tilt directive

now we want to add some funk to our component by letting it animate when we enter it with the mouse.

generate the directive

```bash
ng generate directive tilt

OR

ng g d tilt
```

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

<details>
    <summary>show result</summary>

```ts

@Directive({
    selector: '[tilt]'
})
export class TiltDirective implements OnInit {
    
    constructor(private el: ElementRef) {}
    
    ngOnInit() {}
    
}
```

</details>

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

apply the `tilt` directive to the `movie-card.component.html` template.

It should be applied to the `div.movie-card`.

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

initialize the HostListeners and HostBindings needed to replace our manual eventListeners and move the code accordingly.

```ts

@HostListener('mouseenter', ['$event'])
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
