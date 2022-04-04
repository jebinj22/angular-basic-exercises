# Exercise 9: Auth State & Router Guards

## CanActivate AuthGuard

next up, we want to make use of `RouterGuards`.

Since we are using lazy-loaded routing, we want to introduce a `CanActivateChildGuard`.  
Let's start by introducing an `AuthGuard` which implements the `CanActivateChild` interface

`ng g guard auth/auth`

```shell
? Which interfaces would you like to implement? (Press <space> to select, <a> to toggle all, <i> to invert selection, and <enter> to proceed)
 ◯ CanActivate
❯◉ CanActivateChild
 ◯ CanDeactivate
 ◯ CanLoad
```

let's return false for the moment in order to test the behavior

```ts
// auth.guard.ts

canActivateChild(
    childRoute: ActivatedRouteSnapshot,
    state: RouterStateSnapshot): Observable<boolean | UrlTree> | Promise<boolean | UrlTree> | boolean | UrlTree {
    return false;
}
```

## Create protected route

To make use of the newly created `AuthGuard` let's create a new component which should only be accessible when a user is logged in.

implement a `MyMovieListModule`

`ng g m movie/my-movie-list`

make sure to integrate a router config, make it accessible as `/my-movies`

```ts
// my-movie-list.module.ts

const routes: Routes = [{
    path: '',
    component: MyMovieListComponent
}];

RouterModule.forChild(routes)

```

implement a `MyMovieListComponent`

`ng g c movie/my-movie-list`

when adding the route the `AppModule`s configuration, make sure to apply the `AuthGuard` to the `canActivateChild` property.


```ts
// app.module.ts

{
    path: 'my-movies',
        loadChildren: () => import('./movie/my-movie-list/my-movie-list.module')
    .then(m => m.MyMovieListModule),
    canActivateChild: [AuthGuard]
},

```

Serve the application and try to navigate to `/my-movies`. the guard should not let you do that.

```shell
ERROR Error: Uncaught (in promise): Error: Invalid CanActivateChild guard
```

## Auth Service

Now we want to add a bit more of a real world behavior, let's create an `AuthService` to handle our auth state

`ng g service auth/auth`

```ts
// auth.service.ts

isAuthenticated = false;

login() {
    this.isAuthenticated = true;
}

logout() {
    this.isAuthenticated = false;
}

```

We can now use the `AuthService` in the `AuthGuard` by simply injecting it

```ts
// auth.guard.ts

constructor(
    private authService: AuthService
) {}

canActivate(
    route: ActivatedRouteSnapshot,
    state: RouterStateSnapshot): Observable<boolean | UrlTree> | Promise<boolean | UrlTree> | boolean | UrlTree {
    return this.authService.isAuthenticated;
}

```

## Toggle Auth State

The last thing we need to do is to introduce a user interaction to toggle our faked auth service state.

go to `AppShellComponent` and implement an auth toggle

```ts
// app-shell.component.ts

get loggedIn(): boolean {
    return this.authService.isAuthenticated;
}

constructor(private authService: AuthService) {}

toggleAuth() {
    if (this.loggedIn) {
        this.authService.logout();
    } else {
        this.authservice.login();
    }
}
```

add the template to display a toggle button and a visualization of the auth state for the user

```html
<!-- app-shell.component.html -->

<button (click)="toggleAuth()">
    {{ loggedIn ? 'log in' : 'log out' }}
</button>
```

serve the application, test out the behavior of the implemented toggle button and see if you can navigate to the
desired route based on the auth state you triggered

## Add navigation item to MyMovieList

go to `AppShellComponent`s template and add a navigation item to navigate to `MyMovieList`

```html
<!-- app-shell.component.html -->

<h3 class="navigation--headline">Account</h3>
<a
        class="navigation--link"
        routerLink="/my-movies"
        routerLinkActive="active"
>
    <div class="navigation--menu-item">
        <svg-icon class="navigation--menu-item-icon" name="account"></svg-icon>
        My Movies
    </div>
</a>
```

### Bonus: Show item only when loggedIn

hide the navigation item when the user is not loggedIn

