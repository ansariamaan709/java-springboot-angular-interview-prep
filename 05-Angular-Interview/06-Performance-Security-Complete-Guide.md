# Angular Performance & Security - Complete Interview Guide

## Table of Contents

1. [Change Detection](#change-detection)
2. [OnPush Strategy](#onpush-strategy)
3. [Lazy Loading & Code Splitting](#lazy-loading--code-splitting)
4. [Bundle Optimization](#bundle-optimization)
5. [Runtime Performance](#runtime-performance)
6. [Memory Management](#memory-management)
7. [Security Best Practices](#security-best-practices)
8. [XSS Prevention](#xss-prevention)
9. [CSRF Protection](#csrf-protection)
10. [Authentication & Authorization](#authentication--authorization)
11. [Interview Questions & Answers](#interview-questions--answers)

---

## Change Detection

### How Change Detection Works

```
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚                    ANGULAR CHANGE DETECTION                              â”‚
â”œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¤
â”‚                                                                          â”‚
â”‚  Zone.js patches async APIs (setTimeout, Promise, DOM events, etc.)     â”‚
â”‚                              â”‚                                           â”‚
â”‚                              â–¼                                           â”‚
â”‚                     â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”                                   â”‚
â”‚                     â”‚ Async Event   â”‚                                   â”‚
â”‚                     â”‚ (click, http) â”‚                                   â”‚
â”‚                     â””â”€â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”€â”˜                                   â”‚
â”‚                             â”‚                                           â”‚
â”‚                             â–¼                                           â”‚
â”‚                     â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”                                   â”‚
â”‚                     â”‚   Zone.js     â”‚                                   â”‚
â”‚                     â”‚   Notifies    â”‚                                   â”‚
â”‚                     â””â”€â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”€â”˜                                   â”‚
â”‚                             â”‚                                           â”‚
â”‚                             â–¼                                           â”‚
â”‚                     â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”                                   â”‚
â”‚                     â”‚    Angular    â”‚                                   â”‚
â”‚                     â”‚    CD Run     â”‚                                   â”‚
â”‚                     â””â”€â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”€â”˜                                   â”‚
â”‚                             â”‚                                           â”‚
â”‚              â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”                           â”‚
â”‚              â”‚              â”‚              â”‚                            â”‚
â”‚              â–¼              â–¼              â–¼                            â”‚
â”‚        â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”  â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”  â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”                        â”‚
â”‚        â”‚Component â”‚  â”‚Component â”‚  â”‚Component â”‚                        â”‚
â”‚        â”‚    A     â”‚  â”‚    B     â”‚  â”‚    C     â”‚                        â”‚
â”‚        â””â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”˜  â””â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”˜  â””â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”˜                        â”‚
â”‚             â”‚             â”‚             â”‚                               â”‚
â”‚             â–¼             â–¼             â–¼                               â”‚
â”‚        Check & Update  Check & Update  Check & Update                  â”‚
â”‚             DOM            DOM            DOM                           â”‚
â”‚                                                                          â”‚
â”‚  DEFAULT Strategy: Checks ALL components top-to-bottom                  â”‚
â”‚  OnPush Strategy:  Checks ONLY when inputs change or events fire       â”‚
â”‚                                                                          â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

### Change Detection Triggers

```typescript
// These trigger change detection by default:
// 1. DOM events (click, input, etc.)
// 2. HTTP responses
// 3. setTimeout / setInterval
// 4. Promises
// 5. Manual trigger

@Component({...})
export class MyComponent {
  constructor(private cdr: ChangeDetectorRef) {}

  // Manual triggers
  triggerDetection() {
    this.cdr.detectChanges();    // Run CD for this component and children
    this.cdr.markForCheck();     // Mark component for check (OnPush)
  }

  // Detach from CD tree (advanced)
  detach() {
    this.cdr.detach();           // Remove from CD tree
    this.cdr.reattach();         // Re-add to CD tree
  }
}
```

---

## OnPush Strategy

### Enabling OnPush

```typescript
@Component({
  selector: "app-user-card",
  template: `
    <div class="card">
      <h3>{{ user.name }}</h3>
      <p>{{ user.email }}</p>
    </div>
  `,
  changeDetection: ChangeDetectionStrategy.OnPush, // Enable OnPush
})
export class UserCardComponent {
  @Input() user!: User;
}
```

### When OnPush Triggers Change Detection

```typescript
@Component({
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    <p>{{ data }}</p>
    <button (click)="onClick()">Click</button>
  `,
})
export class OnPushComponent {
  @Input() data!: string;

  constructor(private cdr: ChangeDetectorRef) {}

  // 1. Input reference changes (primitive or new object reference)
  // @Input() data changes â†’ triggers CD

  // 2. DOM event within component
  onClick() {
    // This click triggers CD for this component
  }

  // 3. Async pipe emits
  // Observable | async â†’ triggers CD

  // 4. Manual trigger
  manualTrigger() {
    this.cdr.markForCheck(); // Marks for next CD cycle
    this.cdr.detectChanges(); // Immediate CD
  }
}
```

### OnPush Gotchas

```typescript
// PROBLEM: Mutating objects doesn't trigger CD
@Component({
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: ` <p>{{ user.name }}</p> `,
})
export class UserComponent {
  @Input() user!: User;
}

// Parent component
@Component({
  template: `<app-user [user]="currentUser"></app-user>`,
})
export class ParentComponent {
  currentUser = { name: "John" };

  // BAD - Mutation, won't trigger child CD
  updateNameBad() {
    this.currentUser.name = "Jane"; // Same reference!
  }

  // GOOD - New reference
  updateNameGood() {
    this.currentUser = { ...this.currentUser, name: "Jane" };
  }
}
```

### Best Practices with OnPush

```typescript
// 1. Use immutable data patterns
updateUser(changes: Partial<User>) {
  this.user = { ...this.user, ...changes };
}

// 2. Use async pipe (auto marks for check)
@Component({
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    <div *ngIf="user$ | async as user">
      {{ user.name }}
    </div>
  `
})
export class UserComponent {
  user$ = this.userService.getUser();
}

// 3. Push data through observables
@Component({
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    <ul>
      <li *ngFor="let item of items$ | async">{{ item.name }}</li>
    </ul>
  `
})
export class ListComponent {
  items$ = this.store.select(selectItems);
}
```

---

## Lazy Loading & Code Splitting

### Lazy Loading Routes

```typescript
// app.routes.ts
const routes: Routes = [
  { path: "", component: HomeComponent },

  // Lazy load entire feature
  {
    path: "admin",
    loadChildren: () =>
      import("./admin/admin.routes").then((m) => m.ADMIN_ROUTES),
  },

  // Lazy load single component
  {
    path: "profile",
    loadComponent: () =>
      import("./profile/profile.component").then((m) => m.ProfileComponent),
  },
];
```

### Preloading Strategies

```typescript
import { PreloadAllModules } from '@angular/router';

// Preload all lazy modules after initial load
provideRouter(routes, withPreloading(PreloadAllModules));

// Custom preloading (selective)
@Injectable({ providedIn: 'root' })
export class SelectivePreloader implements PreloadingStrategy {
  preload(route: Route, load: () => Observable<any>): Observable<any> {
    // Only preload routes marked with data.preload = true
    if (route.data?.['preload']) {
      return load();
    }
    return of(null);
  }
}

// Route config
{
  path: 'dashboard',
  loadChildren: () => import('./dashboard/dashboard.routes'),
  data: { preload: true }  // Will be preloaded
}
```

### Dynamic Imports

```typescript
@Component({
  template: `
    <button (click)="loadChart()">Show Chart</button>
    <div #chartContainer></div>
  `,
})
export class ReportComponent {
  @ViewChild("chartContainer", { read: ViewContainerRef })
  container!: ViewContainerRef;

  async loadChart() {
    // Dynamically import heavy library only when needed
    const { ChartComponent } = await import("./chart/chart.component");
    this.container.createComponent(ChartComponent);
  }
}
```

---

## Bundle Optimization

### Analyzing Bundle Size

```bash
# Generate stats.json
ng build --stats-json

# Analyze with webpack-bundle-analyzer
npx webpack-bundle-analyzer dist/your-app/stats.json
```

### Tree Shaking

```typescript
// BAD - Imports entire library
import * as _ from "lodash";
_.map(items, fn);

// GOOD - Import only what you need
import map from "lodash-es/map";
map(items, fn);

// Or use subpath imports
import { map, filter } from "lodash-es";
```

### Production Build Optimizations

```json
// angular.json
{
  "configurations": {
    "production": {
      "budgets": [
        {
          "type": "initial",
          "maximumWarning": "500kb",
          "maximumError": "1mb"
        },
        {
          "type": "anyComponentStyle",
          "maximumWarning": "2kb",
          "maximumError": "4kb"
        }
      ],
      "outputHashing": "all",
      "optimization": true,
      "sourceMap": false,
      "namedChunks": false,
      "extractLicenses": true
    }
  }
}
```

### Differential Loading

```json
// .browserslistrc
# Support modern browsers
last 2 Chrome versions
last 2 Firefox versions
last 2 Safari versions
last 2 Edge versions

# Don't support IE (enables ES2015+ bundles)
not IE 11
```

---

## Runtime Performance

### trackBy for ngFor

```typescript
// WITHOUT trackBy - Angular recreates all DOM elements on change
@Component({
  template: `
    <ul>
      <li *ngFor="let item of items">{{ item.name }}</li>
    </ul>
  `,
})
export class ListBadComponent {
  items = [
    { id: 1, name: "A" },
    { id: 2, name: "B" },
  ];
}

// WITH trackBy - Angular reuses DOM elements
@Component({
  template: `
    <ul>
      <li *ngFor="let item of items; trackBy: trackById">{{ item.name }}</li>
    </ul>
  `,
})
export class ListGoodComponent {
  items = [
    { id: 1, name: "A" },
    { id: 2, name: "B" },
  ];

  trackById(index: number, item: Item): number {
    return item.id;
  }
}
```

### Pure Pipes vs Impure Pipes

```typescript
// PURE PIPE (default) - Only re-runs when input reference changes
@Pipe({ name: "filter", pure: true })
export class FilterPipe implements PipeTransform {
  transform(items: Item[], search: string): Item[] {
    return items.filter((item) => item.name.includes(search));
  }
}

// IMPURE PIPE - Runs on EVERY change detection cycle (expensive!)
@Pipe({ name: "filter", pure: false })
export class ImpureFilterPipe implements PipeTransform {
  transform(items: Item[], search: string): Item[] {
    return items.filter((item) => item.name.includes(search));
  }
}

// BEST: Avoid impure pipes, use component methods or computed values
```

### Avoiding Expensive Template Expressions

```typescript
// BAD - Function called on every CD cycle
@Component({
  template: `
    <div>{{ calculateTotal() }}</div>
    <!-- Called every CD! -->
  `,
})
export class BadComponent {
  calculateTotal(): number {
    return this.items.reduce((sum, item) => sum + item.price, 0);
  }
}

// GOOD - Use memoization or computed property
@Component({
  template: ` <div>{{ total }}</div> `,
})
export class GoodComponent {
  total = 0;

  updateTotal() {
    this.total = this.items.reduce((sum, item) => sum + item.price, 0);
  }
}

// BEST with Signals (Angular 16+)
@Component({
  template: `<div>{{ total() }}</div>`,
})
export class SignalComponent {
  items = signal<Item[]>([]);
  total = computed(() =>
    this.items().reduce((sum, item) => sum + item.price, 0)
  );
}
```

### Virtual Scrolling

```typescript
import { ScrollingModule } from "@angular/cdk/scrolling";

@Component({
  standalone: true,
  imports: [ScrollingModule],
  template: `
    <cdk-virtual-scroll-viewport itemSize="50" class="viewport">
      <div *cdkVirtualFor="let item of items" class="item">
        {{ item.name }}
      </div>
    </cdk-virtual-scroll-viewport>
  `,
  styles: [
    `
      .viewport {
        height: 400px;
      }
      .item {
        height: 50px;
      }
    `,
  ],
})
export class VirtualListComponent {
  items = Array.from({ length: 10000 }, (_, i) => ({ name: `Item ${i}` }));
}
```

---

## Memory Management

### Avoiding Memory Leaks

```typescript
// Common memory leak sources in Angular:
// 1. Unsubscribed Observables
// 2. Event listeners not removed
// 3. Timers not cleared
// 4. Detached DOM references

@Component({...})
export class MemoryLeakFreeComponent implements OnInit, OnDestroy {
  private destroy$ = new Subject<void>();
  private resizeHandler!: () => void;
  private intervalId!: number;

  ngOnInit() {
    // 1. Subscribe with takeUntil
    this.dataService.getData().pipe(
      takeUntil(this.destroy$)
    ).subscribe();

    // 2. Store event listener reference
    this.resizeHandler = () => this.onResize();
    window.addEventListener('resize', this.resizeHandler);

    // 3. Store timer reference
    this.intervalId = window.setInterval(() => this.poll(), 5000);
  }

  ngOnDestroy() {
    // 1. Complete destroy subject
    this.destroy$.next();
    this.destroy$.complete();

    // 2. Remove event listener
    window.removeEventListener('resize', this.resizeHandler);

    // 3. Clear timer
    clearInterval(this.intervalId);
  }

  private onResize() { /* ... */ }
  private poll() { /* ... */ }
}

// Angular 16+ with takeUntilDestroyed
@Component({...})
export class ModernComponent {
  private destroyRef = inject(DestroyRef);

  ngOnInit() {
    this.dataService.getData().pipe(
      takeUntilDestroyed(this.destroyRef)
    ).subscribe();
  }
}
```

### Detaching Components

```typescript
// For heavy components that don't need frequent updates
@Component({
  template: `<div>{{ heavyData }}</div>`,
})
export class HeavyComponent implements OnInit, OnDestroy {
  heavyData = "";

  constructor(private cdr: ChangeDetectorRef) {}

  ngOnInit() {
    // Detach from CD tree
    this.cdr.detach();

    // Manually run CD only when needed
    this.loadData();
  }

  loadData() {
    this.heavyData = this.processHeavyData();
    this.cdr.detectChanges(); // Manual update
  }
}
```

---

## Security Best Practices

### Security Overview

```
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚                    ANGULAR SECURITY LAYERS                               â”‚
â”œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¤
â”‚                                                                          â”‚
â”‚  CLIENT SIDE (Angular)                                                  â”‚
â”‚  â”œâ”€â”€ XSS Prevention (built-in sanitization)                            â”‚
â”‚  â”œâ”€â”€ Template security (no eval/innerHTML by default)                   â”‚
â”‚  â”œâ”€â”€ Route guards (authorization)                                       â”‚
â”‚  â””â”€â”€ HttpOnly cookie handling                                          â”‚
â”‚                                                                          â”‚
â”‚  TRANSPORT                                                              â”‚
â”‚  â”œâ”€â”€ HTTPS only                                                         â”‚
â”‚  â”œâ”€â”€ CSRF tokens                                                        â”‚
â”‚  â””â”€â”€ Secure headers (CSP, HSTS, etc.)                                  â”‚
â”‚                                                                          â”‚
â”‚  SERVER SIDE (API)                                                      â”‚
â”‚  â”œâ”€â”€ Authentication (JWT, OAuth)                                       â”‚
â”‚  â”œâ”€â”€ Authorization (roles, permissions)                                â”‚
â”‚  â”œâ”€â”€ Input validation                                                   â”‚
â”‚  â””â”€â”€ Rate limiting                                                      â”‚
â”‚                                                                          â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

---

## XSS Prevention

### Angular's Built-in Sanitization

```typescript
// Angular automatically sanitizes these contexts:
// - HTML (innerHTML)
// - Style (style bindings)
// - URL (href, src)
// - Resource URL (script src)

@Component({
  template: `
    <!-- SAFE: Angular sanitizes this -->
    <div [innerHTML]="userContent"></div>

    <!-- SAFE: Interpolation is always escaped -->
    <p>{{ userInput }}</p>

    <!-- SAFE: Property binding is sanitized -->
    <a [href]="userUrl">Link</a>
  `,
})
export class SafeComponent {
  userContent = '<script>alert("xss")</script><b>Bold</b>';
  // Rendered: <b>Bold</b> (script removed)

  userInput = '<script>alert("xss")</script>';
  // Rendered: &lt;script&gt;alert("xss")&lt;/script&gt; (escaped)

  userUrl = 'javascript:alert("xss")';
  // Rendered: unsafe:javascript:alert("xss") (prefixed with unsafe:)
}
```

### Bypassing Sanitization (Use Carefully!)

```typescript
import { DomSanitizer, SafeHtml, SafeUrl } from "@angular/platform-browser";

@Component({
  template: `
    <div [innerHTML]="trustedHtml"></div>
    <iframe [src]="trustedUrl"></iframe>
  `,
})
export class TrustedContentComponent {
  trustedHtml: SafeHtml;
  trustedUrl: SafeUrl;

  constructor(private sanitizer: DomSanitizer) {
    // DANGEROUS - Only use with trusted content!
    // Typically from your own server, never from users

    this.trustedHtml = this.sanitizer.bypassSecurityTrustHtml(
      '<video><source src="trusted.mp4"></video>'
    );

    this.trustedUrl = this.sanitizer.bypassSecurityTrustResourceUrl(
      "https://trusted-domain.com/embed"
    );
  }
}

// Bypass methods (all dangerous if misused):
// - bypassSecurityTrustHtml(value)
// - bypassSecurityTrustStyle(value)
// - bypassSecurityTrustScript(value)
// - bypassSecurityTrustUrl(value)
// - bypassSecurityTrustResourceUrl(value)
```

### Content Security Policy (CSP)

```html
<!-- index.html -->
<meta
  http-equiv="Content-Security-Policy"
  content="
  default-src 'self';
  script-src 'self';
  style-src 'self' 'unsafe-inline';
  img-src 'self' data: https:;
  font-src 'self';
  connect-src 'self' https://api.example.com;
"
/>
```

---

## CSRF Protection

### CSRF Token with Interceptor

```typescript
// CSRF interceptor
export const csrfInterceptor: HttpInterceptorFn = (req, next) => {
  // Get CSRF token from cookie (set by server)
  const csrfToken = getCookie("XSRF-TOKEN");

  if (csrfToken && !req.headers.has("X-XSRF-TOKEN")) {
    req = req.clone({
      setHeaders: { "X-XSRF-TOKEN": csrfToken },
    });
  }

  return next(req);
};

function getCookie(name: string): string | null {
  const match = document.cookie.match(new RegExp("(^| )" + name + "=([^;]+)"));
  return match ? match[2] : null;
}

// Angular's built-in XSRF support
// HttpClientXsrfModule automatically reads XSRF-TOKEN cookie
// and adds X-XSRF-TOKEN header

// With provideHttpClient
provideHttpClient(
  withXsrfConfiguration({
    cookieName: "XSRF-TOKEN",
    headerName: "X-XSRF-TOKEN",
  })
);
```

---

## Authentication & Authorization

### JWT Authentication Service

```typescript
@Injectable({ providedIn: "root" })
export class AuthService {
  private currentUserSubject = new BehaviorSubject<User | null>(null);
  currentUser$ = this.currentUserSubject.asObservable();

  private tokenKey = "auth_token";

  constructor(private http: HttpClient, private router: Router) {
    this.loadStoredUser();
  }

  login(credentials: LoginCredentials): Observable<AuthResponse> {
    return this.http.post<AuthResponse>("/api/auth/login", credentials).pipe(
      tap((response) => {
        this.storeToken(response.token);
        this.currentUserSubject.next(response.user);
      })
    );
  }

  logout(): void {
    localStorage.removeItem(this.tokenKey);
    this.currentUserSubject.next(null);
    this.router.navigate(["/login"]);
  }

  getToken(): string | null {
    return localStorage.getItem(this.tokenKey);
  }

  isAuthenticated(): boolean {
    const token = this.getToken();
    if (!token) return false;

    // Check if token is expired
    try {
      const payload = JSON.parse(atob(token.split(".")[1]));
      return payload.exp > Date.now() / 1000;
    } catch {
      return false;
    }
  }

  hasRole(role: string): boolean {
    const user = this.currentUserSubject.value;
    return user?.roles?.includes(role) ?? false;
  }

  private storeToken(token: string): void {
    localStorage.setItem(this.tokenKey, token);
  }

  private loadStoredUser(): void {
    const token = this.getToken();
    if (token && this.isAuthenticated()) {
      // Decode user from token or fetch from API
      const payload = JSON.parse(atob(token.split(".")[1]));
      this.currentUserSubject.next(payload.user);
    }
  }
}
```

### Auth Guard

```typescript
export const authGuard: CanActivateFn = (route, state) => {
  const authService = inject(AuthService);
  const router = inject(Router);

  if (authService.isAuthenticated()) {
    return true;
  }

  return router.createUrlTree(["/login"], {
    queryParams: { returnUrl: state.url },
  });
};

export const roleGuard: CanActivateFn = (route, state) => {
  const authService = inject(AuthService);
  const router = inject(Router);

  const requiredRoles = route.data["roles"] as string[];

  if (!authService.isAuthenticated()) {
    return router.createUrlTree(["/login"]);
  }

  const hasRole = requiredRoles.some((role) => authService.hasRole(role));

  if (hasRole) {
    return true;
  }

  return router.createUrlTree(["/unauthorized"]);
};

// Route config
const routes: Routes = [
  {
    path: "admin",
    canActivate: [authGuard, roleGuard],
    data: { roles: ["admin"] },
    loadChildren: () => import("./admin/admin.routes"),
  },
];
```

### Token Refresh

```typescript
export const tokenRefreshInterceptor: HttpInterceptorFn = (req, next) => {
  const authService = inject(AuthService);

  // Skip for auth endpoints
  if (req.url.includes("/auth/")) {
    return next(req);
  }

  return next(req).pipe(
    catchError((error: HttpErrorResponse) => {
      if (error.status === 401) {
        return authService.refreshToken().pipe(
          switchMap((newToken) => {
            const clonedReq = req.clone({
              setHeaders: { Authorization: `Bearer ${newToken}` },
            });
            return next(clonedReq);
          }),
          catchError((refreshError) => {
            authService.logout();
            return throwError(() => refreshError);
          })
        );
      }
      return throwError(() => error);
    })
  );
};
```

### Secure Storage Considerations

```typescript
// Token storage options:

// 1. localStorage - Vulnerable to XSS, persists
localStorage.setItem("token", token);

// 2. sessionStorage - Vulnerable to XSS, cleared on tab close
sessionStorage.setItem("token", token);

// 3. HttpOnly Cookie - Not accessible via JS, CSRF protection needed
// Set by server:
// Set-Cookie: token=xxx; HttpOnly; Secure; SameSite=Strict

// 4. Memory only - Most secure but lost on refresh
@Injectable({ providedIn: "root" })
export class TokenService {
  private token: string | null = null;

  setToken(token: string) {
    this.token = token;
  }
  getToken() {
    return this.token;
  }
  clearToken() {
    this.token = null;
  }
}

// Best practice for SPAs:
// - Use HttpOnly cookies for refresh tokens
// - Store short-lived access tokens in memory
// - Implement silent refresh before expiry
```

---

## Interview Questions & Answers

### Q1: What is change detection in Angular and how do you optimize it?

**Answer:**

Change detection is how Angular updates the DOM when data changes.

```typescript
// Default: Checks entire component tree
@Component({ ... })
export class DefaultComponent { }

// OnPush: Only checks when inputs change or events occur
@Component({
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class OptimizedComponent {
  @Input() data!: Data;
  
  // Manual trigger when needed
  constructor(private cdr: ChangeDetectorRef) {}
  
  forceUpdate() {
    this.cdr.markForCheck();  // Marks path to root
    // OR
    this.cdr.detectChanges(); // Runs detection immediately
  }
}
```

**Best practices:**
- Use `OnPush` for all presentational components
- Use immutable data patterns
- Use `trackBy` in `ngFor`
- Detach change detection for frequently updating data

---

### Q2: What is XSS and how does Angular protect against it?

**Answer:**

XSS (Cross-Site Scripting) allows attackers to inject malicious scripts.

```typescript
// Angular auto-sanitizes by default
@Component({
  template: `
    <!-- Safe: Angular sanitizes this -->
    <div>{{ userInput }}</div>
    <div [innerHTML]="htmlContent"></div>
    
    <!-- DANGEROUS: Bypassing security -->
    <div [innerHTML]="trustedHtml"></div>
  `
})
export class SecurityComponent {
  // Auto-sanitized
  userInput = '<script>alert("xss")</script>';
  htmlContent = '<img src=x onerror=alert(1)>';
  
  // Only when ABSOLUTELY necessary and content is trusted
  constructor(private sanitizer: DomSanitizer) {}
  
  get trustedHtml() {
    // CAREFUL: Only use for trusted content
    return this.sanitizer.bypassSecurityTrustHtml(this.htmlContent);
  }
}
```

**Angular protections:**
- Auto-sanitizes interpolation `{{ }}`
- Sanitizes property bindings `[innerHTML]`
- Sanitizes URL bindings `[href]`, `[src]`

---

### Q3: How do you implement lazy loading for performance?

**Answer:**

```typescript
// Route-level lazy loading
const routes: Routes = [
  {
    path: 'admin',
    loadComponent: () => import('./admin/admin.component')
      .then(m => m.AdminComponent)
  },
  {
    path: 'dashboard',
    loadChildren: () => import('./dashboard/routes')
      .then(m => m.DASHBOARD_ROUTES)
  }
];

// Component-level lazy loading with @defer
@Component({
  template: `
    @defer (on viewport) {
      <app-heavy-chart [data]="chartData"></app-heavy-chart>
    } @loading {
      <div class="skeleton"></div>
    } @placeholder {
      <div>Chart will load when visible</div>
    }
  `
})
export class DashboardComponent { }

// Preloading strategies
@NgModule({
  imports: [RouterModule.forRoot(routes, {
    preloadingStrategy: PreloadAllModules
  })]
})
```

---

### Q4: What is CSRF protection and how does Angular handle it?

**Answer:**

CSRF (Cross-Site Request Forgery) tricks users into performing unwanted actions.

```typescript
// Angular automatically reads XSRF token from cookie
// and includes it in requests

// Server sets cookie: XSRF-TOKEN=abc123
// Angular includes header: X-XSRF-TOKEN=abc123

// Configure custom header/cookie names
@NgModule({
  imports: [
    HttpClientModule,
    HttpClientXsrfModule.withOptions({
      cookieName: 'MY-CSRF-TOKEN',
      headerName: 'X-MY-CSRF-TOKEN'
    })
  ]
})
export class AppModule { }

// Manual token handling for non-standard setups
@Injectable()
export class CsrfInterceptor implements HttpInterceptor {
  intercept(req: HttpRequest<any>, next: HttpHandler) {
    const token = this.cookieService.get('csrf');
    if (token) {
      req = req.clone({
        setHeaders: { 'X-CSRF-Token': token }
      });
    }
    return next.handle(req);
  }
}
```

---

### Q5: How do you use trackBy to improve ngFor performance?

**Answer:**

```typescript
@Component({
  template: `
    <!-- Without trackBy: Entire list re-renders on change -->
    <div *ngFor="let item of items">{{ item.name }}</div>
    
    <!-- With trackBy: Only changed items re-render -->
    <div *ngFor="let item of items; trackBy: trackById">{{ item.name }}</div>
  `
})
export class ListComponent {
  items: Item[] = [];
  
  // Track by unique identifier
  trackById(index: number, item: Item): number {
    return item.id;
  }
  
  // Alternative: track by index (use when no unique ID)
  trackByIndex(index: number): number {
    return index;
  }
}
```

**Performance impact:**
- Without `trackBy`: O(n) DOM operations
- With `trackBy`: Only changed elements update

---

### Q6: What is Content Security Policy and how do you configure it?

**Answer:**

CSP prevents unauthorized script execution.

```html
<!-- Server response header -->
Content-Security-Policy: 
  default-src 'self';
  script-src 'self' 'unsafe-inline' https://apis.google.com;
  style-src 'self' 'unsafe-inline';
  img-src 'self' data: https:;
  connect-src 'self' https://api.example.com;
  font-src 'self' https://fonts.gstatic.com;
```

```typescript
// Angular CLI configuration for nonce-based CSP
// angular.json
{
  "projects": {
    "app": {
      "architect": {
        "build": {
          "options": {
            "outputHashing": "all",
            "crossOrigin": "anonymous"
          }
        }
      }
    }
  }
}

// Server-side nonce injection (express example)
app.use((req, res, next) => {
  res.locals.nonce = crypto.randomBytes(16).toString('base64');
  res.setHeader('Content-Security-Policy', 
    `script-src 'nonce-${res.locals.nonce}' 'strict-dynamic'`);
  next();
});
```

---

### Q7: How do you optimize bundle size in Angular?

**Answer:**

```bash
# Analyze bundle
ng build --stats-json
npx webpack-bundle-analyzer dist/app/stats.json

# Production build optimizations
ng build --configuration=production
```

```typescript
// Lazy load routes
loadComponent: () => import('./feature.component')

// Avoid barrel imports
// BAD
import { something } from '@shared';  // Imports entire barrel

// GOOD
import { something } from '@shared/utils/something';

// Use ES modules for libraries
// package.json
{
  "sideEffects": false
}

// Tree-shakeable providers
@Injectable({ providedIn: 'root' })  // Tree-shakeable
export class MyService { }

// Remove unused imports
import { map } from 'rxjs/operators';  // Good
import * as rxjs from 'rxjs';  // Bad - imports everything
```

---

### Q8: What are Signals and how do they improve performance?

**Answer:**

```typescript
@Component({
  template: `
    <p>Count: {{ count() }}</p>
    <p>Double: {{ doubled() }}</p>
    <button (click)="increment()">+</button>
  `
})
export class SignalComponent {
  // Writable signal
  count = signal(0);
  
  // Computed signal (automatically tracked)
  doubled = computed(() => this.count() * 2);
  
  // Effect (side effects on signal changes)
  constructor() {
    effect(() => {
      console.log('Count changed:', this.count());
    });
  }
  
  increment() {
    this.count.update(c => c + 1);
    // OR
    this.count.set(this.count() + 1);
  }
}

// Performance benefits:
// - Fine-grained reactivity (only affected parts update)
// - No zone.js dependency possible
// - Better memory efficiency than RxJS for simple state
```

---

### Q9: How do you prevent memory leaks in Angular?

**Answer:**

```typescript
@Component({...})
export class MemoryLeakFreeComponent implements OnDestroy {
  private destroy$ = new Subject<void>();
  
  // Method 1: takeUntil
  ngOnInit() {
    this.dataService.getData().pipe(
      takeUntil(this.destroy$)
    ).subscribe(data => this.data = data);
  }
  
  ngOnDestroy() {
    this.destroy$.next();
    this.destroy$.complete();
  }
}

// Method 2: DestroyRef (Angular 16+)
@Component({...})
export class ModernComponent {
  private destroyRef = inject(DestroyRef);
  
  ngOnInit() {
    this.dataService.getData().pipe(
      takeUntilDestroyed(this.destroyRef)
    ).subscribe();
  }
}

// Method 3: Async pipe (auto-unsubscribes)
@Component({
  template: `<div>{{ data$ | async }}</div>`
})
export class AsyncComponent {
  data$ = this.service.getData();
}

// Common leak sources:
// - Unsubscribed observables
// - Event listeners not removed
// - setInterval/setTimeout not cleared
// - DOM references in services
```

---

### Q10: How do you implement route guards for authorization?

**Answer:**

```typescript
// Functional guard
export const authGuard: CanActivateFn = (route, state) => {
  const auth = inject(AuthService);
  const router = inject(Router);
  
  if (!auth.isAuthenticated()) {
    return router.createUrlTree(['/login'], {
      queryParams: { returnUrl: state.url }
    });
  }
  
  // Role-based access
  const requiredRole = route.data['role'];
  if (requiredRole && !auth.hasRole(requiredRole)) {
    return router.createUrlTree(['/unauthorized']);
  }
  
  return true;
};

// Route configuration
const routes: Routes = [
  {
    path: 'admin',
    component: AdminComponent,
    canActivate: [authGuard],
    data: { role: 'admin' }
  },
  {
    path: 'dashboard',
    loadChildren: () => import('./dashboard/routes'),
    canMatch: [authGuard]  // Prevents even loading if unauthorized
  }
];
```

---

### Q11: What is zone.js and can you run Angular without it?

**Answer:**

```typescript
// Zone.js patches async APIs to trigger change detection
// Default behavior

// Zoneless Angular (experimental)
bootstrapApplication(AppComponent, {
  providers: [
    provideExperimentalZonelessChangeDetection()
  ]
});

// Manual change detection without zone.js
@Component({
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class ZonelessComponent {
  data = signal('');
  
  // Signals work without zone.js
  updateData() {
    this.data.set('new value');  // Auto-updates view
  }
  
  // For observables, manual trigger needed
  constructor(private cdr: ChangeDetectorRef) {
    someObservable$.subscribe(value => {
      this.value = value;
      this.cdr.markForCheck();
    });
  }
}
```

---

### Q12: How do you secure API calls in Angular?

**Answer:**

```typescript
// 1. Add authentication token
export const authInterceptor: HttpInterceptorFn = (req, next) => {
  const token = inject(AuthService).getToken();
  if (token) {
    req = req.clone({
      setHeaders: { Authorization: `Bearer ${token}` }
    });
  }
  return next(req);
};

// 2. Handle token expiration
export const tokenRefreshInterceptor: HttpInterceptorFn = (req, next) => {
  const auth = inject(AuthService);
  
  return next(req).pipe(
    catchError(error => {
      if (error.status === 401) {
        return auth.refreshToken().pipe(
          switchMap(newToken => {
            req = req.clone({
              setHeaders: { Authorization: `Bearer ${newToken}` }
            });
            return next(req);
          })
        );
      }
      return throwError(() => error);
    })
  );
};

// 3. Use HTTPS only
// environment.ts
export const environment = {
  apiUrl: 'https://api.example.com'  // Never http://
};

// 4. Validate responses
@Injectable({ providedIn: 'root' })
export class ApiService {
  getData(): Observable<Data> {
    return this.http.get<Data>('/api/data').pipe(
      map(response => this.validateResponse(response))
    );
  }
}
```

---

### Q13: How do you implement virtual scrolling for large lists?

**Answer:**

```typescript
import { CdkVirtualScrollViewport } from '@angular/cdk/scrolling';

@Component({
  template: `
    <cdk-virtual-scroll-viewport itemSize="50" class="viewport">
      <div *cdkVirtualFor="let item of items; trackBy: trackById" class="item">
        {{ item.name }}
      </div>
    </cdk-virtual-scroll-viewport>
  `,
  styles: [`
    .viewport {
      height: 400px;
      width: 100%;
    }
    .item {
      height: 50px;
    }
  `]
})
export class VirtualScrollComponent {
  items: Item[] = Array.from({ length: 10000 }, (_, i) => ({
    id: i,
    name: `Item ${i}`
  }));
  
  trackById(index: number, item: Item) {
    return item.id;
  }
}

// Dynamic item sizes
<cdk-virtual-scroll-viewport
  [itemSize]="50"
  [minBufferPx]="200"
  [maxBufferPx]="400">
```

---

### Q14: What are the Angular security best practices?

**Answer:**

| Practice | Description |
|----------|-------------|
| Never bypass sanitizer | Unless absolutely necessary for trusted content |
| Use HttpClient | Built-in XSRF protection |
| AOT compilation | Prevents template injection |
| HTTPS only | All API calls over TLS |
| CSP headers | Prevent unauthorized scripts |
| Validate inputs | Both client and server side |
| Avoid innerHTML | Use template binding instead |

```typescript
// Security checklist
@Component({...})
export class SecureComponent {
  // GOOD: Safe binding
  safeUrl = '/api/resource';
  
  // BAD: Never do this
  // [innerHTML]="userInput"
  // [href]="untrustedUrl"
  
  // Use pipes for display
  // {{ value | date }}
  
  // Sanitize if needed
  getSafeHtml(content: string) {
    // Only for trusted, server-validated content
    return this.sanitizer.bypassSecurityTrustHtml(content);
  }
}
```

---

### Q15: How do you optimize images and assets in Angular?

**Answer:**

```typescript
// NgOptimizedImage directive (Angular 15+)
import { NgOptimizedImage } from '@angular/common';

@Component({
  imports: [NgOptimizedImage],
  template: `
    <!-- Optimized image with lazy loading -->
    <img ngSrc="/assets/hero.jpg" 
         width="800" 
         height="600"
         priority
         placeholder="blur">
    
    <!-- Responsive images -->
    <img ngSrc="/assets/product.jpg"
         width="400" height="300"
         sizes="(max-width: 768px) 100vw, 50vw"
         [ngSrcset]="'320w, 640w, 960w'">
  `
})
export class ImageComponent { }

// Image loader for CDN
providers: [
  provideImgixLoader('https://my-site.imgix.net/')
]

// Lazy load images below fold
@Component({
  template: `
    @defer (on viewport) {
      <img ngSrc="/large-image.jpg" width="800" height="600">
    } @placeholder {
      <div class="image-placeholder"></div>
    }
  `
})
```

---
## Summary Checklist

âœ… **Change Detection**

- Understand default vs OnPush
- Use OnPush with immutable data
- Prefer async pipe

âœ… **Performance**

- Lazy load routes
- Use trackBy with ngFor
- Avoid expensive template expressions
- Virtual scrolling for long lists

âœ… **Bundle Optimization**

- Tree shaking
- Bundle budgets
- Analyze with webpack-bundle-analyzer

âœ… **Security**

- Trust Angular's sanitization
- Never bypass security with user input
- Use CSRF tokens
- Implement proper auth guards

âœ… **Memory Management**

- Always unsubscribe
- Use takeUntilDestroyed
- Prefer async pipe

---

**Key Takeaway:** Performance and security are ongoing concerns. Use OnPush change detection and lazy loading for performance. Trust Angular's built-in sanitization for security, and always validate on the server side. Memory leaks are common - use `takeUntilDestroyed` or async pipe consistently.
