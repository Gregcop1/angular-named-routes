# ng-named-routes
Service to set and get named routes for angular 4+ router

One day Angular team had the idea to remove the name parameter of their routes. They had certainly good reason, ... I'm pretty sure they had... but in some case those reasons don't apply.
This service is the easiest way to reuse them. It allow you to name some routes and use those names in component or template instead of the full path you usually copy/paste everywhere in your app (what a joy when the client ask you to change paths ;))

## Installation

for npm user:
```
npm install ng-named-routes
```

for yarn user:
```
yarn add ng-named-routes
```

## Configuration

Give some names to your routes by adding an extra property `name` in their declaration:
```
export const appRoutes: any[] = [
  { path: 'login', component: LoginComponent, name: 'login' },
  {
    path: '',
    children: [
      { path: 'users', name: 'users.list', children: [
        { path: 'create', component: UserCreateComponent, name: 'user.create', canActivate: [AuthGuard] },
        { path: 'edit/:id', component: UserEditComponent, name: 'user.edit', canActivate: [AuthGuard] },
        { path: '', component: UserListComponent, canActivate: [AuthGuard] },
      ]},
      { path: '', component: SidebarComponent, outlet: 'aside', canActivate: [AuthGuard] },
    ],
  },
];
```
As you can see, you can apply name on each level of routes.

> Note: this `name` property is not compatible with `Routes` type, that's why I declared my var with a `any[]` type, but it's not a problem cause this array is still compatible with route's declaration (`RouterModule.forRoot(appRoutes)`).

Ok, our routes have names so ng-named-routes can enter in action if you declare it in your app module.

```
// app.module.ts

@NgModule({
  declarations: [ ... ],
  imports: [ ...
    RouterModule.forRoot(appRoutes),
  ],
  providers: [
    NamedRoutesService,
  ],
  ...
})
export class AppModule { }
```

Now our service is ready to serve but still needs one more step. Feed him with your routes:

```
// app.module.ts
@NgModule({
  declarations: [ ... ],
  imports: [ ... ],
  providers: [
    NamedRoutesService,
  ],
  ...
})
export class AppModule {
  constructor(router: Router, namedRoutesService: NamedRoutesService) {
    namedRoutesService.setRoutes(router.config);
  }
}
```

That's it for config part, now let's see how to use it in our app.

## Usage without parameter

### In a controller

If you want to use a named route in your component, declare the service with dependency injection and get the route full path by calling the `getRoute` function with a name you declared earlier as parameter.

```
@Component({...})
export class DashboardComponent {
  constructor(private namedRoutesService: NamedRoutesService, private router: Router) { }

  public goToList() {
    this.router.navigate([this.namedRoutesService.getRoute('users.list')]);
  }
}
```

> Note: Does it work with nested routes? Of course! when you fed the service, the class has build a mapping of named and full paths auto-magically found.

### In template

If you want to use a route in a template you can expose the `getRoute` function of the service to your template.

```
@Component({...})
export class DashboardComponent {
  public getRoute: (...any) => string;

  constructor(namedRoutesService: NamedRoutesService) {
    this.getRoute = namedRoutesService.getRoute.bind(namedRoutesService);
  }
}
```

And then in the template:

```
<a [routerLink]="getRoute('user.list')">User list</a>
```

> Note: don't forget to rewrite context of the `getRoute` function by using `bind` otherwise you'll encounter some problem to find your routes.

### Route with parameters

Your route needs parameters? It's not a problem. `getRoute` function accept an object as second parameter where you can set dynamic parts of your route

```
<a [routerLink]="[getRoute('user.edit', {id: 42})]">{{user.name}}</a>
```

will redirect you to: "users/edit/42"

## Q/A

<details>
    <summary>Can I use it with nested routes?</summary>
    Of course, you can. When service register name, it calculate the full path of routes and bind it to the name.
</details>
<details>
    <summary>Can I use it with routes which have parameters?</summary>
    Of course, you can. Feed the "getRoute" function with your parameters and it will return you a formated route.
</details>

## I've found a bug

I will not agitate my hand before your eyes, saying that's not the feature you are looking for. Ok! Shits happen.

Feel free to [create an issue][issues] and if you have some time maybe you can propose a Pull Request. :)

[issues]:https://github.com/Gregcop1/ng-named-routes/issues
