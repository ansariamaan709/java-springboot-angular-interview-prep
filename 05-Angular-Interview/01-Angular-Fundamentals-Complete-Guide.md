# Angular Fundamentals - Complete Interview Guide

## Table of Contents

1. [Angular Architecture](#angular-architecture)
2. [Components Deep Dive](#components-deep-dive)
3. [Templates & Data Binding](#templates--data-binding)
4. [Directives](#directives)
5. [Pipes](#pipes)
6. [Services & Dependency Injection](#services--dependency-injection)
7. [Modules (NgModules)](#modules-ngmodules)
8. [Standalone Components](#standalone-components)
9. [Component Communication](#component-communication)
10. [Lifecycle Hooks](#lifecycle-hooks)
11. [Interview Questions & Answers](#interview-questions--answers)

---

## Angular Architecture

### Angular Application Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         ANGULAR APPLICATION                              │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │                         ROOT MODULE                              │    │
│  │                        (AppModule)                               │    │
│  │  ┌─────────────────────────────────────────────────────────┐    │    │
│  │  │                    FEATURE MODULES                       │    │    │
│  │  │   ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐   │    │    │
│  │  │   │  Auth   │  │  User   │  │  Order  │  │ Shared  │   │    │    │
│  │  │   │ Module  │  │ Module  │  │ Module  │  │ Module  │   │    │    │
│  │  │   └─────────┘  └─────────┘  └─────────┘  └─────────┘   │    │    │
│  │  └─────────────────────────────────────────────────────────┘    │    │
│  │                                                                  │    │
│  │  ┌─────────────────────────────────────────────────────────┐    │    │
│  │  │                     COMPONENTS                           │    │    │
│  │  │   ┌──────────┐  ┌──────────┐  ┌──────────┐             │    │    │
│  │  │   │ Template │  │   Class  │  │  Styles  │             │    │    │
│  │  │   │  (HTML)  │  │   (TS)   │  │  (CSS)   │             │    │    │
│  │  │   └──────────┘  └──────────┘  └──────────┘             │    │    │
│  │  └─────────────────────────────────────────────────────────┘    │    │
│  │                                                                  │    │
│  │  ┌─────────────────────────────────────────────────────────┐    │    │
│  │  │                     SERVICES                             │    │    │
│  │  │   HTTP Client │ State Management │ Business Logic        │    │    │
│  │  └─────────────────────────────────────────────────────────┘    │    │
│  └─────────────────────────────────────────────────────────────────┘    │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### Core Building Blocks

| Building Block  | Purpose                                        | Example                            |
| --------------- | ---------------------------------------------- | ---------------------------------- |
| **Component**   | UI building block with template, class, styles | `UserListComponent`                |
| **Template**    | HTML with Angular syntax                       | `<div *ngFor="let user of users">` |
| **Directive**   | Extend HTML behavior                           | `*ngIf`, `*ngFor`, `[ngClass]`     |
| **Pipe**        | Transform data in templates                    | `{{ date \| date:'short' }}`       |
| **Service**     | Reusable business logic                        | `UserService`, `AuthService`       |
| **Module**      | Organize related code                          | `UserModule`, `SharedModule`       |
| **Guard**       | Route protection                               | `AuthGuard`, `RoleGuard`           |
| **Interceptor** | HTTP request/response handling                 | `AuthInterceptor`                  |
| **Resolver**    | Pre-fetch data before route activation         | `UserResolver`                     |

---

## Components Deep Dive

### Component Structure

```typescript
// user.component.ts
import {
  Component,
  Input,
  Output,
  EventEmitter,
  OnInit,
  OnDestroy,
} from "@angular/core";
import { User } from "./user.model";

@Component({
  selector: "app-user",
  templateUrl: "./user.component.html",
  styleUrls: ["./user.component.scss"],
  // Or inline:
  // template: `<div>{{ user.name }}</div>`,
  // styles: [`.user { color: blue; }`]
})
export class UserComponent implements OnInit, OnDestroy {
  // Input properties (parent → child)
  @Input() user!: User;
  @Input() isEditable = false;

  // Output events (child → parent)
  @Output() userSelected = new EventEmitter<User>();
  @Output() userDeleted = new EventEmitter<number>();

  // Component state
  isExpanded = false;

  constructor() {
    console.log("Constructor called");
  }

  ngOnInit(): void {
    console.log("ngOnInit called - user:", this.user);
  }

  ngOnDestroy(): void {
    console.log("ngOnDestroy called - cleanup");
  }

  // Methods
  onSelect(): void {
    this.userSelected.emit(this.user);
  }

  onDelete(): void {
    this.userDeleted.emit(this.user.id);
  }

  toggleExpand(): void {
    this.isExpanded = !this.isExpanded;
  }
}
```

### Component Metadata Options

```typescript
@Component({
  // Required
  selector: 'app-user',  // HTML tag name

  // Template (one of these required)
  template: '<div>Inline template</div>',
  templateUrl: './user.component.html',

  // Styles (optional)
  styles: ['.user { color: blue; }'],
  styleUrls: ['./user.component.scss'],

  // View Encapsulation
  encapsulation: ViewEncapsulation.Emulated,  // Default - scoped styles
  // ViewEncapsulation.None - global styles
  // ViewEncapsulation.ShadowDom - native Shadow DOM

  // Change Detection Strategy
  changeDetection: ChangeDetectionStrategy.OnPush,  // Performance optimization
  // ChangeDetectionStrategy.Default - check on every cycle

  // Providers (component-level DI)
  providers: [UserService],  // New instance for this component tree

  // Host bindings
  host: {
    'class': 'user-component',
    '[class.active]': 'isActive',
    '(click)': 'onClick($event)'
  },

  // Animations
  animations: [fadeInAnimation],

  // Standalone (Angular 14+)
  standalone: true,
  imports: [CommonModule, FormsModule]
})
```

### Selector Types

```typescript
// Element selector (most common)
selector: "app-user";
// Usage: <app-user></app-user>

// Attribute selector
selector: "[appHighlight]";
// Usage: <div appHighlight></div>

// Class selector
selector: ".app-button";
// Usage: <div class="app-button"></div>

// Multiple selectors
selector: "app-user, [user-card]";
// Usage: <app-user> OR <div user-card>
```

---

## Templates & Data Binding

### Four Types of Data Binding

```
┌─────────────────────────────────────────────────────────────────┐
│                       DATA BINDING                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. INTERPOLATION (Component → View)                            │
│     {{ expression }}                                             │
│     <h1>{{ user.name }}</h1>                                    │
│                                                                  │
│  2. PROPERTY BINDING (Component → View)                         │
│     [property]="expression"                                      │
│     <img [src]="user.avatar">                                   │
│     <button [disabled]="isLoading">                             │
│                                                                  │
│  3. EVENT BINDING (View → Component)                            │
│     (event)="handler($event)"                                   │
│     <button (click)="save()">                                   │
│     <input (keyup.enter)="search()">                            │
│                                                                  │
│  4. TWO-WAY BINDING (Component ↔ View)                          │
│     [(ngModel)]="property"                                       │
│     <input [(ngModel)]="user.name">                             │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Interpolation Examples

```html
<!-- Basic interpolation -->
<h1>{{ title }}</h1>
<p>Welcome, {{ user.firstName + ' ' + user.lastName }}</p>

<!-- Method calls -->
<p>Full name: {{ getFullName() }}</p>

<!-- Expressions -->
<p>Total: {{ price * quantity }}</p>
<p>Status: {{ isActive ? 'Active' : 'Inactive' }}</p>

<!-- With pipes -->
<p>Date: {{ createdAt | date:'fullDate' }}</p>
<p>Price: {{ amount | currency:'USD' }}</p>

<!-- Safe navigation (null check) -->
<p>Address: {{ user?.address?.city }}</p>

<!-- Non-null assertion -->
<p>{{ user!.name }}</p>
```

### Property Binding Examples

```html
<!-- DOM property binding -->
<img [src]="imageUrl" />
<button [disabled]="isSubmitting">Submit</button>
<input [value]="username" />
<div [hidden]="!isVisible">Content</div>

<!-- Component input binding -->
<app-user [user]="selectedUser" [isEditable]="canEdit"></app-user>

<!-- Attribute binding (for non-DOM properties) -->
<button [attr.aria-label]="helpText">Help</button>
<table [attr.colspan]="columnSpan">
  <!-- Class binding -->
  <div [class.active]="isActive"></div>
  <div [class]="'btn ' + buttonType"></div>
  <div [ngClass]="{ 'active': isActive, 'disabled': isDisabled }"></div>

  <!-- Style binding -->
  <div [style.color]="textColor"></div>
  <div [style.font-size.px]="fontSize"></div>
  <div [ngStyle]="{ 'color': textColor, 'font-size': fontSize + 'px' }"></div>
</table>
```

### Event Binding Examples

```html
<!-- Click events -->
<button (click)="onClick()">Click me</button>
<button (click)="onClick($event)">With event</button>
<button (click)="save(); showMessage()">Multiple handlers</button>

<!-- Input events -->
<input (input)="onInput($event)" />
<input (change)="onChange($event)" />
<input (focus)="onFocus()" (blur)="onBlur()" />

<!-- Keyboard events -->
<input (keyup)="onKeyUp($event)" />
<input (keydown.enter)="onEnter()" />
<input (keydown.escape)="onEscape()" />
<input (keyup.control.shift.t)="onShortcut()" />

<!-- Form events -->
<form (ngSubmit)="onSubmit()">
  <select (selectionChange)="onSelect($event)">
    <!-- Custom events from child components -->
    <app-user (userSelected)="onUserSelected($event)"></app-user>
  </select>
</form>
```

### Two-Way Binding

```html
<!-- Requires FormsModule -->
<input [(ngModel)]="username" />

<!-- Expanded form (banana in a box) -->
<input [ngModel]="username" (ngModelChange)="username = $event" />

<!-- Custom two-way binding -->
<app-counter [(value)]="count"></app-counter>
```

```typescript
// Custom two-way binding in child component
@Component({
  selector: "app-counter",
  template: `
    <button (click)="decrement()">-</button>
    <span>{{ value }}</span>
    <button (click)="increment()">+</button>
  `,
})
export class CounterComponent {
  @Input() value = 0;
  @Output() valueChange = new EventEmitter<number>(); // Must be propertyName + 'Change'

  increment() {
    this.value++;
    this.valueChange.emit(this.value);
  }

  decrement() {
    this.value--;
    this.valueChange.emit(this.value);
  }
}
```

### Template Reference Variables

```html
<!-- Reference to DOM element -->
<input #emailInput type="email" />
<button (click)="validateEmail(emailInput.value)">Validate</button>

<!-- Reference to component -->
<app-user-form #userForm></app-user-form>
<button (click)="userForm.reset()">Reset Form</button>

<!-- Reference to directive -->
<form #myForm="ngForm">
  <button [disabled]="!myForm.valid">Submit</button>
</form>

<!-- Access in component with @ViewChild -->
<input #searchInput />
```

```typescript
@Component({ ... })
export class SearchComponent implements AfterViewInit {
  @ViewChild('searchInput') searchInput!: ElementRef<HTMLInputElement>;

  ngAfterViewInit() {
    this.searchInput.nativeElement.focus();
  }
}
```

---

## Directives

### Types of Directives

```
                      DIRECTIVES
                          │
        ┌─────────────────┼─────────────────┐
        │                 │                 │
   Component         Structural        Attribute
   Directives        Directives        Directives
        │                 │                 │
  (with template)    *ngIf              ngClass
                     *ngFor             ngStyle
                     *ngSwitch          [appHighlight]
```

### Structural Directives

```html
<!-- *ngIf -->
<div *ngIf="isLoggedIn">Welcome back!</div>

<!-- *ngIf with else -->
<div *ngIf="isLoggedIn; else loginTemplate">Welcome, {{ user.name }}!</div>
<ng-template #loginTemplate>
  <button (click)="login()">Login</button>
</ng-template>

<!-- *ngIf with then/else -->
<div *ngIf="isLoading; then loadingTpl; else contentTpl"></div>
<ng-template #loadingTpl>Loading...</ng-template>
<ng-template #contentTpl>{{ content }}</ng-template>

<!-- *ngIf with as (unwrap observable) -->
<div *ngIf="user$ | async as user">{{ user.name }}</div>

<!-- *ngFor -->
<ul>
  <li *ngFor="let item of items">{{ item.name }}</li>
</ul>

<!-- *ngFor with index and other variables -->
<div
  *ngFor="let item of items; 
             let i = index; 
             let first = first; 
             let last = last;
             let even = even;
             let odd = odd;
             trackBy: trackByFn"
>
  {{ i + 1 }}. {{ item.name }}
  <span *ngIf="first">(First)</span>
  <span *ngIf="last">(Last)</span>
</div>

<!-- trackBy for performance -->
trackByFn(index: number, item: User): number { return item.id; // Unique
identifier }

<!-- *ngSwitch -->
<div [ngSwitch]="status">
  <p *ngSwitchCase="'active'">User is active</p>
  <p *ngSwitchCase="'inactive'">User is inactive</p>
  <p *ngSwitchCase="'pending'">Awaiting approval</p>
  <p *ngSwitchDefault>Unknown status</p>
</div>
```

### Attribute Directives

```html
<!-- ngClass -->
<div [ngClass]="'active'">Single class</div>
<div [ngClass]="['btn', 'btn-primary']">Array of classes</div>
<div [ngClass]="{ 'active': isActive, 'disabled': isDisabled }">Object</div>
<div [ngClass]="getClasses()">Method returning object</div>

<!-- ngStyle -->
<div [ngStyle]="{ 'color': textColor, 'font-size.px': fontSize }">
  Styled text
</div>
<div [ngStyle]="getStyles()">Dynamic styles</div>
```

### Custom Attribute Directive

```typescript
// highlight.directive.ts
import { Directive, ElementRef, HostListener, Input, OnInit } from '@angular/core';

@Directive({
  selector: '[appHighlight]'
})
export class HighlightDirective implements OnInit {
  @Input() appHighlight = 'yellow';  // Default color
  @Input() highlightTextColor = 'black';

  constructor(private el: ElementRef) {}

  ngOnInit() {
    // Set initial style
    this.el.nativeElement.style.transition = 'background-color 0.3s';
  }

  @HostListener('mouseenter')
  onMouseEnter() {
    this.highlight(this.appHighlight);
  }

  @HostListener('mouseleave')
  onMouseLeave() {
    this.highlight('');
  }

  private highlight(color: string) {
    this.el.nativeElement.style.backgroundColor = color;
    this.el.nativeElement.style.color = color ? this.highlightTextColor : '';
  }
}

// Usage
<p appHighlight>Default yellow highlight</p>
<p [appHighlight]="'lightblue'">Blue highlight</p>
<p [appHighlight]="'pink'" highlightTextColor="white">Pink with white text</p>
```

### Custom Structural Directive

```typescript
// unless.directive.ts
import { Directive, Input, TemplateRef, ViewContainerRef } from '@angular/core';

@Directive({
  selector: '[appUnless]'
})
export class UnlessDirective {
  private hasView = false;

  constructor(
    private templateRef: TemplateRef<any>,
    private viewContainer: ViewContainerRef
  ) {}

  @Input() set appUnless(condition: boolean) {
    if (!condition && !this.hasView) {
      // Create view when condition is false
      this.viewContainer.createEmbeddedView(this.templateRef);
      this.hasView = true;
    } else if (condition && this.hasView) {
      // Remove view when condition is true
      this.viewContainer.clear();
      this.hasView = false;
    }
  }
}

// Usage (opposite of *ngIf)
<div *appUnless="isLoggedIn">Please login to continue</div>
```

---

## Pipes

### Built-in Pipes

```html
<!-- Date Pipe -->
{{ today | date }}
<!-- Jun 15, 2024 -->
{{ today | date:'short' }}
<!-- 6/15/24, 3:45 PM -->
{{ today | date:'fullDate' }}
<!-- Saturday, June 15, 2024 -->
{{ today | date:'yyyy-MM-dd' }}
<!-- 2024-06-15 -->
{{ today | date:'HH:mm:ss' }}
<!-- 15:45:30 -->

<!-- Number Pipes -->
{{ 3.14159 | number }}
<!-- 3.142 -->
{{ 3.14159 | number:'1.0-2' }}
<!-- 3.14 (min 1 int, 0-2 decimal) -->
{{ 0.5 | percent }}
<!-- 50% -->
{{ 0.5 | percent:'1.0-0' }}
<!-- 50% (no decimals) -->
{{ 1234.5 | currency }}
<!-- $1,234.50 -->
{{ 1234.5 | currency:'EUR' }}
<!-- €1,234.50 -->
{{ 1234.5 | currency:'INR':'symbol':'1.0-0' }}
<!-- ₹1,235 -->

<!-- String Pipes -->
{{ 'hello world' | uppercase }}
<!-- HELLO WORLD -->
{{ 'HELLO WORLD' | lowercase }}
<!-- hello world -->
{{ 'hello' | titlecase }}
<!-- Hello -->
{{ 'Angular is awesome' | slice:0:7 }}
<!-- Angular -->

<!-- JSON Pipe (debugging) -->
{{ user | json }}
<!-- {"name":"John","age":30} -->

<!-- Async Pipe (observables/promises) -->
{{ user$ | async }} {{ (users$ | async)?.length }}
<div *ngIf="data$ | async as data">{{ data.value }}</div>

<!-- KeyValue Pipe (iterate objects) -->
<div *ngFor="let item of object | keyvalue">
  {{ item.key }}: {{ item.value }}
</div>

<!-- I18nSelect / I18nPlural -->
{{ gender | i18nSelect: genderMap }} {{ count | i18nPlural: pluralMap }}
```

### Pipe Chaining

```html
<!-- Multiple pipes -->
{{ birthday | date:'fullDate' | uppercase }}
<!-- SATURDAY, JUNE 15, 2024 -->

{{ name | lowercase | titlecase }}
<!-- John Doe -->
```

### Custom Pipe

```typescript
// truncate.pipe.ts
import { Pipe, PipeTransform } from '@angular/core';

@Pipe({
  name: 'truncate',
  standalone: true  // Angular 14+
})
export class TruncatePipe implements PipeTransform {
  transform(value: string, limit = 25, ellipsis = '...'): string {
    if (!value) return '';
    if (value.length <= limit) return value;
    return value.substring(0, limit) + ellipsis;
  }
}

// Usage
{{ longText | truncate }}           <!-- First 25 chars... -->
{{ longText | truncate:50 }}        <!-- First 50 chars... -->
{{ longText | truncate:50:'→' }}    <!-- First 50 chars→ -->
```

### Pure vs Impure Pipes

```typescript
// Pure pipe (default) - called only when input reference changes
@Pipe({
  name: "filterUsers",
  pure: true, // Default
})
export class FilterUsersPipe implements PipeTransform {
  transform(users: User[], searchTerm: string): User[] {
    return users.filter((u) => u.name.includes(searchTerm));
  }
}
// Won't update if array is mutated (users.push(newUser))
// Only updates if new array reference: users = [...users, newUser]

// Impure pipe - called on every change detection cycle
@Pipe({
  name: "filterUsersImpure",
  pure: false, // Called frequently - use sparingly!
})
export class FilterUsersImpurePipe implements PipeTransform {
  transform(users: User[], searchTerm: string): User[] {
    return users.filter((u) => u.name.includes(searchTerm));
  }
}
// Updates even when array is mutated
// Performance impact - avoid for expensive operations
```

---

## Services & Dependency Injection

### Creating Services

```typescript
// user.service.ts
import { Injectable } from "@angular/core";
import { HttpClient } from "@angular/common/http";
import { Observable, BehaviorSubject } from "rxjs";
import { map, tap } from "rxjs/operators";

@Injectable({
  providedIn: "root", // Singleton - available throughout app
})
export class UserService {
  private apiUrl = "/api/users";
  private currentUserSubject = new BehaviorSubject<User | null>(null);

  currentUser$ = this.currentUserSubject.asObservable();

  constructor(private http: HttpClient) {}

  getUsers(): Observable<User[]> {
    return this.http.get<User[]>(this.apiUrl);
  }

  getUser(id: number): Observable<User> {
    return this.http.get<User>(`${this.apiUrl}/${id}`);
  }

  createUser(user: Partial<User>): Observable<User> {
    return this.http.post<User>(this.apiUrl, user);
  }

  updateUser(id: number, user: Partial<User>): Observable<User> {
    return this.http.put<User>(`${this.apiUrl}/${id}`, user);
  }

  deleteUser(id: number): Observable<void> {
    return this.http.delete<void>(`${this.apiUrl}/${id}`);
  }

  setCurrentUser(user: User): void {
    this.currentUserSubject.next(user);
  }

  logout(): void {
    this.currentUserSubject.next(null);
  }
}
```

### Dependency Injection Hierarchy

```
                    Root Injector
                  (providedIn: 'root')
                         │
        ┌────────────────┼────────────────┐
        │                │                │
   Module Injector  Module Injector  Module Injector
    (providers:[])   (providers:[])   (providers:[])
        │                                 │
   Component           Component     Component
   Injector            Injector      Injector
  (providers:[])      (providers:[]) (providers:[])
        │
   Child Component
      Injector
```

### Provider Types

```typescript
// 1. Class Provider (default)
providers: [UserService];
// Same as:
providers: [{ provide: UserService, useClass: UserService }];

// 2. Alternative Class
providers: [
  { provide: UserService, useClass: MockUserService }, // For testing
];

// 3. Value Provider
providers: [{ provide: "API_URL", useValue: "https://api.example.com" }];

// 4. Factory Provider
providers: [
  {
    provide: UserService,
    useFactory: (http: HttpClient, config: AppConfig) => {
      return config.useMock ? new MockUserService() : new UserService(http);
    },
    deps: [HttpClient, AppConfig],
  },
];

// 5. Existing Provider (alias)
providers: [
  UserService,
  { provide: "UserServiceAlias", useExisting: UserService },
];
```

### Injection Tokens

```typescript
// tokens.ts
import { InjectionToken } from "@angular/core";

export const API_URL = new InjectionToken<string>("API_URL");
export const APP_CONFIG = new InjectionToken<AppConfig>("APP_CONFIG");

export interface AppConfig {
  apiUrl: string;
  production: boolean;
  features: string[];
}

// app.module.ts
providers: [
  { provide: API_URL, useValue: "https://api.example.com" },
  {
    provide: APP_CONFIG,
    useValue: {
      apiUrl: "https://api.example.com",
      production: true,
      features: ["feature1", "feature2"],
    },
  },
];

// service.ts
@Injectable()
export class ApiService {
  constructor(
    @Inject(API_URL) private apiUrl: string,
    @Inject(APP_CONFIG) private config: AppConfig
  ) {}
}
```

### Injection Decorators

```typescript
@Injectable()
export class ChildService {
  constructor(
    // Normal injection
    private userService: UserService,

    // Optional (won't throw if not found)
    @Optional() private logService: LogService | null,

    // Self (only look in current injector)
    @Self() private localService: LocalService,

    // SkipSelf (skip current, look in parent)
    @SkipSelf() private parentService: ParentService,

    // Host (stop at host component)
    @Host() private hostService: HostService,

    // Inject by token
    @Inject(API_URL) private apiUrl: string
  ) {}
}
```

---

## Modules (NgModules)

### Module Structure

```typescript
// user.module.ts
import { NgModule } from "@angular/core";
import { CommonModule } from "@angular/common";
import { FormsModule, ReactiveFormsModule } from "@angular/forms";
import { HttpClientModule } from "@angular/common/http";

import { UserRoutingModule } from "./user-routing.module";
import { UserListComponent } from "./user-list/user-list.component";
import { UserDetailComponent } from "./user-detail/user-detail.component";
import { UserFormComponent } from "./user-form/user-form.component";
import { UserService } from "./user.service";

@NgModule({
  declarations: [
    // Components, directives, pipes owned by this module
    UserListComponent,
    UserDetailComponent,
    UserFormComponent,
  ],
  imports: [
    // Other modules needed by this module
    CommonModule,
    FormsModule,
    ReactiveFormsModule,
    HttpClientModule,
    UserRoutingModule,
  ],
  exports: [
    // Components/directives/pipes available to importing modules
    UserListComponent,
  ],
  providers: [
    // Services (prefer providedIn: 'root' instead)
    UserService,
  ],
})
export class UserModule {}
```

### Module Types

| Module Type        | Purpose                   | Example                     |
| ------------------ | ------------------------- | --------------------------- |
| **Root Module**    | Bootstrap app             | `AppModule`                 |
| **Feature Module** | Organize features         | `UserModule`, `OrderModule` |
| **Shared Module**  | Reusable components/pipes | `SharedModule`              |
| **Core Module**    | Singleton services        | `CoreModule`                |
| **Routing Module** | Route configuration       | `AppRoutingModule`          |

### Shared Module Pattern

```typescript
// shared.module.ts
@NgModule({
  declarations: [
    // Reusable components
    LoadingSpinnerComponent,
    ConfirmDialogComponent,
    PaginationComponent,

    // Reusable directives
    HighlightDirective,

    // Reusable pipes
    TruncatePipe,
    TimeAgoPipe,
  ],
  imports: [CommonModule, FormsModule, ReactiveFormsModule],
  exports: [
    // Re-export common modules
    CommonModule,
    FormsModule,
    ReactiveFormsModule,

    // Export declarations
    LoadingSpinnerComponent,
    ConfirmDialogComponent,
    PaginationComponent,
    HighlightDirective,
    TruncatePipe,
    TimeAgoPipe,
  ],
})
export class SharedModule {}
```

### Core Module Pattern

```typescript
// core.module.ts
@NgModule({
  providers: [
    // Singleton services
    AuthService,
    LoggingService,
    ErrorHandlerService,
  ],
})
export class CoreModule {
  // Prevent re-importing (ensures singleton)
  constructor(@Optional() @SkipSelf() parentModule: CoreModule) {
    if (parentModule) {
      throw new Error(
        "CoreModule is already loaded. Import it only in AppModule."
      );
    }
  }
}
```

---

## Standalone Components

### Angular 14+ Standalone Components

```typescript
// user.component.ts
import { Component } from "@angular/core";
import { CommonModule } from "@angular/common";
import { RouterModule } from "@angular/router";
import { UserService } from "./user.service";
import { UserCardComponent } from "./user-card.component";

@Component({
  selector: "app-user-list",
  standalone: true, // No NgModule needed!
  imports: [
    CommonModule,
    RouterModule,
    UserCardComponent, // Import other standalone components
  ],
  template: `
    <div *ngFor="let user of users$ | async">
      <app-user-card [user]="user"></app-user-card>
    </div>
  `,
})
export class UserListComponent {
  users$ = inject(UserService).getUsers();
}

// Bootstrapping standalone app (main.ts)
import { bootstrapApplication } from "@angular/platform-browser";
import { AppComponent } from "./app/app.component";
import { provideRouter } from "@angular/router";
import { provideHttpClient } from "@angular/common/http";

bootstrapApplication(AppComponent, {
  providers: [provideRouter(routes), provideHttpClient()],
});
```

### inject() Function (Angular 14+)

```typescript
// Traditional constructor injection
@Component({ ... })
export class UserComponent {
  constructor(
    private userService: UserService,
    private router: Router
  ) {}
}

// Modern inject() function
@Component({ ... })
export class UserComponent {
  private userService = inject(UserService);
  private router = inject(Router);

  // Can also use in field initializers
  users$ = inject(UserService).getUsers();
}
```

---

## Component Communication

### 1. Parent to Child (@Input)

```typescript
// Parent component
@Component({
  template: `<app-child [user]="selectedUser" [config]="settings"></app-child>`,
})
export class ParentComponent {
  selectedUser: User = { id: 1, name: "John" };
  settings = { showAvatar: true, editable: false };
}

// Child component
@Component({
  selector: "app-child",
  template: `<div>{{ user.name }}</div>`,
})
export class ChildComponent implements OnChanges {
  @Input() user!: User;
  @Input() config!: Config;

  // React to input changes
  ngOnChanges(changes: SimpleChanges) {
    if (changes["user"]) {
      console.log("User changed:", changes["user"].currentValue);
    }
  }

  // Or use setter
  private _user!: User;
  @Input()
  set user(value: User) {
    this._user = value;
    this.processUser();
  }
  get user(): User {
    return this._user;
  }
}
```

### 2. Child to Parent (@Output)

```typescript
// Child component
@Component({
  selector: "app-child",
  template: `
    <button (click)="onSave()">Save</button>
    <button (click)="onDelete()">Delete</button>
  `,
})
export class ChildComponent {
  @Output() saved = new EventEmitter<User>();
  @Output() deleted = new EventEmitter<number>();

  onSave() {
    this.saved.emit({ id: 1, name: "John" });
  }

  onDelete() {
    this.deleted.emit(1);
  }
}

// Parent component
@Component({
  template: `
    <app-child (saved)="onUserSaved($event)" (deleted)="onUserDeleted($event)">
    </app-child>
  `,
})
export class ParentComponent {
  onUserSaved(user: User) {
    console.log("User saved:", user);
  }

  onUserDeleted(userId: number) {
    console.log("User deleted:", userId);
  }
}
```

### 3. ViewChild / ViewChildren

```typescript
@Component({
  template: `
    <app-child #child1></app-child>
    <app-child #child2></app-child>
    <input #nameInput />
  `,
})
export class ParentComponent implements AfterViewInit {
  @ViewChild("child1") firstChild!: ChildComponent;
  @ViewChild(ChildComponent) anyChild!: ChildComponent;
  @ViewChildren(ChildComponent) allChildren!: QueryList<ChildComponent>;
  @ViewChild("nameInput") inputElement!: ElementRef<HTMLInputElement>;

  ngAfterViewInit() {
    // Access child component methods/properties
    this.firstChild.doSomething();

    // Access all children
    this.allChildren.forEach((child) => child.reset());

    // Access DOM element
    this.inputElement.nativeElement.focus();
  }
}
```

### 4. ContentChild / ContentChildren (Projected Content)

```typescript
// Parent template (content projection)
<app-card>
  <app-card-header>Title</app-card-header>
  <app-card-body>Content here</app-card-body>
</app-card>;

// Card component
@Component({
  selector: "app-card",
  template: `
    <div class="card">
      <ng-content select="app-card-header"></ng-content>
      <ng-content select="app-card-body"></ng-content>
    </div>
  `,
})
export class CardComponent implements AfterContentInit {
  @ContentChild(CardHeaderComponent) header!: CardHeaderComponent;
  @ContentChildren(CardBodyComponent) bodies!: QueryList<CardBodyComponent>;

  ngAfterContentInit() {
    console.log("Header:", this.header);
    console.log("Bodies:", this.bodies.length);
  }
}
```

### 5. Service with Subject (Unrelated Components)

```typescript
// notification.service.ts
@Injectable({ providedIn: 'root' })
export class NotificationService {
  private messageSubject = new Subject<string>();
  message$ = this.messageSubject.asObservable();

  showMessage(message: string) {
    this.messageSubject.next(message);
  }
}

// Component A (sender)
@Component({ ... })
export class ComponentA {
  constructor(private notificationService: NotificationService) {}

  sendNotification() {
    this.notificationService.showMessage('Hello from A!');
  }
}

// Component B (receiver)
@Component({ ... })
export class ComponentB implements OnInit, OnDestroy {
  private destroy$ = new Subject<void>();

  constructor(private notificationService: NotificationService) {}

  ngOnInit() {
    this.notificationService.message$
      .pipe(takeUntil(this.destroy$))
      .subscribe(message => {
        console.log('Received:', message);
      });
  }

  ngOnDestroy() {
    this.destroy$.next();
    this.destroy$.complete();
  }
}
```

---

## Lifecycle Hooks

### Lifecycle Hooks Order

```
Constructor
    │
    ▼
ngOnChanges ←────────────── Called when @Input changes
    │
    ▼
ngOnInit ←─────────────────── Called once after first ngOnChanges
    │
    ▼
ngDoCheck ←────────────────── Called during every change detection
    │
    ▼
ngAfterContentInit ←───────── Called after content projection (ng-content)
    │
    ▼
ngAfterContentChecked ←────── Called after every content check
    │
    ▼
ngAfterViewInit ←──────────── Called after view (and child views) initialized
    │
    ▼
ngAfterViewChecked ←───────── Called after every view check
    │
    ▼
ngOnDestroy ←──────────────── Called before component is destroyed
```

### Lifecycle Hooks Example

```typescript
@Component({
  selector: "app-lifecycle-demo",
  template: `<p>{{ data }}</p>`,
})
export class LifecycleDemoComponent
  implements
    OnInit,
    OnChanges,
    DoCheck,
    AfterContentInit,
    AfterContentChecked,
    AfterViewInit,
    AfterViewChecked,
    OnDestroy
{
  @Input() data!: string;

  constructor() {
    // DON'T: Access @Input values here (not yet set)
    // DON'T: Make HTTP calls here
    // DO: Simple initialization
    console.log("1. Constructor");
  }

  ngOnChanges(changes: SimpleChanges) {
    // Called before ngOnInit and whenever @Input changes
    console.log("2. ngOnChanges", changes);
    if (changes["data"]) {
      const prev = changes["data"].previousValue;
      const curr = changes["data"].currentValue;
      console.log(`Data changed from ${prev} to ${curr}`);
    }
  }

  ngOnInit() {
    // Called once after first ngOnChanges
    // DO: Fetch initial data, setup subscriptions
    console.log("3. ngOnInit");
  }

  ngDoCheck() {
    // Called during every change detection run
    // Use for custom change detection logic
    console.log("4. ngDoCheck");
  }

  ngAfterContentInit() {
    // Called after ng-content projection is complete
    // @ContentChild queries are available here
    console.log("5. ngAfterContentInit");
  }

  ngAfterContentChecked() {
    // Called after every content check
    console.log("6. ngAfterContentChecked");
  }

  ngAfterViewInit() {
    // Called after component's view (and child views) initialized
    // @ViewChild queries are available here
    console.log("7. ngAfterViewInit");
  }

  ngAfterViewChecked() {
    // Called after every view check
    console.log("8. ngAfterViewChecked");
  }

  ngOnDestroy() {
    // Called before component is destroyed
    // DO: Cleanup subscriptions, timers, event listeners
    console.log("9. ngOnDestroy");
  }
}
```

---

## Interview Questions & Answers

### Q1: What's the difference between `ngOnInit` and `constructor`?

**Answer:**

| Aspect               | Constructor         | ngOnInit                 |
| -------------------- | ------------------- | ------------------------ |
| **Purpose**          | Class instantiation | Component initialization |
| **@Input available** | No                  | Yes                      |
| **Called**           | By JavaScript       | By Angular               |
| **HTTP calls**       | Avoid               | Recommended              |
| **Subscriptions**    | Avoid               | Recommended              |

```typescript
@Component({...})
export class UserComponent implements OnInit {
  @Input() userId!: number;

  constructor(private userService: UserService) {
    // userId is undefined here!
    console.log(this.userId);  // undefined
  }

  ngOnInit() {
    // userId is available here
    console.log(this.userId);  // actual value
    this.userService.getUser(this.userId).subscribe(...);
  }
}
```

---

### Q2: Explain change detection strategies in Angular.

**Answer:**

**Default Strategy:**

- Checks entire component tree on every event
- Simple but can be slow for large apps

**OnPush Strategy:**

- Only checks when:
  1. `@Input` reference changes
  2. Event originates from component or child
  3. Async pipe emits new value
  4. Manual `markForCheck()` or `detectChanges()`

```typescript
@Component({
  selector: "app-user",
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `{{ user.name }}`,
})
export class UserComponent {
  @Input() user!: User;

  constructor(private cdr: ChangeDetectorRef) {}

  // For external data updates
  externalUpdate() {
    // Mutating won't trigger OnPush
    this.user.name = "New Name"; // Won't update view!

    // Option 1: New reference
    this.user = { ...this.user, name: "New Name" };

    // Option 2: Manual trigger
    this.cdr.markForCheck();

    // Option 3: Detach/Reattach
    this.cdr.detectChanges();
  }
}
```

---

### Q3: What are the differences between template-driven and reactive forms?

**Answer:**

| Aspect              | Template-Driven | Reactive            |
| ------------------- | --------------- | ------------------- |
| **Setup**           | FormsModule     | ReactiveFormsModule |
| **Form definition** | In template     | In component class  |
| **Data model**      | Two-way binding | Immutable           |
| **Validation**      | Directives      | Functions           |
| **Testing**         | Harder          | Easier              |
| **Dynamic forms**   | Difficult       | Easy                |

```typescript
// Template-driven
<form #form="ngForm" (ngSubmit)="onSubmit(form)">
  <input name="email" ngModel required email>
  <div *ngIf="form.controls.email?.invalid">Invalid email</div>
</form>

// Reactive
@Component({...})
export class UserFormComponent {
  form = new FormGroup({
    email: new FormControl('', [Validators.required, Validators.email])
  });

  onSubmit() {
    if (this.form.valid) {
      console.log(this.form.value);
    }
  }
}

<form [formGroup]="form" (ngSubmit)="onSubmit()">
  <input formControlName="email">
  <div *ngIf="form.get('email')?.invalid">Invalid email</div>
</form>
```

---

### Q4: How does dependency injection work in Angular?

**Answer:**

Angular's DI creates and manages instances of dependencies:

1. **Registration**: Define providers (what to create)
2. **Resolution**: Angular finds provider in injector hierarchy
3. **Injection**: Instance is injected into constructor

```typescript
// 1. Registration
@Injectable({
  providedIn: 'root'  // Root injector - singleton
})
export class UserService {}

// Or in module
@NgModule({
  providers: [UserService]  // Module injector
})

// Or in component
@Component({
  providers: [UserService]  // Component injector - new instance per component
})

// 2 & 3. Resolution and Injection
@Component({...})
export class UserComponent {
  constructor(private userService: UserService) {
    // Angular looks up injector hierarchy
    // Component → Parent → Module → Root → Platform
  }
}
```

---

### Q5: What is content projection in Angular?

**Answer:**

Content projection (transclusion) allows passing content into a component:

```typescript
// card.component.ts
@Component({
  selector: "app-card",
  template: `
    <div class="card">
      <div class="header">
        <ng-content select="[card-header]"></ng-content>
      </div>
      <div class="body">
        <ng-content></ng-content>
      </div>
      <div class="footer">
        <ng-content select="[card-footer]"></ng-content>
      </div>
    </div>
  `,
})
export class CardComponent {}

// Usage
<app-card>
  <h2 card-header>Card Title</h2>
  <p>This is the card body content.</p>
  <p>More content here.</p>
  <button card-footer>Action</button>
</app-card>;
```

**Multi-slot projection selectors:**

- `select="app-header"` - By component
- `select="[card-header]"` - By attribute
- `select=".header-class"` - By class
- `select="h2"` - By element
- No selector - Default slot for unmatched content

---

### Q6: What's the difference between `ViewChild` and `ContentChild`?

**Answer:**

| Aspect           | ViewChild                | ContentChild                   |
| ---------------- | ------------------------ | ------------------------------ |
| **Queries**      | Component's own template | Projected content (ng-content) |
| **Available in** | ngAfterViewInit          | ngAfterContentInit             |
| **Defined in**   | Component's template     | Parent's template              |

```typescript
// Parent template
<app-card>
  <app-item #projectedItem></app-item>  <!-- ContentChild -->
</app-card>

// Card component template
<div class="card">
  <ng-content></ng-content>
  <app-footer #viewItem></app-footer>  <!-- ViewChild -->
</div>

// Card component class
@Component({...})
export class CardComponent implements AfterViewInit, AfterContentInit {
  @ViewChild('viewItem') viewItem!: FooterComponent;
  @ContentChild('projectedItem') contentItem!: ItemComponent;

  ngAfterContentInit() {
    console.log(this.contentItem);  // Available here
  }

  ngAfterViewInit() {
    console.log(this.viewItem);  // Available here
  }
}
```

---

## Summary: Angular Fundamentals Checklist

✅ **Components**

- Understand component metadata options
- Know all four types of data binding
- Use appropriate lifecycle hooks

✅ **Templates**

- Master interpolation and property binding
- Use template reference variables
- Understand structural vs attribute directives

✅ **Services & DI**

- Use `providedIn: 'root'` for singletons
- Understand injector hierarchy
- Know when to use InjectionToken

✅ **Modules**

- Create feature, shared, and core modules
- Understand lazy loading benefits
- Consider standalone components (Angular 14+)

✅ **Communication**

- @Input/@Output for parent-child
- Services with Subjects for unrelated components
- ViewChild/ContentChild for direct access

---

**Key Takeaway:** Angular's architecture promotes modularity and testability. Understanding the component lifecycle, change detection, and dependency injection is crucial for building performant Angular applications.
