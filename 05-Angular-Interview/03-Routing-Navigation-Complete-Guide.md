# Angular Routing & Navigation - Complete Interview Guide

## Table of Contents

1. [Router Fundamentals](#router-fundamentals)
2. [Route Configuration](#route-configuration)
3. [Navigation Methods](#navigation-methods)
4. [Route Parameters](#route-parameters)
5. [Child & Nested Routes](#child--nested-routes)
6. [Lazy Loading](#lazy-loading)
7. [Route Guards](#route-guards)
8. [Resolvers](#resolvers)
9. [Router Events](#router-events)
10. [Advanced Patterns](#advanced-patterns)
11. [Interview Questions & Answers](#interview-questions--answers)

---

## Router Fundamentals

### Angular Router Architecture

```
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚                      ANGULAR ROUTER ARCHITECTURE                          â”‚
â”œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¤
â”‚                                                                           â”‚
â”‚  URL                                                                      â”‚
â”‚   â”‚                                                                       â”‚
â”‚   â–¼                                                                       â”‚
â”‚  â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”                                                     â”‚
â”‚  â”‚  URL PARSING    â”‚  Parse URL into segments                            â”‚
â”‚  â””â”€â”€â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”€â”€â”˜                                                     â”‚
â”‚           â”‚                                                               â”‚
â”‚           â–¼                                                               â”‚
â”‚  â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”                                                     â”‚
â”‚  â”‚ ROUTE MATCHING  â”‚  Match URL to route configuration                   â”‚
â”‚  â””â”€â”€â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”€â”€â”˜                                                     â”‚
â”‚           â”‚                                                               â”‚
â”‚           â–¼                                                               â”‚
â”‚  â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”      â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”                           â”‚
â”‚  â”‚    GUARDS       â”‚â”€â”€â”€â”€â”€â–¶â”‚   Resolvers     â”‚                           â”‚
â”‚  â”‚ (Can activate?) â”‚      â”‚  (Pre-fetch)    â”‚                           â”‚
â”‚  â””â”€â”€â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”€â”€â”˜      â””â”€â”€â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”€â”€â”˜                           â”‚
â”‚           â”‚                        â”‚                                     â”‚
â”‚           â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜                                     â”‚
â”‚                        â–¼                                                  â”‚
â”‚  â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”       â”‚
â”‚  â”‚                    <router-outlet>                            â”‚       â”‚
â”‚  â”‚  â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”  â”‚       â”‚
â”‚  â”‚  â”‚              ACTIVATED COMPONENT                        â”‚  â”‚       â”‚
â”‚  â”‚  â”‚  â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”  â”‚  â”‚       â”‚
â”‚  â”‚  â”‚  â”‚         <router-outlet> (child)                  â”‚  â”‚  â”‚       â”‚
â”‚  â”‚  â”‚  â”‚  â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”  â”‚  â”‚  â”‚       â”‚
â”‚  â”‚  â”‚  â”‚  â”‚         CHILD COMPONENT                    â”‚  â”‚  â”‚  â”‚       â”‚
â”‚  â”‚  â”‚  â”‚  â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜  â”‚  â”‚  â”‚       â”‚
â”‚  â”‚  â”‚  â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜  â”‚  â”‚       â”‚
â”‚  â”‚  â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜  â”‚       â”‚
â”‚  â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜       â”‚
â”‚                                                                           â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

### Basic Setup

```typescript
// app.routes.ts (Standalone - Angular 14+)
import { Routes } from "@angular/router";

export const routes: Routes = [
  { path: "", component: HomeComponent },
  { path: "about", component: AboutComponent },
  { path: "users", component: UsersComponent },
  { path: "**", component: NotFoundComponent }, // Wildcard
];

// main.ts
import { bootstrapApplication } from "@angular/platform-browser";
import { provideRouter } from "@angular/router";
import { routes } from "./app/app.routes";

bootstrapApplication(AppComponent, {
  providers: [provideRouter(routes)],
});

// app.component.ts
@Component({
  selector: "app-root",
  standalone: true,
  imports: [RouterOutlet, RouterLink, RouterLinkActive],
  template: `
    <nav>
      <a
        routerLink="/"
        routerLinkActive="active"
        [routerLinkActiveOptions]="{ exact: true }"
        >Home</a
      >
      <a routerLink="/about" routerLinkActive="active">About</a>
      <a routerLink="/users" routerLinkActive="active">Users</a>
    </nav>

    <router-outlet />
  `,
})
export class AppComponent {}
```

### NgModule Setup (Traditional)

```typescript
// app-routing.module.ts
@NgModule({
  imports: [
    RouterModule.forRoot(routes, {
      scrollPositionRestoration: "enabled",
      anchorScrolling: "enabled",
      preloadingStrategy: PreloadAllModules,
    }),
  ],
  exports: [RouterModule],
})
export class AppRoutingModule {}

// app.module.ts
@NgModule({
  imports: [BrowserModule, AppRoutingModule],
  declarations: [AppComponent],
  bootstrap: [AppComponent],
})
export class AppModule {}
```

---

## Route Configuration

### Route Properties

```typescript
const routes: Routes = [
  {
    path: "users", // URL path
    component: UsersComponent, // Component to render
    title: "User List", // Page title (Angular 14+)
    data: { role: "admin" }, // Static data
    pathMatch: "full", // Matching strategy
    redirectTo: "/home", // Redirect target
    canActivate: [AuthGuard], // Guards
    canDeactivate: [UnsavedGuard],
    canActivateChild: [RoleGuard],
    canMatch: [FeatureGuard], // Angular 14.1+
    resolve: { user: UserResolver }, // Resolvers
    loadComponent: () => import("./user.component"), // Lazy component
    loadChildren: () => import("./user.routes"), // Lazy routes
    children: [
      // Child routes
      { path: ":id", component: UserDetailComponent },
    ],
    outlet: "sidebar", // Named outlet
    providers: [UserService], // Route-scoped providers
    runGuardsAndResolvers: "always", // When to re-run
  },
];
```

### Path Matching Strategies

```typescript
const routes: Routes = [
  // 'prefix' (default) - matches if URL starts with path
  { path: "users", component: UsersComponent }, // Matches /users, /users/123

  // 'full' - entire URL must match
  { path: "", component: HomeComponent, pathMatch: "full" }, // Only /

  // Required for redirects from empty path
  { path: "", redirectTo: "/home", pathMatch: "full" },
];
```

### Redirects

```typescript
const routes: Routes = [
  // Simple redirect
  { path: "", redirectTo: "/dashboard", pathMatch: "full" },

  // Redirect with preserving query params
  {
    path: "old-path",
    redirectTo: "/new-path",
    pathMatch: "prefix", // /old-path/sub â†’ /new-path/sub
  },

  // Redirect to external (use guard)
  {
    path: "external",
    canActivate: [
      () => {
        window.location.href = "https://external.com";
        return false;
      },
    ],
    component: EmptyComponent,
  },
];
```

### Wildcard Route (404)

```typescript
const routes: Routes = [
  { path: "", component: HomeComponent },
  { path: "users", component: UsersComponent },

  // MUST be last - catches all unmatched routes
  { path: "**", component: NotFoundComponent },
];
```

---

## Navigation Methods

### RouterLink Directive

```html
<!-- Basic navigation -->
<a routerLink="/users">Users</a>

<!-- With parameters -->
<a [routerLink]="['/users', userId]">View User</a>
<!-- Generates: /users/123 -->

<!-- With query params -->
<a [routerLink]="['/users']" [queryParams]="{ page: 1, sort: 'name' }">
  Users
</a>
<!-- Generates: /users?page=1&sort=name -->

<!-- Preserve query params -->
<a [routerLink]="['/users', userId]" queryParamsHandling="preserve">
  View User
</a>
<!-- Options: 'merge', 'preserve', '' (default - remove) -->

<!-- With fragment -->
<a [routerLink]="['/help']" fragment="section2">Help Section 2</a>
<!-- Generates: /help#section2 -->

<!-- Relative navigation -->
<a routerLink="./details">Details</a>
<!-- Relative to current -->
<a routerLink="../edit">Edit</a>
<!-- Up one level -->

<!-- RouterLinkActive -->
<a
  routerLink="/users"
  routerLinkActive="active highlighted"
  [routerLinkActiveOptions]="{ exact: true }"
>
  Users
</a>

<!-- Multiple classes -->
<a routerLink="/dashboard" [routerLinkActive]="['active', 'current']">
  Dashboard
</a>

<!-- Check if active in template -->
<a routerLink="/users" routerLinkActive="active" #rla="routerLinkActive">
  Users {{ rla.isActive ? '(current)' : '' }}
</a>
```

### Programmatic Navigation

```typescript
import { Router, ActivatedRoute } from '@angular/router';

@Component({...})
export class NavigationComponent {
  constructor(
    private router: Router,
    private route: ActivatedRoute
  ) {}

  // Absolute navigation
  goToUsers() {
    this.router.navigate(['/users']);
  }

  // With parameters
  goToUser(id: number) {
    this.router.navigate(['/users', id]);
  }

  // With query params
  goToUsersWithParams() {
    this.router.navigate(['/users'], {
      queryParams: { page: 1, sort: 'name' },
      queryParamsHandling: 'merge'  // Merge with existing
    });
  }

  // With fragment
  goToSection() {
    this.router.navigate(['/help'], { fragment: 'faq' });
  }

  // Relative navigation
  goToDetails() {
    this.router.navigate(['details'], { relativeTo: this.route });
  }

  goUp() {
    this.router.navigate(['..'], { relativeTo: this.route });
  }

  // Replace current history entry
  replaceNavigation() {
    this.router.navigate(['/new-page'], { replaceUrl: true });
  }

  // Skip location change (no URL update)
  silentNavigation() {
    this.router.navigate(['/page'], { skipLocationChange: true });
  }

  // Using navigateByUrl (full URL)
  goByUrl() {
    this.router.navigateByUrl('/users/123?tab=profile#info');
  }

  // With state (hidden data)
  goWithState() {
    this.router.navigate(['/checkout'], {
      state: { cart: this.cartItems }
    });
  }

  // Access state in target component
  ngOnInit() {
    const state = this.router.getCurrentNavigation()?.extras.state;
    // Or
    const state2 = history.state;
  }
}
```

---

## Route Parameters

### Route Parameters (Path Variables)

```typescript
// Route configuration
const routes: Routes = [
  { path: 'users/:id', component: UserDetailComponent },
  { path: 'posts/:postId/comments/:commentId', component: CommentComponent }
];

// Component - Reading parameters
@Component({...})
export class UserDetailComponent implements OnInit {
  private route = inject(ActivatedRoute);

  // Method 1: Snapshot (one-time read)
  ngOnInit() {
    const id = this.route.snapshot.paramMap.get('id');
    console.log('User ID:', id);
  }

  // Method 2: Observable (for param changes)
  userId$ = this.route.paramMap.pipe(
    map(params => params.get('id')),
    filter((id): id is string => id !== null)
  );

  // Method 3: Using resolve
  user$ = this.route.data.pipe(
    map(data => data['user'])
  );
}

// For multiple params
@Component({...})
export class CommentComponent {
  private route = inject(ActivatedRoute);

  ngOnInit() {
    this.route.paramMap.subscribe(params => {
      const postId = params.get('postId');
      const commentId = params.get('commentId');
    });
  }
}
```

### Query Parameters

```typescript
// URL: /users?page=1&sort=name&filter=active

@Component({...})
export class UsersComponent implements OnInit {
  private route = inject(ActivatedRoute);
  private router = inject(Router);

  // Read query params
  ngOnInit() {
    // Snapshot
    const page = this.route.snapshot.queryParamMap.get('page');

    // Observable
    this.route.queryParamMap.subscribe(params => {
      const page = params.get('page');
      const sort = params.get('sort');
      const filters = params.getAll('filter');  // Multiple values
    });

    // Or as object
    this.route.queryParams.subscribe(params => {
      console.log(params);  // { page: '1', sort: 'name', filter: 'active' }
    });
  }

  // Update query params
  changePage(page: number) {
    this.router.navigate([], {
      relativeTo: this.route,
      queryParams: { page },
      queryParamsHandling: 'merge'  // Keep other params
    });
  }

  // Remove query param
  clearFilter() {
    this.router.navigate([], {
      relativeTo: this.route,
      queryParams: { filter: null },
      queryParamsHandling: 'merge'
    });
  }
}
```

### URL Fragment

```typescript
// URL: /help#faq

@Component({...})
export class HelpComponent implements OnInit {
  private route = inject(ActivatedRoute);

  ngOnInit() {
    // Snapshot
    const fragment = this.route.snapshot.fragment;

    // Observable
    this.route.fragment.subscribe(fragment => {
      if (fragment) {
        document.getElementById(fragment)?.scrollIntoView();
      }
    });
  }
}
```

### Route Data

```typescript
// Static data in route config
const routes: Routes = [
  {
    path: 'admin',
    component: AdminComponent,
    data: {
      title: 'Admin Panel',
      roles: ['admin', 'superadmin'],
      animation: 'AdminPage'
    }
  }
];

// Access in component
@Component({...})
export class AdminComponent implements OnInit {
  private route = inject(ActivatedRoute);

  ngOnInit() {
    // Snapshot
    const title = this.route.snapshot.data['title'];

    // Observable (includes resolved data)
    this.route.data.subscribe(data => {
      console.log(data.title);
      console.log(data.roles);
    });
  }
}
```

---

## Child & Nested Routes

### Basic Child Routes

```typescript
const routes: Routes = [
  {
    path: "users",
    component: UsersComponent,
    children: [
      { path: "", component: UserListComponent },
      { path: ":id", component: UserDetailComponent },
      { path: ":id/edit", component: UserEditComponent },
    ],
  },
];

// Parent component MUST have <router-outlet>
@Component({
  selector: "app-users",
  template: `
    <h1>Users Section</h1>
    <router-outlet></router-outlet>
  `,
})
export class UsersComponent {}
```

### Accessing Parent Route Data

```typescript
@Component({...})
export class UserEditComponent implements OnInit {
  private route = inject(ActivatedRoute);

  ngOnInit() {
    // Get parent params
    const userId = this.route.parent?.snapshot.paramMap.get('id');

    // Or traverse up
    let currentRoute = this.route;
    while (currentRoute.parent) {
      console.log(currentRoute.snapshot.params);
      currentRoute = currentRoute.parent;
    }

    // Using paramsInheritanceStrategy in router config
    // Allows accessing all ancestor params directly
  }
}

// Router config to inherit params
provideRouter(routes, withRouterConfig({
  paramsInheritanceStrategy: 'always'  // 'emptyOnly' is default
}));
```

### Named Outlets

```typescript
const routes: Routes = [
  { path: 'dashboard', component: DashboardComponent },
  { path: 'notifications', component: NotificationsComponent, outlet: 'sidebar' },
  { path: 'chat', component: ChatComponent, outlet: 'sidebar' }
];

// Template with named outlets
@Component({
  template: `
    <router-outlet></router-outlet>           <!-- Primary -->
    <router-outlet name="sidebar"></router-outlet>  <!-- Named -->
  `
})
export class AppComponent {}

// Navigation to named outlets
<a [routerLink]="[{ outlets: { sidebar: 'notifications' } }]">
  Show Notifications
</a>

<a [routerLink]="['/dashboard', { outlets: { sidebar: 'chat' } }]">
  Dashboard with Chat
</a>

// Clear named outlet
<a [routerLink]="[{ outlets: { sidebar: null } }]">
  Close Sidebar
</a>

// Programmatic
this.router.navigate([{ outlets: { sidebar: 'notifications' } }]);
```

---

## Lazy Loading

### Lazy Loading Modules

```typescript
// app.routes.ts
const routes: Routes = [
  { path: "", component: HomeComponent },
  {
    path: "admin",
    loadChildren: () =>
      import("./admin/admin.routes").then((m) => m.ADMIN_ROUTES),
  },
  {
    path: "users",
    loadChildren: () =>
      import("./users/users.module").then((m) => m.UsersModule),
  },
];

// admin/admin.routes.ts
export const ADMIN_ROUTES: Routes = [
  { path: "", component: AdminDashboardComponent },
  { path: "settings", component: AdminSettingsComponent },
];

// users/users.module.ts (NgModule approach)
@NgModule({
  imports: [
    RouterModule.forChild([
      { path: "", component: UsersListComponent },
      { path: ":id", component: UserDetailComponent },
    ]),
  ],
})
export class UsersModule {}
```

### Lazy Loading Standalone Components

```typescript
const routes: Routes = [
  {
    path: "profile",
    loadComponent: () =>
      import("./profile/profile.component").then((m) => m.ProfileComponent),
  },
  {
    path: "settings",
    loadComponent: () =>
      import("./settings/settings.component").then((m) => m.SettingsComponent),
    providers: [SettingsService], // Route-level providers
  },
];
```

### Preloading Strategies

```typescript
import { PreloadAllModules, NoPreloading } from "@angular/router";

// Preload all lazy modules after initial load
provideRouter(routes, withPreloading(PreloadAllModules));

// Or with NgModule
RouterModule.forRoot(routes, {
  preloadingStrategy: PreloadAllModules,
});

// Custom preloading strategy
@Injectable({ providedIn: "root" })
export class SelectivePreloadingStrategy implements PreloadingStrategy {
  preload(route: Route, load: () => Observable<any>): Observable<any> {
    // Preload only routes with data.preload = true
    return route.data?.["preload"] ? load() : of(null);
  }
}

const routes: Routes = [
  {
    path: "admin",
    loadChildren: () => import("./admin/admin.routes"),
    data: { preload: true }, // Will be preloaded
  },
  {
    path: "reports",
    loadChildren: () => import("./reports/reports.routes"),
    data: { preload: false }, // Won't be preloaded
  },
];

provideRouter(routes, withPreloading(SelectivePreloadingStrategy));
```

---

## Route Guards

### Guard Types Overview

```
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚                           ROUTE GUARDS                                    â”‚
â”œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¤
â”‚                                                                           â”‚
â”‚  canMatch        â†’ Can the route be matched? (Lazy loading decision)     â”‚
â”‚  canActivate     â†’ Can the route be activated?                           â”‚
â”‚  canActivateChild â†’ Can child routes be activated?                       â”‚
â”‚  canDeactivate   â†’ Can the user leave the route?                         â”‚
â”‚  resolve         â†’ Pre-fetch data before activation                      â”‚
â”‚                                                                           â”‚
â”‚  EXECUTION ORDER:                                                        â”‚
â”‚  canMatch â†’ canActivate â†’ canActivateChild â†’ resolve â†’ activate          â”‚
â”‚                                                                           â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

### Functional Guards (Angular 14+)

```typescript
// auth.guard.ts
import { inject } from "@angular/core";
import { CanActivateFn, Router } from "@angular/router";

export const authGuard: CanActivateFn = (route, state) => {
  const authService = inject(AuthService);
  const router = inject(Router);

  if (authService.isLoggedIn()) {
    return true;
  }

  // Redirect to login with return URL
  return router.createUrlTree(["/login"], {
    queryParams: { returnUrl: state.url },
  });
};

// Role guard
export const roleGuard: CanActivateFn = (route, state) => {
  const authService = inject(AuthService);
  const requiredRoles = route.data["roles"] as string[];

  return requiredRoles.some((role) => authService.hasRole(role));
};

// Route configuration
const routes: Routes = [
  {
    path: "admin",
    component: AdminComponent,
    canActivate: [authGuard, roleGuard],
    data: { roles: ["admin", "superadmin"] },
  },
];
```

### Class-Based Guards (Traditional)

```typescript
@Injectable({ providedIn: "root" })
export class AuthGuard implements CanActivate {
  constructor(private authService: AuthService, private router: Router) {}

  canActivate(
    route: ActivatedRouteSnapshot,
    state: RouterStateSnapshot
  ): boolean | UrlTree | Observable<boolean | UrlTree> {
    if (this.authService.isLoggedIn()) {
      return true;
    }

    return this.router.createUrlTree(["/login"], {
      queryParams: { returnUrl: state.url },
    });
  }
}
```

### CanDeactivate Guard (Unsaved Changes)

```typescript
// Define interface for components with unsaved changes
export interface CanComponentDeactivate {
  canDeactivate: () => boolean | Observable<boolean>;
}

// Functional guard
export const unsavedChangesGuard: CanDeactivateFn<CanComponentDeactivate> =
  (component, currentRoute, currentState, nextState) => {
    return component.canDeactivate ? component.canDeactivate() : true;
  };

// Component implementation
@Component({...})
export class EditFormComponent implements CanComponentDeactivate {
  form = inject(FormBuilder).group({
    name: [''],
    email: ['']
  });

  canDeactivate(): boolean {
    if (this.form.dirty) {
      return confirm('You have unsaved changes. Leave anyway?');
    }
    return true;
  }
}

// Route config
const routes: Routes = [
  {
    path: 'edit',
    component: EditFormComponent,
    canDeactivate: [unsavedChangesGuard]
  }
];
```

### CanMatch Guard (Angular 14.1+)

```typescript
// Only match route if condition is met
export const featureGuard: CanMatchFn = (route, segments) => {
  const featureService = inject(FeatureService);
  return featureService.isFeatureEnabled("newDashboard");
};

const routes: Routes = [
  {
    path: "dashboard",
    canMatch: [featureGuard],
    loadComponent: () => import("./new-dashboard.component"),
  },
  {
    path: "dashboard",
    loadComponent: () => import("./old-dashboard.component"),
  },
];
// If feature enabled: loads new dashboard
// If feature disabled: falls through to old dashboard
```

### Combining Multiple Guards

```typescript
const routes: Routes = [
  {
    path: "admin",
    canActivate: [authGuard, roleGuard, subscriptionGuard],
    canActivateChild: [roleGuard],
    canDeactivate: [unsavedChangesGuard],
    canMatch: [featureGuard],
    component: AdminComponent,
    children: [
      { path: "users", component: AdminUsersComponent },
      { path: "settings", component: AdminSettingsComponent },
    ],
  },
];

// Guards run in order - if any returns false/UrlTree, navigation stops
```

---

## Resolvers

### Data Resolvers

Resolvers pre-fetch data before route activation.

```typescript
// user.resolver.ts
import { ResolveFn } from '@angular/router';

export const userResolver: ResolveFn<User> = (route, state) => {
  const userService = inject(UserService);
  const userId = route.paramMap.get('id')!;

  return userService.getUser(userId).pipe(
    catchError(error => {
      // Handle error - maybe redirect
      const router = inject(Router);
      router.navigate(['/users']);
      return EMPTY;
    })
  );
};

// Route config
const routes: Routes = [
  {
    path: 'users/:id',
    component: UserDetailComponent,
    resolve: { user: userResolver }
  }
];

// Component
@Component({...})
export class UserDetailComponent implements OnInit {
  private route = inject(ActivatedRoute);

  // Resolved data is available immediately
  user = this.route.snapshot.data['user'];

  // Or as observable
  user$ = this.route.data.pipe(
    map(data => data['user'] as User)
  );
}
```

### Class-Based Resolver

```typescript
@Injectable({ providedIn: "root" })
export class UserResolver implements Resolve<User> {
  constructor(private userService: UserService) {}

  resolve(
    route: ActivatedRouteSnapshot,
    state: RouterStateSnapshot
  ): Observable<User> {
    return this.userService.getUser(route.paramMap.get("id")!);
  }
}
```

### Multiple Resolvers

```typescript
const routes: Routes = [
  {
    path: 'dashboard',
    component: DashboardComponent,
    resolve: {
      user: userResolver,
      stats: statsResolver,
      notifications: notificationsResolver
    }
  }
];

// All resolvers run in parallel
// Component receives all data at once
@Component({...})
export class DashboardComponent {
  private route = inject(ActivatedRoute);

  data$ = this.route.data;
  // { user: User, stats: Stats, notifications: Notification[] }
}
```

---

## Router Events

### All Router Events

```typescript
import {
  Router,
  NavigationStart,
  NavigationEnd,
  NavigationCancel,
  NavigationError,
  GuardsCheckStart,
  GuardsCheckEnd,
  ResolveStart,
  ResolveEnd,
  RoutesRecognized
} from '@angular/router';

@Component({...})
export class AppComponent implements OnInit {
  private router = inject(Router);
  isLoading = false;

  ngOnInit() {
    this.router.events.subscribe(event => {
      if (event instanceof NavigationStart) {
        this.isLoading = true;
        console.log('Navigation started to:', event.url);
      }

      if (event instanceof NavigationEnd) {
        this.isLoading = false;
        console.log('Navigation ended:', event.urlAfterRedirects);
      }

      if (event instanceof NavigationCancel) {
        this.isLoading = false;
        console.log('Navigation cancelled:', event.reason);
      }

      if (event instanceof NavigationError) {
        this.isLoading = false;
        console.error('Navigation error:', event.error);
      }
    });
  }
}
```

### Filtering Events

```typescript
import { filter } from "rxjs/operators";

// Filter specific event types
this.router.events
  .pipe(filter((event) => event instanceof NavigationEnd))
  .subscribe((event: NavigationEnd) => {
    // Track page view
    this.analytics.trackPageView(event.urlAfterRedirects);
  });

// Or type-safe filter
this.router.events
  .pipe(
    filter((event): event is NavigationEnd => event instanceof NavigationEnd)
  )
  .subscribe((event) => {
    console.log(event.url);
  });
```

### Loading Indicator Service

```typescript
@Injectable({ providedIn: "root" })
export class LoadingService {
  private loadingSubject = new BehaviorSubject<boolean>(false);
  loading$ = this.loadingSubject.asObservable();

  constructor(private router: Router) {
    this.router.events
      .pipe(
        filter(
          (event) =>
            event instanceof NavigationStart ||
            event instanceof NavigationEnd ||
            event instanceof NavigationCancel ||
            event instanceof NavigationError
        )
      )
      .subscribe((event) => {
        this.loadingSubject.next(event instanceof NavigationStart);
      });
  }
}
```

---

## Advanced Patterns

### Page Title Strategy

```typescript
// Angular 14+ built-in title
const routes: Routes = [
  { path: "", component: HomeComponent, title: "Home" },
  { path: "about", component: AboutComponent, title: "About Us" },
  {
    path: "users/:id",
    component: UserComponent,
    title: userTitleResolver, // Dynamic title
  },
];

// Dynamic title resolver
export const userTitleResolver: ResolveFn<string> = (route) => {
  const userService = inject(UserService);
  const userId = route.paramMap.get("id")!;

  return userService.getUser(userId).pipe(map((user) => `User: ${user.name}`));
};

// Custom title strategy
@Injectable()
export class CustomTitleStrategy extends TitleStrategy {
  constructor(private title: Title) {
    super();
  }

  updateTitle(snapshot: RouterStateSnapshot): void {
    const title = this.buildTitle(snapshot);
    this.title.setTitle(title ? `${title} | MyApp` : "MyApp");
  }
}

// Provide custom strategy
provideRouter(
  routes,
  withRouterConfig({
    titleStrategy: CustomTitleStrategy,
  })
);
```

### Route Animations

```typescript
// Define animations
import {
  trigger,
  transition,
  style,
  query,
  animate,
  group,
} from "@angular/animations";

export const routeAnimations = trigger("routeAnimations", [
  transition("HomePage <=> AboutPage", [
    style({ position: "relative" }),
    query(":enter, :leave", [
      style({
        position: "absolute",
        top: 0,
        left: 0,
        width: "100%",
      }),
    ]),
    query(":enter", [style({ left: "-100%" })]),
    query(":leave", [animate("300ms ease-out", style({ left: "100%" }))]),
    query(":enter", [animate("300ms ease-out", style({ left: "0%" }))]),
  ]),
]);

// Add data to routes
const routes: Routes = [
  { path: "", component: HomeComponent, data: { animation: "HomePage" } },
  {
    path: "about",
    component: AboutComponent,
    data: { animation: "AboutPage" },
  },
];

// Component
@Component({
  template: `
    <div [@routeAnimations]="getRouteAnimationData()">
      <router-outlet></router-outlet>
    </div>
  `,
  animations: [routeAnimations],
})
export class AppComponent {
  getRouteAnimationData() {
    return this.route.snapshot.firstChild?.data["animation"];
  }
}
```

### Scroll Position Restoration

```typescript
// Enable scroll restoration
provideRouter(
  routes,
  withInMemoryScrolling({
    scrollPositionRestoration: "enabled", // 'disabled' | 'enabled' | 'top'
    anchorScrolling: "enabled", // Enable #fragment scrolling
  })
);

// With NgModule
RouterModule.forRoot(routes, {
  scrollPositionRestoration: "enabled",
  anchorScrolling: "enabled",
  scrollOffset: [0, 64], // Offset for fixed header
});

// Custom scroll behavior
@Injectable()
export class CustomScrollBehavior implements ViewportScroller {
  // Implement custom logic
}
```

### URL Serializer

```typescript
// Custom URL serialization (e.g., lowercase URLs)
@Injectable()
export class LowerCaseUrlSerializer implements UrlSerializer {
  private defaultSerializer = new DefaultUrlSerializer();

  parse(url: string): UrlTree {
    return this.defaultSerializer.parse(url.toLowerCase());
  }

  serialize(tree: UrlTree): string {
    return this.defaultSerializer.serialize(tree).toLowerCase();
  }
}

// Provide
{ provide: UrlSerializer, useClass: LowerCaseUrlSerializer }
```

---

## Interview Questions & Answers

### Q1: What is the difference between RouterModule.forRoot() and RouterModule.forChild()?

**Answer:**

| Aspect | forRoot() | forChild() |
|--------|-----------|------------|
| Usage | App module (once) | Feature modules |
| Services | Creates Router, registers routes | Only registers routes |
| Location | Main app.module.ts | Lazy-loaded modules |

```typescript
// app.module.ts - Main application
@NgModule({
  imports: [RouterModule.forRoot(routes, {
    preloadingStrategy: PreloadAllModules,
    scrollPositionRestoration: 'enabled'
  })]
})
export class AppModule {}

// feature.module.ts - Feature module
@NgModule({
  imports: [RouterModule.forChild(featureRoutes)]
})
export class FeatureModule {}
```

**Important:** Using forRoot() in a lazy-loaded module creates duplicate Router service!

---

### Q2: How do you implement lazy loading in Angular?

**Answer:**

```typescript
// Modern approach (Angular 14+)
const routes: Routes = [
  {
    path: 'admin',
    loadComponent: () => import('./admin/admin.component')
      .then(m => m.AdminComponent)
  },
  {
    path: 'dashboard',
    loadChildren: () => import('./dashboard/dashboard.routes')
      .then(m => m.DASHBOARD_ROUTES)
  }
];

// With preloading strategy
@NgModule({
  imports: [RouterModule.forRoot(routes, {
    preloadingStrategy: PreloadAllModules // or custom strategy
  })]
})
```

**Custom preloading strategy:**

```typescript
@Injectable({ providedIn: 'root' })
export class SelectivePreloadingStrategy implements PreloadingStrategy {
  preload(route: Route, load: () => Observable<any>): Observable<any> {
    return route.data?.['preload'] ? load() : of(null);
  }
}

// Route with preload flag
{ path: 'reports', loadChildren: ..., data: { preload: true } }
```

---

### Q3: Explain route guards and their execution order.

**Answer:**

**Guard Types:**

| Guard | Purpose | Runs When |
|-------|---------|-----------|
| canMatch | Check if route can match | Before route matching |
| canActivate | Allow/deny navigation | Before activating route |
| canActivateChild | Protect child routes | Before activating children |
| canDeactivate | Allow leaving route | Before leaving current route |
| canLoad | Control lazy loading | Before loading module |

**Execution Order:**

```
URL Change  canMatch  canActivate  canActivateChild  Resolvers  Component
                                                                       
         canLoad (if lazy)                                     canDeactivate
```

```typescript
// Functional guard (modern approach)
export const authGuard: CanActivateFn = (route, state) => {
  const auth = inject(AuthService);
  const router = inject(Router);
  
  if (auth.isAuthenticated()) {
    return true;
  }
  
  return router.createUrlTree(['/login'], {
    queryParams: { returnUrl: state.url }
  });
};

// Usage
{ path: 'admin', component: AdminComponent, canActivate: [authGuard] }
```

---

### Q4: What is the difference between route params and query params?

**Answer:**

| Aspect | Route Params | Query Params |
|--------|--------------|--------------|
| URL Format | /users/:id  /users/123 | /users?sort=name&page=1 |
| Required | Yes | No (optional) |
| Route Config | Defined in path | Not in route config |
| Use Case | Resource identification | Filtering, pagination |

```typescript
// Route params
{ path: 'products/:category/:id', component: ProductComponent }
// URL: /products/electronics/123

// Accessing params
@Component({...})
export class ProductComponent {
  private route = inject(ActivatedRoute);
  
  ngOnInit() {
    // Route params (reactive)
    this.route.paramMap.subscribe(params => {
      this.categoryId = params.get('category');
      this.productId = params.get('id');
    });
    
    // Query params
    this.route.queryParamMap.subscribe(params => {
      this.sortBy = params.get('sort');
      this.page = +(params.get('page') ?? 1);
    });
  }
}

// Navigating with both
this.router.navigate(['/products', 'electronics', 123], {
  queryParams: { sort: 'price', order: 'desc' },
  queryParamsHandling: 'merge' // preserve existing
});
```

---

### Q5: How do resolvers work and when should you use them?

**Answer:**

**Resolvers** pre-fetch data before a route activates.

**Flow:**
1. Navigation starts  Guards run  Resolvers run  Component activates

```typescript
// Resolver function
export const productResolver: ResolveFn<Product> = (route) => {
  const service = inject(ProductService);
  const router = inject(Router);
  
  return service.getProduct(route.paramMap.get('id')!).pipe(
    catchError(() => {
      router.navigate(['/404']);
      return EMPTY;
    })
  );
};

// Route configuration
{
  path: 'products/:id',
  component: ProductDetailComponent,
  resolve: { product: productResolver }
}

// Component - data available immediately
export class ProductDetailComponent {
  product = inject(ActivatedRoute).snapshot.data['product'];
}
```

**Use When:** Critical data needed before render, prevent loading states
**Avoid When:** Large datasets (blocks navigation), optional data

---

### Q6: How do you handle navigation errors and 404 routes?

**Answer:**

```typescript
const routes: Routes = [
  { path: '', component: HomeComponent },
  { path: 'products', loadChildren: () => import('./products/routes') },
  
  // 404 catch-all (must be last)
  { path: '**', component: NotFoundComponent }
];

// Global error handling via Router events
@Injectable({ providedIn: 'root' })
export class NavigationErrorHandler {
  constructor(private router: Router) {
    this.router.events.pipe(
      filter(e => e instanceof NavigationError)
    ).subscribe((error: NavigationError) => {
      console.error('Navigation failed:', error.url);
      this.router.navigate(['/error'], {
        queryParams: { url: error.url }
      });
    });
  }
}
```

---

### Q7: Explain the difference between snapshot and paramMap observable.

**Answer:**

| Aspect | snapshot | paramMap (Observable) |
|--------|----------|----------------------|
| Type | Static value | Stream of values |
| Updates | One-time read | Reacts to changes |
| Use case | Initial load | Same component, different params |

```typescript
export class UserComponent {
  private route = inject(ActivatedRoute);
  
  // Snapshot - won't update if params change on same component
  userId = this.route.snapshot.paramMap.get('id');
  
  // Observable - updates when navigating /users/1  /users/2
  ngOnInit() {
    this.route.paramMap.subscribe(params => {
      this.loadUser(params.get('id'));
    });
  }
}
```

**When to use Observable:** Links within component navigate to same route with different params (e.g., "Next/Previous" buttons).

---

### Q8: How do you implement route animations?

**Answer:**

```typescript
// animations.ts
export const routeAnimations = trigger('routeAnimations', [
  transition('* <=> *', [
    query(':enter, :leave', [
      style({
        position: 'absolute',
        width: '100%',
        opacity: 0,
        transform: 'translateX(100%)'
      })
    ], { optional: true }),
    query(':leave', [
      animate('300ms ease-out', style({ opacity: 0, transform: 'translateX(-100%)' }))
    ], { optional: true }),
    query(':enter', [
      animate('300ms ease-out', style({ opacity: 1, transform: 'translateX(0)' }))
    ], { optional: true })
  ])
]);

// app.component.ts
@Component({
  animations: [routeAnimations],
  template: `
    <div [@routeAnimations]="getRouteAnimationData()">
      <router-outlet></router-outlet>
    </div>
  `
})
export class AppComponent {
  getRouteAnimationData() {
    return this.route.snapshot.firstChild?.data['animation'];
  }
}

// Route config
{ path: 'home', component: HomeComponent, data: { animation: 'HomePage' } }
```

---

### Q9: How do you pass data between routes?

**Answer:**

```typescript
// Method 1: Route params
this.router.navigate(['/user', userId]);

// Method 2: Query params
this.router.navigate(['/search'], { 
  queryParams: { q: 'angular', page: 1 } 
});

// Method 3: State (hidden from URL)
this.router.navigate(['/checkout'], { 
  state: { cart: this.cartItems } 
});

// Accessing state
constructor() {
  const navigation = this.router.getCurrentNavigation();
  this.cart = navigation?.extras.state?.['cart'];
  // OR
  this.cart = history.state.cart;
}

// Method 4: Route data (static)
{ path: 'admin', component: AdminComponent, data: { role: 'admin' } }

// Method 5: Shared service (for complex data)
@Injectable({ providedIn: 'root' })
export class DataTransferService {
  private data = signal<any>(null);
  setData(data: any) { this.data.set(data); }
  getData() { return this.data(); }
}
```

---

### Q10: What is ActivatedRoute and how do you use it?

**Answer:**

```typescript
@Component({...})
export class ProductComponent {
  private route = inject(ActivatedRoute);
  
  ngOnInit() {
    // Access various route information
    
    // URL segments
    this.route.url.subscribe(segments => {
      console.log('URL:', segments.map(s => s.path).join('/'));
    });
    
    // Route params
    this.route.paramMap.subscribe(params => {
      this.id = params.get('id');
    });
    
    // Query params
    this.route.queryParamMap.subscribe(params => {
      this.filter = params.get('filter');
    });
    
    // Fragment (#anchor)
    this.route.fragment.subscribe(f => console.log('Fragment:', f));
    
    // Static route data
    this.route.data.subscribe(data => {
      this.title = data['title'];
    });
    
    // Parent route access
    this.route.parent?.paramMap.subscribe(...);
    
    // Children routes
    this.route.children; // ActivatedRoute[]
  }
}
```

---

### Q11: How do you implement breadcrumbs using Angular Router?

**Answer:**

```typescript
// Route config with breadcrumb data
const routes: Routes = [
  { path: '', data: { breadcrumb: 'Home' }, children: [
    { path: 'products', data: { breadcrumb: 'Products' }, children: [
      { path: ':id', data: { breadcrumb: 'Details' }, component: ProductDetailComponent }
    ]}
  ]}
];

// Breadcrumb service
@Injectable({ providedIn: 'root' })
export class BreadcrumbService {
  breadcrumbs$ = this.router.events.pipe(
    filter(e => e instanceof NavigationEnd),
    map(() => this.buildBreadcrumbs(this.route.root))
  );
  
  private buildBreadcrumbs(route: ActivatedRoute, url = '', breadcrumbs: any[] = []): any[] {
    const children = route.children;
    
    for (const child of children) {
      const routeUrl = child.snapshot.url.map(s => s.path).join('/');
      url += routeUrl ? '/' + routeUrl : '';
      
      const label = child.snapshot.data['breadcrumb'];
      if (label) {
        breadcrumbs.push({ label, url });
      }
      
      return this.buildBreadcrumbs(child, url, breadcrumbs);
    }
    
    return breadcrumbs;
  }
}
```

---

### Q12: How do you handle authentication redirects with returnUrl?

**Answer:**

```typescript
// Auth guard with returnUrl
export const authGuard: CanActivateFn = (route, state) => {
  const auth = inject(AuthService);
  const router = inject(Router);
  
  if (auth.isLoggedIn()) {
    return true;
  }
  
  // Store attempted URL for redirecting
  return router.createUrlTree(['/login'], {
    queryParams: { returnUrl: state.url }
  });
};

// Login component
@Component({...})
export class LoginComponent {
  private route = inject(ActivatedRoute);
  private router = inject(Router);
  
  onLoginSuccess() {
    const returnUrl = this.route.snapshot.queryParams['returnUrl'] || '/dashboard';
    this.router.navigateByUrl(returnUrl);
  }
}
```

---

### Q13: What are secondary (named) routes and when to use them?

**Answer:**

```typescript
// Route configuration
const routes: Routes = [
  { path: 'products', component: ProductListComponent },
  { path: 'help', component: HelpComponent, outlet: 'sidebar' },
  { path: 'chat', component: ChatComponent, outlet: 'sidebar' }
];

// Template with named outlet
```html
<router-outlet></router-outlet>
<router-outlet name="sidebar"></router-outlet>
```

// Navigation
// URL: /products(sidebar:help)
<a [routerLink]="['/products', { outlets: { sidebar: 'help' } }]">
  Show Help
</a>

// Programmatic
this.router.navigate(['/products', { outlets: { sidebar: 'chat' } }]);

// Close named outlet
this.router.navigate([{ outlets: { sidebar: null } }]);
```

**Use cases:** Side panels, modals, chat widgets that coexist with main content

---

### Q14: How do you preload modules based on user actions?

**Answer:**

```typescript
// Preload service for on-demand loading
@Injectable({ providedIn: 'root' })
export class OnDemandPreloadService implements PreloadingStrategy {
  private preloadSubject = new Subject<string>();
  
  preload(route: Route, load: () => Observable<any>): Observable<any> {
    return this.preloadSubject.pipe(
      filter(path => route.path === path),
      take(1),
      switchMap(() => load())
    );
  }
  
  startPreload(path: string) {
    this.preloadSubject.next(path);
  }
}

// Usage - preload on hover
@Directive({ selector: '[preloadOnHover]' })
export class PreloadOnHoverDirective {
  @Input() preloadOnHover!: string;
  
  constructor(private preloadService: OnDemandPreloadService) {}
  
  @HostListener('mouseenter')
  onHover() {
    this.preloadService.startPreload(this.preloadOnHover);
  }
}

// Template
<a routerLink="/reports" [preloadOnHover]="'reports'">Reports</a>
```

---

### Q15: How do you handle route reuse strategies?

**Answer:**

```typescript
// Custom RouteReuseStrategy
@Injectable()
export class CacheRouteReuseStrategy implements RouteReuseStrategy {
  private cache = new Map<string, DetachedRouteHandle>();
  
  shouldDetach(route: ActivatedRouteSnapshot): boolean {
    return route.data['reuse'] === true;
  }
  
  store(route: ActivatedRouteSnapshot, handle: DetachedRouteHandle): void {
    const key = this.getKey(route);
    this.cache.set(key, handle);
  }
  
  shouldAttach(route: ActivatedRouteSnapshot): boolean {
    return this.cache.has(this.getKey(route));
  }
  
  retrieve(route: ActivatedRouteSnapshot): DetachedRouteHandle | null {
    return this.cache.get(this.getKey(route)) || null;
  }
  
  shouldReuseRoute(future: ActivatedRouteSnapshot, curr: ActivatedRouteSnapshot): boolean {
    return future.routeConfig === curr.routeConfig;
  }
  
  private getKey(route: ActivatedRouteSnapshot): string {
    return route.pathFromRoot.map(r => r.url.join('/')).join('/');
  }
}

// Register strategy
providers: [
  { provide: RouteReuseStrategy, useClass: CacheRouteReuseStrategy }
]

// Mark route as reusable
{ path: 'search', component: SearchComponent, data: { reuse: true } }
```

---
## Summary Checklist

âœ… **Route Configuration**

- Understand all route properties
- Know pathMatch strategies
- Handle wildcards correctly

âœ… **Navigation**

- RouterLink for templates
- Router.navigate for code
- Handle query params and fragments

âœ… **Child Routes**

- Nested router-outlets
- Accessing parent params
- Named outlets

âœ… **Lazy Loading**

- loadChildren vs loadComponent
- Preloading strategies
- Route-level providers

âœ… **Guards**

- canActivate, canDeactivate, canMatch
- Functional vs class-based
- Return types (boolean, UrlTree, Observable)

âœ… **Resolvers**

- Pre-fetch data
- Handle errors
- Know when to use

---

**Key Takeaway:** Angular Router is powerful but complex. Master guards and resolvers for production apps, use lazy loading for performance, and always consider the user experience when handling navigation states.
