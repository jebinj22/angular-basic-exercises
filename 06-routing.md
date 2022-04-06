# Exercise 6: Router setup

We know will get to know the basics of angulars routing system.
The goal of this exercise is to enable routing for the three main categories
of our movies app:
* popular
* top_rated
* upcoming

For this we will introduce a new `MoviePageComponent` which will handle the routing
and the display of the respective movies for us.

We will continue creating our first `RoutingModule` to configure our Router.

In the end we will have accomplished the following goals:

* default route to `list/popular`
* not found page (with `NotFoundPageComponent`)
* enable the user to navigate our page with `routerLink`
* have a dedicated component to handle routing for our movie categories

## Create movie-list-page component

introduce a new `MovieListPageComponent` as `SCAM`.

the `MovieListPageModule` does not need to export the `MovieListPageComponent`, we plan to
only configure a route to it.

<details>
    <summary>generate component and module</summary>

```bash
# generate module
ng g m movie/movie-list-page

# generate component
ng g c movie/movie-list-page
```

</details>

The `MovieListPageComponent` in its first state should just do what the `AppComponent`
did before.

Please move (just the code, don't delete any component pls!!) everything movie-list
related including data-fetching from `AppComponent` to `MovieListPageComponent`.

At this point in time you can also remove the `MovieListComponent` from the `exports`
section in the `NgModule` so that it has no exports left whatsoever.

Also remove the import to `MovieModule` in `AppModule` and replace it with `MovieListPageModule` instead.

<details>
    <summary>full solution</summary>

```ts
// movie-list-page.module.ts
import { NgModule } from '@angular/core';

@NgModule({
    declarations: [
        MovieListPageComponent,
    ],
    imports: [MovieModule],
    exports: [],
})
export class MovieListPageModule {}
```

```ts
// movie-list-page.component.ts
import { NgModule } from '@angular/core';
import { Component, OnInit } from '@angular/core';
import { HttpClient } from '@angular/common/http';

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

</details>

## Setup AppRouting module

now that we have a component dedicated for our router to work with, all thats left is to configure
the router.

create an `AppRoutingModule` next to `AppModule` (`--flat`)

<details>
    <summary>show solution</summary>

```bash
# flat means that we don't want to create a dedicated folder for it

ng g m app-routing --flat
```
</details>

Create a `const routes: Routes` and configure two routes.

* `list/popular` to `MovieListPageComponent`
* `''` redirecting to `list/popular`

in the `AppRoutingModule` you need to import the `RouterModule.forRoot()` modules and pass the `routes`
const as arguments to it.

It is very convenient to add the `RouterModule` not only to the `imports` section, but as well to the `exports`.

Now add the `router-outlet` to the `AppComponent`'s template.

You can also remove any typescript code from the `AppComponent`, it's just an empty class now!

Serve the application. The router should navigate you automatically to `list/popular` if you try to navigate to `/`.
You should see movie-list-page being rendered now.

An invalid route though should end up in an error, we will fix that later.

<details>
    <summary>full solution</summary>


```ts
// app-routing.module.ts
import { RouterModule, Routes } from '@angular/router';

const routes: Routes = [
    {
        path: 'list/popular',
        component: MovieListPageComponent
    },
    {
        path: '',
        redirectTo: '/list/popular',
        pathMatch: 'full',
    },
];

imports: [
    RouterModule.forRoot(routes)
],
exports: [RouterModule] // for convenience only
```

```html
<!-- app.component.html -->

<app-shell>
    <router-outlet></router-outlet>
</app-shell>
```

```ts
// app.module.ts

imports: [AppRoutingModule]

```

</details>

## 404 page

Please try to enter any invalid route into the address-bar of your browser (e.g. `list/populardawdaw`), you will see the application navigates back to the default
route and throw an error in the console

`Error: Uncaught (in promise): Error: Cannot match any routes. URL Segment: 'list/populardawdaw'`

Let's build a **beautiful** 404 page in case of a user entering an invalid url

Create a new `NotFoundPageComponent`. In best case create it as `SCAM` !!

If you create it as `SCAM`, you don't need to export it, though. We only plan to configure a
route for it.

It does not need any typescript logic whatsoever but just should have a template showing the
user that this page is invalid and giving him a link back to a valid site.

<details>
    <summary>styles and template for not-found</summary>

```html
<!-- not-found-page.component.html -->

<div class="not-found-container">
    <svg-icon size="350px" name="error"></svg-icon>
    <h1 class="title">Sorry, page not found</h1>
    <a class="btn" routerLink="/list/popular">See popular</a>
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
</details>

<details>
    <summary>full solution</summary>

```bash
ng g m not-found-page

ng g c not-found-page
```

```ts
import { NgModule } from '@angular/core';

@NgModule({
  declarations: [NotFoundPageComponent],
  imports: [RouterModule, SvgIconModule],
  exports: [],
})
export class NotFoundPageModule {}
```

```ts
// app-routing.module.ts
import { RouterModule, Routes } from '@angular/router';

RouterModule.forRoot([
    // other configuration
    {
        path: '**',
        component: NotFoundPageComponent
    }
])
```

important: we need to import the module in the app.module for now

```ts
// app.module.ts

imports: [
    /** other imports **/,
    NotFoundPageModule
]
```

</details>

Serve the application and again try enter an invalid url. the application now should display the 404 page component instead

## Setup router params

we want to make our list route more dynamic to switch between different categories of movies we want to display

configure the `list/popular` route in `AppRoutingModule` to take a `category` parameter instead.

<details>
    <summary>show solution</summary>

```ts
// app-routing.module.ts
{
    path: 'list/:category',
    component: MovieListPageComponent
}
```

</details>

Now we need to react to the router params in the `MovieListPageComponent`.
Please inject the `ActivatedRoute` into the constructor of your component.

Now we are able to `subscribe` to the `params` Observable exposed by `ActivatedRoute`.

Use `params.category` in order to dynamically generate a url to fetch the movies.

<details>
    <summary>show solution</summary>

```ts
// movie-list-page.component.ts
import { ActivatedRoute } from '@angular/router';

this.activatedRoute.params.subscribe((params) => {
    
    this.movies$ = this.httpClient.get<{ results: MovieModel[] }>(
        `${environment.tmdbBaseUrl}/3/movie/${params.category}`,
        {
            headers: {
                Authorization: `Bearer ${environment.tmdbApiReadAccessKey}`,
            },
        }
    );
    
});

```

</details>

### Bonus

Please implement a validation for the params. If the category is something else
than `popular | upcoming | top_rated` navigate the user to the `NotFoundPage` or some other valid url.

In the most clean way you implement it as a `CanActivateGuard`.

`ng g guard ....`

Please study the return values of the `canActivate` method and think about different solutions
to navigate the user to a correct url :-)

## enable user navigation in the sidebar

In this step we finally want to implement the user based navigation in the sidebar.
For this we need to touch the `AppShellComponent` the first time, since it controls the
contents of the sidebar.

Since the categories are static, we don't need to use `*ngFor` here!

<details>
    <summary>show solution</summary>

Add the following snippet in to contents of `<nav class="navigation">` in `app-shell.component.html`

Insert the missing pieces for upcoming and top_rated in your own!

```html

<h3 class="navigation--headline">Discover</h3>
<a
    class="navigation--link"
    [routerLink]="['???', '???']"
    routerLinkActive="active"
  >
    <div class="navigation--menu-item">
      <svg-icon class="navigation--menu-item-icon" name="popular"></svg-icon>
      Popular
    </div>
</a>

<!-- insert the missing categories top_rated and upcoming -->

```

</details>

Serve the application and check out your new routing system!!! 

### Bonus (if you are really bored, please don't waste time with it otherwise)

make an array out of the categories in the `AppShellComponent` and use `*ngFor` instead of three different
templates

## Enable LazyLoading for all routes

Luckily we were smart enough to implement `SCAM` modules, now we will have a much easier time
to introduce `lazyloading` ;-)

We want to lazyload our `MovieListPageComponent` as well as the `NotFoundPageComponent`.

For this we need to transform our `SCAM` into a `RoutedModule`.

Please add a `const routes: Routes` to both lazyloaded modules.
The configuration should point the path `''` to the respective component you want to lazyload.

<details>
    <summary>show solution</summary>

```ts
// not-found-page.module.ts
import { RouterModule, Routes } from '@angular/router';

const routes: Routes = [
    {
        path: '',
        component: NotFoundPageComponent
    }
]
```

```ts
// movie-list-page.module.ts
import { RouterModule, Routes } from '@angular/router';

const routes: Routes = [
    {
        path: '',
        component: MovieListPageComponent
    }
]
```

</details>

Now that the feature modules are configured for proper lazyloading, we now can add the missing bits to the `AppModule`s
root configuration.

```ts
// replace existing routes for notfound page and movie-list with this setup

{
    path: 'path/',
    loadChildren: () => import('path/to/module').then(m => m.MyModuleToLazyLoad)
},

```

> Don't forget to remove the imports to `MovieListPageModule` and `NotFoundPageComponent` in the `AppModule`

<details>
    <summary>show solution</summary>

```ts
// app-routing.module.ts

const routes: Routes = [
    {
        path: 'list/:category',
        loadChildren: () => {
            return import('./movie/movie-list-page/movie-list-page.module').then(
                (m) => m.MovieListPageModule
            );
        },
    },
    {
        path: '',
        redirectTo: 'list/popular',
        pathMatch: 'full',
    },
    {
        path: '**',
        loadChildren: () => {
            return import('./not-found-page/not-found-page.module').then(
                (m) => m.NotFoundPageModule
            );
        },
    },
];

@NgModule({
    declarations: [],
    imports: [RouterModule.forRoot(routes)],
    exports: [RouterModule],
})
export class AppRoutingModule {}

```

</details>

serve the application, you should notice that the bundler now produces two new bundles which resemble the lazyloaded pieces of code
we have configured with our new router configuration.

```shell
Lazy Chunk Files                                                                 | Names         |      Size
projects_movies_src_app_movie_movie-list-page_movie-list-page_module_ts.js       | -             |   6.95 kB
projects_movies_src_app_not-found_not-found_module_ts.js                         | -             |   6.06 kB
```

## Super Bonus (this will be hard, like really ^^)

Implement a `MovieGenrePageComponent`, it should be accessible under the route `genre/:id`.

The component itself should be quite similar to `MovieListPageComponent`, but uses a different 
endpoint and router params for getting data.

you will need to send the request to `/3/discover/movie` and add `with_genres` as query parameter.

Please see the [`movie-discover docs`](https://developers.themoviedb.org/3/discover/movie-discover) for more information.

As a precondition for this, you will need to show the genre navigation list.
The template for it is already in place, so you just have to comment it in, it lives in the
`AppShellComponent`.

The missing piece is the `genre$: Observable`.
The genre list will be fetched from the tmdb api as well.

You will need to set up an HTTP call in the `AppShellComponent` for it.

Consider using the `TMDBMovieGenreModel` interface for typings which lives in `shared/model/...`

To fetch the genres, use the endpoint [`/3/genre/movie/list`](https://developers.themoviedb.org/3/genres/get-movie-list).

If you have the observable in place, you should now get a list of genres as a navigation list.
You maybe need to adjust the `routerLink` directives parameter to point to the route you have configured for
`MovieGenrePageComponent`.





