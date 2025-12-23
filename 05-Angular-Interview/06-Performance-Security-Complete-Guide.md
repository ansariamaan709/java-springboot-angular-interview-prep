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
┌──────────────────────────────────────────────────────────────────────────┐
│                    ANGULAR CHANGE DETECTION                              │
├──────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  Zone.js patches async APIs (setTimeout, Promise, DOM events, etc.)     │
│                              │                                           │
│                              ▼                                           │
│                     ┌───────────────┐                                   │
│                     │ Async Event   │                                   │
│                     │ (click, http) │                                   │
│                     └───────┬───────┘                                   │
│                             │                                           │
│                             ▼                                           │
│                     ┌───────────────┐                                   │
│                     │   Zone.js     │                                   │
│                     │   Notifies    │                                   │
│                     └───────┬───────┘                                   │
│                             │                                           │
│                             ▼                                           │
│                     ┌───────────────┐                                   │
│                     │    Angular    │                                   │
│                     │    CD Run     │                                   │
│                     └───────┬───────┘                                   │
│                             │                                           │
│              ┌──────────────┼──────────────┐                           │
│              │              │              │                            │
│              ▼              ▼              ▼                            │
│        ┌──────────┐  ┌──────────┐  ┌──────────┐                        │
│        │Component │  │Component │  │Component │                        │
│        │    A     │  │    B     │  │    C     │                        │
│        └────┬─────┘  └────┬─────┘  └────┬─────┘                        │
│             │             │             │                               │
│             ▼             ▼             ▼                               │
│        Check & Update  Check & Update  Check & Update                  │
│             DOM            DOM            DOM                           │
│                                                                          │
│  DEFAULT Strategy: Checks ALL components top-to-bottom                  │
│  OnPush Strategy:  Checks ONLY when inputs change or events fire       │
│                                                                          │
└──────────────────────────────────────────────────────────────────────────┘
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
  selector: 'app-user-card',
  template: `
    <div class="card">
      <h3>{{ user.name }}</h3>
      <p>{{ user.email }}</p>
    </div>
  `,
  changeDetection: ChangeDetectionStrategy.OnPush  // Enable OnPush
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
  `
})
export class OnPushComponent {
  @Input() data!: string;
  
  constructor(private cdr: ChangeDetectorRef) {}
  
  // 1. Input reference changes (primitive or new object reference)
  // @Input() data changes → triggers CD
  
  // 2. DOM event within component
  onClick() {
    // This click triggers CD for this component
  }
  
  // 3. Async pipe emits
  // Observable | async → triggers CD
  
  // 4. Manual trigger
  manualTrigger() {
    this.cdr.markForCheck();  // Marks for next CD cycle
    this.cdr.detectChanges(); // Immediate CD
  }
}
```

### OnPush Gotchas

```typescript
// PROBLEM: Mutating objects doesn't trigger CD
@Component({
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    <p>{{ user.name }}</p>
  `
})
export class UserComponent {
  @Input() user!: User;
}

// Parent component
@Component({
  template: `<app-user [user]="currentUser"></app-user>`
})
export class ParentComponent {
  currentUser = { name: 'John' };
  
  // BAD - Mutation, won't trigger child CD
  updateNameBad() {
    this.currentUser.name = 'Jane';  // Same reference!
  }
  
  // GOOD - New reference
  updateNameGood() {
    this.currentUser = { ...this.currentUser, name: 'Jane' };
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
  { path: '', component: HomeComponent },
  
  // Lazy load entire feature
  {
    path: 'admin',
    loadChildren: () => import('./admin/admin.routes')
      .then(m => m.ADMIN_ROUTES)
  },
  
  // Lazy load single component
  {
    path: 'profile',
    loadComponent: () => import('./profile/profile.component')
      .then(m => m.ProfileComponent)
  }
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
  `
})
export class ReportComponent {
  @ViewChild('chartContainer', { read: ViewContainerRef }) 
  container!: ViewContainerRef;
  
  async loadChart() {
    // Dynamically import heavy library only when needed
    const { ChartComponent } = await import('./chart/chart.component');
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
import * as _ from 'lodash';
_.map(items, fn);

// GOOD - Import only what you need
import map from 'lodash-es/map';
map(items, fn);

// Or use subpath imports
import { map, filter } from 'lodash-es';
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
  `
})
export class ListBadComponent {
  items = [{ id: 1, name: 'A' }, { id: 2, name: 'B' }];
}

// WITH trackBy - Angular reuses DOM elements
@Component({
  template: `
    <ul>
      <li *ngFor="let item of items; trackBy: trackById">{{ item.name }}</li>
    </ul>
  `
})
export class ListGoodComponent {
  items = [{ id: 1, name: 'A' }, { id: 2, name: 'B' }];
  
  trackById(index: number, item: Item): number {
    return item.id;
  }
}
```

### Pure Pipes vs Impure Pipes

```typescript
// PURE PIPE (default) - Only re-runs when input reference changes
@Pipe({ name: 'filter', pure: true })
export class FilterPipe implements PipeTransform {
  transform(items: Item[], search: string): Item[] {
    return items.filter(item => item.name.includes(search));
  }
}

// IMPURE PIPE - Runs on EVERY change detection cycle (expensive!)
@Pipe({ name: 'filter', pure: false })
export class ImpureFilterPipe implements PipeTransform {
  transform(items: Item[], search: string): Item[] {
    return items.filter(item => item.name.includes(search));
  }
}

// BEST: Avoid impure pipes, use component methods or computed values
```

### Avoiding Expensive Template Expressions

```typescript
// BAD - Function called on every CD cycle
@Component({
  template: `
    <div>{{ calculateTotal() }}</div>  <!-- Called every CD! -->
  `
})
export class BadComponent {
  calculateTotal(): number {
    return this.items.reduce((sum, item) => sum + item.price, 0);
  }
}

// GOOD - Use memoization or computed property
@Component({
  template: `
    <div>{{ total }}</div>
  `
})
export class GoodComponent {
  total = 0;
  
  updateTotal() {
    this.total = this.items.reduce((sum, item) => sum + item.price, 0);
  }
}

// BEST with Signals (Angular 16+)
@Component({
  template: `<div>{{ total() }}</div>`
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
import { ScrollingModule } from '@angular/cdk/scrolling';

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
  styles: [`
    .viewport {
      height: 400px;
    }
    .item {
      height: 50px;
    }
  `]
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
  template: `<div>{{ heavyData }}</div>`
})
export class HeavyComponent implements OnInit, OnDestroy {
  heavyData = '';
  
  constructor(private cdr: ChangeDetectorRef) {}
  
  ngOnInit() {
    // Detach from CD tree
    this.cdr.detach();
    
    // Manually run CD only when needed
    this.loadData();
  }
  
  loadData() {
    this.heavyData = this.processHeavyData();
    this.cdr.detectChanges();  // Manual update
  }
}
```

---

## Security Best Practices

### Security Overview

```
┌──────────────────────────────────────────────────────────────────────────┐
│                    ANGULAR SECURITY LAYERS                               │
├──────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  CLIENT SIDE (Angular)                                                  │
│  ├── XSS Prevention (built-in sanitization)                            │
│  ├── Template security (no eval/innerHTML by default)                   │
│  ├── Route guards (authorization)                                       │
│  └── HttpOnly cookie handling                                          │
│                                                                          │
│  TRANSPORT                                                              │
│  ├── HTTPS only                                                         │
│  ├── CSRF tokens                                                        │
│  └── Secure headers (CSP, HSTS, etc.)                                  │
│                                                                          │
│  SERVER SIDE (API)                                                      │
│  ├── Authentication (JWT, OAuth)                                       │
│  ├── Authorization (roles, permissions)                                │
│  ├── Input validation                                                   │
│  └── Rate limiting                                                      │
│                                                                          │
└──────────────────────────────────────────────────────────────────────────┘
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
  `
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
import { DomSanitizer, SafeHtml, SafeUrl } from '@angular/platform-browser';

@Component({
  template: `
    <div [innerHTML]="trustedHtml"></div>
    <iframe [src]="trustedUrl"></iframe>
  `
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
      'https://trusted-domain.com/embed'
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
<meta http-equiv="Content-Security-Policy" content="
  default-src 'self';
  script-src 'self';
  style-src 'self' 'unsafe-inline';
  img-src 'self' data: https:;
  font-src 'self';
  connect-src 'self' https://api.example.com;
">
```

---

## CSRF Protection

### CSRF Token with Interceptor

```typescript
// CSRF interceptor
export const csrfInterceptor: HttpInterceptorFn = (req, next) => {
  // Get CSRF token from cookie (set by server)
  const csrfToken = getCookie('XSRF-TOKEN');
  
  if (csrfToken && !req.headers.has('X-XSRF-TOKEN')) {
    req = req.clone({
      setHeaders: { 'X-XSRF-TOKEN': csrfToken }
    });
  }
  
  return next(req);
};

function getCookie(name: string): string | null {
  const match = document.cookie.match(new RegExp('(^| )' + name + '=([^;]+)'));
  return match ? match[2] : null;
}

// Angular's built-in XSRF support
// HttpClientXsrfModule automatically reads XSRF-TOKEN cookie
// and adds X-XSRF-TOKEN header

// With provideHttpClient
provideHttpClient(withXsrfConfiguration({
  cookieName: 'XSRF-TOKEN',
  headerName: 'X-XSRF-TOKEN'
}));
```

---

## Authentication & Authorization

### JWT Authentication Service

```typescript
@Injectable({ providedIn: 'root' })
export class AuthService {
  private currentUserSubject = new BehaviorSubject<User | null>(null);
  currentUser$ = this.currentUserSubject.asObservable();
  
  private tokenKey = 'auth_token';
  
  constructor(
    private http: HttpClient,
    private router: Router
  ) {
    this.loadStoredUser();
  }
  
  login(credentials: LoginCredentials): Observable<AuthResponse> {
    return this.http.post<AuthResponse>('/api/auth/login', credentials).pipe(
      tap(response => {
        this.storeToken(response.token);
        this.currentUserSubject.next(response.user);
      })
    );
  }
  
  logout(): void {
    localStorage.removeItem(this.tokenKey);
    this.currentUserSubject.next(null);
    this.router.navigate(['/login']);
  }
  
  getToken(): string | null {
    return localStorage.getItem(this.tokenKey);
  }
  
  isAuthenticated(): boolean {
    const token = this.getToken();
    if (!token) return false;
    
    // Check if token is expired
    try {
      const payload = JSON.parse(atob(token.split('.')[1]));
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
      const payload = JSON.parse(atob(token.split('.')[1]));
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
  
  return router.createUrlTree(['/login'], {
    queryParams: { returnUrl: state.url }
  });
};

export const roleGuard: CanActivateFn = (route, state) => {
  const authService = inject(AuthService);
  const router = inject(Router);
  
  const requiredRoles = route.data['roles'] as string[];
  
  if (!authService.isAuthenticated()) {
    return router.createUrlTree(['/login']);
  }
  
  const hasRole = requiredRoles.some(role => authService.hasRole(role));
  
  if (hasRole) {
    return true;
  }
  
  return router.createUrlTree(['/unauthorized']);
};

// Route config
const routes: Routes = [
  {
    path: 'admin',
    canActivate: [authGuard, roleGuard],
    data: { roles: ['admin'] },
    loadChildren: () => import('./admin/admin.routes')
  }
];
```

### Token Refresh

```typescript
export const tokenRefreshInterceptor: HttpInterceptorFn = (req, next) => {
  const authService = inject(AuthService);
  
  // Skip for auth endpoints
  if (req.url.includes('/auth/')) {
    return next(req);
  }
  
  return next(req).pipe(
    catchError((error: HttpErrorResponse) => {
      if (error.status === 401) {
        return authService.refreshToken().pipe(
          switchMap(newToken => {
            const clonedReq = req.clone({
              setHeaders: { Authorization: `Bearer ${newToken}` }
            });
            return next(clonedReq);
          }),
          catchError(refreshError => {
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
localStorage.setItem('token', token);

// 2. sessionStorage - Vulnerable to XSS, cleared on tab close
sessionStorage.setItem('token', token);

// 3. HttpOnly Cookie - Not accessible via JS, CSRF protection needed
// Set by server:
// Set-Cookie: token=xxx; HttpOnly; Secure; SameSite=Strict

// 4. Memory only - Most secure but lost on refresh
@Injectable({ providedIn: 'root' })
export class TokenService {
  private token: string | null = null;
  
  setToken(token: string) { this.token = token; }
  getToken() { return this.token; }
  clearToken() { this.token = null; }
}

// Best practice for SPAs:
// - Use HttpOnly cookies for refresh tokens
// - Store short-lived access tokens in memory
// - Implement silent refresh before expiry
```

---

## Interview Questions & Answers

### Q1: What is Change Detection and how does OnPush improve performance?

**Answer:**

**Change Detection (CD)** is Angular's mechanism to keep the DOM in sync with component data.

**Default Strategy:**
- Runs for ALL components on every async event
- Top-down traversal of component tree
- Expensive for large apps

**OnPush Strategy:**
- Only runs when:
  1. Input reference changes
  2. Event fires within component
  3. Async pipe emits
  4. Manual trigger (`markForCheck()`)

```typescript
@Component({
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `{{ data | async }}`  // async pipe handles CD
})
export class OptimizedComponent {
  @Input() data!: Observable<Data>;
}
```

**Performance gain:** Can skip checking hundreds of components when only one needs update.

---

### Q2: How do you optimize Angular bundle size?

**Answer:**

**1. Lazy Loading:**
```typescript
{ path: 'admin', loadChildren: () => import('./admin/admin.routes') }
```

**2. Tree Shaking:**
```typescript
// Import only what you need
import { map } from 'lodash-es';  // Not: import * as _ from 'lodash'
```

**3. Production Build:**
```bash
ng build --configuration=production
```

**4. Bundle Budgets:**
```json
"budgets": [{ "type": "initial", "maximumWarning": "500kb" }]
```

**5. Analyze Bundles:**
```bash
ng build --stats-json
npx webpack-bundle-analyzer dist/stats.json
```

**6. Remove unused code:**
- Remove unused imports
- Use `providedIn: 'root'` for tree-shakeable services
- Avoid importing entire libraries

---

### Q3: What are common Angular security vulnerabilities and how do you prevent them?

**Answer:**

| Vulnerability | Prevention |
|--------------|------------|
| **XSS** | Angular auto-sanitizes; never use `bypassSecurityTrust*` with user input |
| **CSRF** | Use HttpOnly cookies + CSRF tokens |
| **Injection** | Validate inputs server-side |
| **Sensitive Data Exposure** | Don't store secrets in frontend code |
| **Broken Auth** | Use JWT properly, refresh tokens, secure storage |

```typescript
// XSS Prevention - Angular does this automatically
// NEVER do this with user input:
this.sanitizer.bypassSecurityTrustHtml(userInput); // DANGEROUS!

// CSRF Protection
provideHttpClient(withXsrfConfiguration({
  cookieName: 'XSRF-TOKEN',
  headerName: 'X-XSRF-TOKEN'
}));
```

---

### Q4: How do you prevent memory leaks in Angular?

**Answer:**

**Common Leak Sources:**
1. Unsubscribed Observables
2. Event listeners
3. Timers

**Solutions:**

```typescript
// 1. takeUntil pattern
private destroy$ = new Subject<void>();

ngOnInit() {
  source$.pipe(takeUntil(this.destroy$)).subscribe();
}

ngOnDestroy() {
  this.destroy$.next();
  this.destroy$.complete();
}

// 2. Angular 16+ takeUntilDestroyed
source$.pipe(takeUntilDestroyed()).subscribe();

// 3. Async pipe (auto-unsubscribes)
<div>{{ data$ | async }}</div>

// 4. Store and clear
private sub!: Subscription;
ngOnInit() { this.sub = source$.subscribe(); }
ngOnDestroy() { this.sub.unsubscribe(); }
```

---

### Q5: Explain trackBy and why it's important for ngFor.

**Answer:**

Without `trackBy`, Angular:
- Can't identify which items changed
- Destroys and recreates ALL DOM elements on any change
- Loses component state, causes flicker

With `trackBy`:
- Angular tracks items by unique identifier
- Only updates/creates/removes changed items
- Preserves DOM and component state

```typescript
// Template
<li *ngFor="let item of items; trackBy: trackById">{{ item.name }}</li>

// Component
trackById(index: number, item: Item): number {
  return item.id;  // Unique identifier
}
```

**Performance impact:**
- List of 1000 items: updating 1 item
- Without trackBy: 1000 DOM operations
- With trackBy: 1 DOM operation

---

## Summary Checklist

✅ **Change Detection**
- Understand default vs OnPush
- Use OnPush with immutable data
- Prefer async pipe

✅ **Performance**
- Lazy load routes
- Use trackBy with ngFor
- Avoid expensive template expressions
- Virtual scrolling for long lists

✅ **Bundle Optimization**
- Tree shaking
- Bundle budgets
- Analyze with webpack-bundle-analyzer

✅ **Security**
- Trust Angular's sanitization
- Never bypass security with user input
- Use CSRF tokens
- Implement proper auth guards

✅ **Memory Management**
- Always unsubscribe
- Use takeUntilDestroyed
- Prefer async pipe

---

**Key Takeaway:** Performance and security are ongoing concerns. Use OnPush change detection and lazy loading for performance. Trust Angular's built-in sanitization for security, and always validate on the server side. Memory leaks are common - use `takeUntilDestroyed` or async pipe consistently.
