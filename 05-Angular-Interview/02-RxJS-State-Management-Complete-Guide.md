# Angular RxJS & State Management - Complete Interview Guide

## Table of Contents

1. [RxJS Fundamentals](#rxjs-fundamentals)
2. [Observables vs Promises](#observables-vs-promises)
3. [Essential RxJS Operators](#essential-rxjs-operators)
4. [Subjects & BehaviorSubjects](#subjects--behaviorsubjects)
5. [Higher-Order Observables](#higher-order-observables)
6. [Error Handling](#error-handling)
7. [Memory Leaks & Unsubscription](#memory-leaks--unsubscription)
8. [State Management Patterns](#state-management-patterns)
9. [NgRx Store](#ngrx-store)
10. [Angular Signals (17+)](#angular-signals-17)
11. [Interview Questions & Answers](#interview-questions--answers)

---

## RxJS Fundamentals

### What is RxJS?

RxJS (Reactive Extensions for JavaScript) is a library for reactive programming using **Observables**, making it easier to compose asynchronous or callback-based code.

### Core Concepts

```
┌─────────────────────────────────────────────────────────────────────┐
│                         RxJS CORE CONCEPTS                          │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  OBSERVABLE ────────────────────────────────────────────────────    │
│  │  A stream of values over time                                    │
│  │  Can emit: next(value), error(err), complete()                   │
│  │                                                                  │
│  │  ┌─────────────────────────────────────────────────────┐        │
│  │  │  ──●──●──●──●──●──●──●──●──|                        │        │
│  │  │     values over time      complete                  │        │
│  │  └─────────────────────────────────────────────────────┘        │
│                                                                      │
│  OBSERVER ──────────────────────────────────────────────────────    │
│  │  Consumes values from Observable                                 │
│  │  { next: fn, error: fn, complete: fn }                          │
│                                                                      │
│  SUBSCRIPTION ──────────────────────────────────────────────────    │
│  │  Represents execution of Observable                              │
│  │  Used to cancel/unsubscribe                                     │
│                                                                      │
│  OPERATORS ─────────────────────────────────────────────────────    │
│  │  Transform, filter, combine streams                              │
│  │  map, filter, mergeMap, switchMap, etc.                         │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Creating Observables

```typescript
import { Observable, of, from, interval, timer, fromEvent } from 'rxjs';

// 1. Create from scratch
const custom$ = new Observable<number>(subscriber => {
  subscriber.next(1);
  subscriber.next(2);
  subscriber.next(3);
  
  setTimeout(() => {
    subscriber.next(4);
    subscriber.complete();
  }, 1000);
  
  // Cleanup function (called on unsubscribe)
  return () => {
    console.log('Cleanup');
  };
});

// 2. From static values
const values$ = of(1, 2, 3, 4, 5);
const array$ = from([1, 2, 3, 4, 5]);
const promise$ = from(fetch('/api/users'));

// 3. Timer-based
const interval$ = interval(1000);  // Emits 0, 1, 2, ... every second
const timer$ = timer(2000, 1000);  // Wait 2s, then emit every 1s
const delayed$ = timer(3000);      // Single value after 3s

// 4. From DOM events
const clicks$ = fromEvent(document, 'click');
const inputs$ = fromEvent(inputElement, 'input');

// 5. From HTTP (Angular)
const users$ = this.http.get<User[]>('/api/users');
```

### Subscribing to Observables

```typescript
// Full observer object
const subscription = source$.subscribe({
  next: (value) => console.log('Value:', value),
  error: (err) => console.error('Error:', err),
  complete: () => console.log('Complete')
});

// Shorthand (just next handler)
source$.subscribe(value => console.log(value));

// Unsubscribe
subscription.unsubscribe();
```

---

## Observables vs Promises

### Key Differences

| Feature | Promise | Observable |
|---------|---------|------------|
| **Values** | Single value | Multiple values over time |
| **Execution** | Eager (immediate) | Lazy (on subscribe) |
| **Cancellation** | Not cancellable | Cancellable via unsubscribe |
| **Operators** | .then(), .catch() | Many (map, filter, etc.) |
| **Multicasting** | Shared | Unicast by default |

### Code Comparison

```typescript
// PROMISE - Executes immediately
const promise = new Promise(resolve => {
  console.log('Promise executing');
  resolve('data');
});
// "Promise executing" logged immediately

// OBSERVABLE - Executes on subscribe (lazy)
const observable$ = new Observable(subscriber => {
  console.log('Observable executing');
  subscriber.next('data');
});
// Nothing logged yet

observable$.subscribe();  // NOW it logs

// PROMISE - Single value
const fetchUser = (): Promise<User> => {
  return this.http.get<User>('/api/user').toPromise();
};

// OBSERVABLE - Multiple values
const userUpdates$ = interval(5000).pipe(
  switchMap(() => this.http.get<User>('/api/user'))
);
// Emits new user data every 5 seconds

// PROMISE - Not cancellable
const promise = fetch('/api/data');
// Can't cancel!

// OBSERVABLE - Cancellable
const subscription = this.http.get('/api/data').subscribe();
subscription.unsubscribe();  // Request cancelled
```

---

## Essential RxJS Operators

### Transformation Operators

```typescript
import { map, pluck, mapTo, scan, reduce } from 'rxjs/operators';

// map - Transform each value
const doubled$ = of(1, 2, 3).pipe(
  map(x => x * 2)
);
// Output: 2, 4, 6

// pluck - Extract property (deprecated, use map)
const names$ = users$.pipe(
  map(user => user.name)
);

// scan - Running accumulator (like reduce but emits each step)
const runningTotal$ = of(1, 2, 3, 4, 5).pipe(
  scan((acc, val) => acc + val, 0)
);
// Output: 1, 3, 6, 10, 15

// reduce - Final accumulated value only
const total$ = of(1, 2, 3, 4, 5).pipe(
  reduce((acc, val) => acc + val, 0)
);
// Output: 15 (only final value)
```

### Filtering Operators

```typescript
import { filter, first, last, take, skip, distinctUntilChanged, debounceTime, throttleTime } from 'rxjs/operators';

// filter - Keep values matching predicate
const evens$ = of(1, 2, 3, 4, 5).pipe(
  filter(x => x % 2 === 0)
);
// Output: 2, 4

// first / last / take / skip
const first$ = source$.pipe(first());           // First value only
const last$ = source$.pipe(last());             // Last value only
const firstThree$ = source$.pipe(take(3));      // First 3 values
const skipTwo$ = source$.pipe(skip(2));         // Skip first 2

// distinctUntilChanged - Ignore consecutive duplicates
const distinct$ = of(1, 1, 2, 2, 3, 1).pipe(
  distinctUntilChanged()
);
// Output: 1, 2, 3, 1

// For objects - provide comparator
const users$ = userUpdates$.pipe(
  distinctUntilChanged((prev, curr) => prev.id === curr.id)
);

// debounceTime - Wait for pause in emissions
const searchTerm$ = searchInput$.pipe(
  debounceTime(300)  // Wait 300ms after last emission
);

// throttleTime - Emit first, ignore for duration
const scrollEvents$ = scroll$.pipe(
  throttleTime(100)  // Max one event per 100ms
);
```

### Combination Operators

```typescript
import { merge, concat, combineLatest, forkJoin, zip, withLatestFrom } from 'rxjs';
import { startWith } from 'rxjs/operators';

// merge - Combine multiple streams into one
const allClicks$ = merge(button1Clicks$, button2Clicks$, button3Clicks$);

// concat - Emit in order (waits for completion)
const sequence$ = concat(
  of(1, 2, 3),
  of(4, 5, 6)
);
// Output: 1, 2, 3, 4, 5, 6

// combineLatest - Latest from each when any emits
const combined$ = combineLatest([
  firstName$,
  lastName$
]).pipe(
  map(([first, last]) => `${first} ${last}`)
);

// forkJoin - Wait for all to complete, emit final values
const allData$ = forkJoin({
  users: this.http.get<User[]>('/api/users'),
  products: this.http.get<Product[]>('/api/products'),
  orders: this.http.get<Order[]>('/api/orders')
});
// Emits: { users: [...], products: [...], orders: [...] }

// zip - Pair values by index
const paired$ = zip(letters$, numbers$);
// Input: ['a', 'b', 'c'], [1, 2, 3]
// Output: ['a', 1], ['b', 2], ['c', 3]

// withLatestFrom - Combine with latest from other stream
const saveWithUser$ = saveButton$.pipe(
  withLatestFrom(currentUser$),
  map(([_, user]) => user)
);
```

### Utility Operators

```typescript
import { tap, delay, timeout, retry, catchError, finalize } from 'rxjs/operators';

// tap - Side effects without modifying stream
const logged$ = source$.pipe(
  tap(value => console.log('Before:', value)),
  map(x => x * 2),
  tap(value => console.log('After:', value))
);

// delay - Delay emissions
const delayed$ = source$.pipe(delay(2000));

// timeout - Error if no emission within duration
const withTimeout$ = this.http.get('/api/data').pipe(
  timeout(5000)  // Error after 5 seconds
);

// retry - Retry on error
const resilient$ = this.http.get('/api/data').pipe(
  retry(3)  // Retry up to 3 times
);

// finalize - Cleanup on complete or error
const withCleanup$ = source$.pipe(
  finalize(() => this.isLoading = false)
);
```

---

## Subjects & BehaviorSubjects

### Subject Types Comparison

```
┌────────────────────────────────────────────────────────────────────────┐
│                          SUBJECT TYPES                                  │
├────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  Subject            ──●──●──●──●──●──                                  │
│  (Hot)               ↑ New subscriber misses previous values           │
│                                                                         │
│  BehaviorSubject    [initial]──●──●──●──●──                            │
│  (Has current)       ↑ New subscriber gets last value immediately      │
│                                                                         │
│  ReplaySubject      ──●──●──●──●──●──                                  │
│  (Replays N)         ↑ New subscriber gets last N values               │
│                                                                         │
│  AsyncSubject       ──●──●──●──●──●──|                                 │
│  (Last on complete)  ↑ Only emits last value on complete               │
│                                                                         │
└────────────────────────────────────────────────────────────────────────┘
```

### Subject

```typescript
import { Subject } from 'rxjs';

const subject = new Subject<number>();

// Subscriber A
subject.subscribe(val => console.log('A:', val));

subject.next(1);  // A: 1
subject.next(2);  // A: 2

// Subscriber B (joins late - misses 1, 2)
subject.subscribe(val => console.log('B:', val));

subject.next(3);  // A: 3, B: 3 (both receive)
```

### BehaviorSubject

```typescript
import { BehaviorSubject } from 'rxjs';

// Requires initial value
const behaviorSubject = new BehaviorSubject<number>(0);

// Subscriber A
behaviorSubject.subscribe(val => console.log('A:', val));
// A: 0 (initial value immediately)

behaviorSubject.next(1);  // A: 1
behaviorSubject.next(2);  // A: 2

// Subscriber B (gets current value immediately)
behaviorSubject.subscribe(val => console.log('B:', val));
// B: 2 (current value)

behaviorSubject.next(3);  // A: 3, B: 3

// Access current value synchronously
console.log(behaviorSubject.getValue());  // 3
// Or use .value property
console.log(behaviorSubject.value);  // 3
```

### ReplaySubject

```typescript
import { ReplaySubject } from 'rxjs';

// Replay last 2 values
const replaySubject = new ReplaySubject<number>(2);

replaySubject.next(1);
replaySubject.next(2);
replaySubject.next(3);

// New subscriber gets last 2 values
replaySubject.subscribe(val => console.log('A:', val));
// A: 2
// A: 3

replaySubject.next(4);  // A: 4

// ReplaySubject with time window
const timedReplay = new ReplaySubject<number>(100, 500);  // 100 values OR 500ms
```

### Practical Service Example

```typescript
@Injectable({ providedIn: 'root' })
export class UserStateService {
  // BehaviorSubject for current user (null = not logged in)
  private currentUserSubject = new BehaviorSubject<User | null>(null);
  
  // Expose as observable (read-only)
  currentUser$ = this.currentUserSubject.asObservable();
  
  // Subject for notifications (no need for replay)
  private notificationSubject = new Subject<Notification>();
  notification$ = this.notificationSubject.asObservable();
  
  // Derived state
  isLoggedIn$ = this.currentUser$.pipe(
    map(user => !!user)
  );
  
  setUser(user: User): void {
    this.currentUserSubject.next(user);
  }
  
  logout(): void {
    this.currentUserSubject.next(null);
    this.notify({ type: 'info', message: 'Logged out successfully' });
  }
  
  notify(notification: Notification): void {
    this.notificationSubject.next(notification);
  }
  
  // Synchronous access when needed
  get currentUser(): User | null {
    return this.currentUserSubject.value;
  }
}
```

---

## Higher-Order Observables

### switchMap vs mergeMap vs concatMap vs exhaustMap

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    HIGHER-ORDER MAPPING OPERATORS                        │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  switchMap  ─────────────────────────────────────────────────────────   │
│  │  Cancels previous inner observable when new value arrives            │
│  │  Best for: search/autocomplete, route changes                        │
│  │                                                                      │
│  │  Source:  ──1────────2────────3──                                   │
│  │  Inner:      ──a──b      ──c──d      ──e──f──                       │
│  │  Output:     ──a──       ──c──       ──e──f──                       │
│  │                    ↑ cancelled   ↑ cancelled                         │
│                                                                          │
│  mergeMap  ──────────────────────────────────────────────────────────   │
│  │  Runs all inner observables concurrently                             │
│  │  Best for: parallel requests, save operations                        │
│  │                                                                      │
│  │  Source:  ──1────────2────────3──                                   │
│  │  Inner:      ──a──b────  ──c──d────  ──e──f──                       │
│  │  Output:     ──a──b──c──d──e──f──  (interleaved)                    │
│                                                                          │
│  concatMap  ─────────────────────────────────────────────────────────   │
│  │  Queues and runs inner observables sequentially                      │
│  │  Best for: order matters, writes that must not overlap               │
│  │                                                                      │
│  │  Source:  ──1────2────3──                                           │
│  │  Inner:      ──a──b──|──c──d──|──e──f──                             │
│  │  Output:     ──a──b────c──d────e──f──  (in order)                   │
│                                                                          │
│  exhaustMap  ────────────────────────────────────────────────────────   │
│  │  Ignores new values while inner observable is active                 │
│  │  Best for: form submissions, button clicks                           │
│  │                                                                      │
│  │  Source:  ──1────2────3────────4──                                  │
│  │  Inner:      ──a──b──|            ──c──d──                          │
│  │  Output:     ──a──b──             ──c──d──                          │
│  │                   ↑ 2,3 ignored (1 still active)                     │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### Practical Examples

```typescript
import { switchMap, mergeMap, concatMap, exhaustMap } from 'rxjs/operators';

// switchMap - Search (cancel previous request on new input)
searchInput$.pipe(
  debounceTime(300),
  distinctUntilChanged(),
  switchMap(term => this.searchService.search(term))
).subscribe(results => this.results = results);

// mergeMap - Load details for multiple items in parallel
items$.pipe(
  mergeMap(item => this.loadDetails(item.id), 5)  // Max 5 concurrent
).subscribe(details => console.log(details));

// concatMap - Save items in order
saveQueue$.pipe(
  concatMap(item => this.saveItem(item))
).subscribe(saved => console.log('Saved:', saved));

// exhaustMap - Form submit (ignore clicks while submitting)
submitButton$.pipe(
  exhaustMap(() => this.submitForm())
).subscribe(response => this.handleResponse(response));
```

---

## Error Handling

### Error Handling Operators

```typescript
import { catchError, retry, retryWhen, throwError, EMPTY } from 'rxjs';
import { delay } from 'rxjs/operators';

// catchError - Handle and recover
const safe$ = this.http.get('/api/data').pipe(
  catchError(error => {
    console.error('Error:', error);
    // Return fallback value
    return of({ data: [], error: true });
    // Or re-throw
    // return throwError(() => new Error('Custom error'));
    // Or complete silently
    // return EMPTY;
  })
);

// retry - Simple retry
const resilient$ = this.http.get('/api/data').pipe(
  retry(3)  // Retry 3 times immediately
);

// retryWhen - Retry with delay
const withBackoff$ = this.http.get('/api/data').pipe(
  retryWhen(errors => errors.pipe(
    delay(1000),  // Wait 1 second between retries
    take(3)       // Max 3 retries
  ))
);

// Exponential backoff
const exponentialRetry$ = this.http.get('/api/data').pipe(
  retryWhen(errors => errors.pipe(
    scan((retryCount, error) => {
      if (retryCount >= 3) throw error;
      return retryCount + 1;
    }, 0),
    delay(attempt => Math.pow(2, attempt) * 1000)  // 1s, 2s, 4s
  ))
);

// Multiple error handling
const robust$ = this.http.get('/api/data').pipe(
  timeout(5000),
  retry(2),
  catchError(error => {
    if (error.status === 404) {
      return of(null);  // Return null for not found
    }
    if (error.status === 401) {
      this.authService.logout();
      return EMPTY;
    }
    return throwError(() => error);  // Re-throw other errors
  })
);
```

### Error Handling in Components

```typescript
@Component({...})
export class UserListComponent implements OnInit {
  users$ = this.userService.getUsers().pipe(
    catchError(error => {
      this.errorMessage = 'Failed to load users';
      return of([]);  // Return empty array
    })
  );
  
  errorMessage: string | null = null;
  
  // Or with loading state
  loadingState$: Observable<LoadingState<User[]>>;
  
  ngOnInit() {
    this.loadingState$ = this.userService.getUsers().pipe(
      map(users => ({ loading: false, data: users, error: null })),
      startWith({ loading: true, data: null, error: null }),
      catchError(error => of({ loading: false, data: null, error: error.message }))
    );
  }
}

interface LoadingState<T> {
  loading: boolean;
  data: T | null;
  error: string | null;
}
```

---

## Memory Leaks & Unsubscription

### The Problem

```typescript
// MEMORY LEAK - Subscription never cleaned up
@Component({...})
export class BadComponent implements OnInit {
  ngOnInit() {
    interval(1000).subscribe(val => {
      console.log(val);  // Keeps running after component destroyed!
    });
  }
}
```

### Solutions

#### 1. Manual Unsubscription

```typescript
@Component({...})
export class ManualComponent implements OnInit, OnDestroy {
  private subscription!: Subscription;
  
  ngOnInit() {
    this.subscription = interval(1000).subscribe(val => {
      console.log(val);
    });
  }
  
  ngOnDestroy() {
    this.subscription.unsubscribe();
  }
}

// Multiple subscriptions
@Component({...})
export class MultipleComponent implements OnInit, OnDestroy {
  private subscriptions = new Subscription();
  
  ngOnInit() {
    this.subscriptions.add(
      source1$.subscribe(...)
    );
    this.subscriptions.add(
      source2$.subscribe(...)
    );
  }
  
  ngOnDestroy() {
    this.subscriptions.unsubscribe();  // Unsubscribes all
  }
}
```

#### 2. takeUntil Pattern (Recommended)

```typescript
@Component({...})
export class TakeUntilComponent implements OnInit, OnDestroy {
  private destroy$ = new Subject<void>();
  
  ngOnInit() {
    interval(1000).pipe(
      takeUntil(this.destroy$)
    ).subscribe(val => console.log(val));
    
    this.userService.getUser().pipe(
      takeUntil(this.destroy$)
    ).subscribe(user => this.user = user);
  }
  
  ngOnDestroy() {
    this.destroy$.next();
    this.destroy$.complete();
  }
}
```

#### 3. Async Pipe (Best for Templates)

```typescript
@Component({
  template: `
    <div *ngIf="user$ | async as user">
      {{ user.name }}
    </div>
    
    <ul>
      <li *ngFor="let item of items$ | async">{{ item }}</li>
    </ul>
  `
})
export class AsyncPipeComponent {
  user$ = this.userService.getUser();
  items$ = this.itemService.getItems();
  
  // No need to unsubscribe - async pipe handles it!
}
```

#### 4. take / first Operators

```typescript
// For one-time requests
this.http.get('/api/data').pipe(
  take(1)  // Automatically completes after first value
).subscribe(data => console.log(data));

// Or use first()
this.http.get('/api/data').pipe(
  first()
).subscribe(data => console.log(data));
```

#### 5. DestroyRef (Angular 16+)

```typescript
import { DestroyRef, inject } from '@angular/core';
import { takeUntilDestroyed } from '@angular/core/rxjs-interop';

@Component({...})
export class ModernComponent implements OnInit {
  private destroyRef = inject(DestroyRef);
  
  ngOnInit() {
    interval(1000).pipe(
      takeUntilDestroyed(this.destroyRef)
    ).subscribe(val => console.log(val));
  }
}

// Or even simpler in field initializers
@Component({...})
export class SimplerComponent {
  // Automatically unsubscribes when component is destroyed
  user$ = inject(UserService).getUser().pipe(
    takeUntilDestroyed()
  );
}
```

---

## State Management Patterns

### Service with BehaviorSubject

```typescript
// state.model.ts
export interface AppState {
  users: User[];
  selectedUser: User | null;
  loading: boolean;
  error: string | null;
}

// state.service.ts
@Injectable({ providedIn: 'root' })
export class StateService {
  private state = new BehaviorSubject<AppState>({
    users: [],
    selectedUser: null,
    loading: false,
    error: null
  });

  // Selectors (derived state)
  state$ = this.state.asObservable();
  users$ = this.state$.pipe(map(s => s.users), distinctUntilChanged());
  selectedUser$ = this.state$.pipe(map(s => s.selectedUser), distinctUntilChanged());
  loading$ = this.state$.pipe(map(s => s.loading), distinctUntilChanged());
  
  constructor(private http: HttpClient) {}
  
  // Actions
  loadUsers(): void {
    this.updateState({ loading: true, error: null });
    
    this.http.get<User[]>('/api/users').pipe(
      tap(users => this.updateState({ users, loading: false })),
      catchError(error => {
        this.updateState({ loading: false, error: error.message });
        return EMPTY;
      })
    ).subscribe();
  }
  
  selectUser(user: User | null): void {
    this.updateState({ selectedUser: user });
  }
  
  addUser(user: User): void {
    const users = [...this.state.value.users, user];
    this.updateState({ users });
  }
  
  updateUser(updated: User): void {
    const users = this.state.value.users.map(u => 
      u.id === updated.id ? updated : u
    );
    this.updateState({ users });
  }
  
  deleteUser(id: number): void {
    const users = this.state.value.users.filter(u => u.id !== id);
    this.updateState({ users });
  }
  
  // Private helper
  private updateState(partial: Partial<AppState>): void {
    this.state.next({ ...this.state.value, ...partial });
  }
}

// Component usage
@Component({
  template: `
    <div *ngIf="loading$ | async">Loading...</div>
    <div *ngIf="error$ | async as error">{{ error }}</div>
    
    <ul>
      <li *ngFor="let user of users$ | async" 
          (click)="selectUser(user)"
          [class.selected]="(selectedUser$ | async)?.id === user.id">
        {{ user.name }}
      </li>
    </ul>
  `
})
export class UserListComponent implements OnInit {
  users$ = this.stateService.users$;
  selectedUser$ = this.stateService.selectedUser$;
  loading$ = this.stateService.loading$;
  error$ = this.stateService.state$.pipe(map(s => s.error));
  
  constructor(private stateService: StateService) {}
  
  ngOnInit() {
    this.stateService.loadUsers();
  }
  
  selectUser(user: User) {
    this.stateService.selectUser(user);
  }
}
```

### Component Store (NgRx)

```typescript
// user.store.ts
import { Injectable } from '@angular/core';
import { ComponentStore } from '@ngrx/component-store';

interface UserState {
  users: User[];
  loading: boolean;
  error: string | null;
}

@Injectable()
export class UserStore extends ComponentStore<UserState> {
  constructor(private userService: UserService) {
    super({ users: [], loading: false, error: null });
  }
  
  // Selectors
  readonly users$ = this.select(state => state.users);
  readonly loading$ = this.select(state => state.loading);
  readonly vm$ = this.select({
    users: this.users$,
    loading: this.loading$
  });
  
  // Updaters (synchronous)
  readonly setLoading = this.updater((state, loading: boolean) => ({
    ...state,
    loading
  }));
  
  readonly setUsers = this.updater((state, users: User[]) => ({
    ...state,
    users,
    loading: false
  }));
  
  readonly addUser = this.updater((state, user: User) => ({
    ...state,
    users: [...state.users, user]
  }));
  
  // Effects (asynchronous)
  readonly loadUsers = this.effect<void>(trigger$ =>
    trigger$.pipe(
      tap(() => this.setLoading(true)),
      switchMap(() => this.userService.getUsers().pipe(
        tapResponse(
          users => this.setUsers(users),
          error => this.patchState({ loading: false, error: error.message })
        )
      ))
    )
  );
}

// Component
@Component({
  providers: [UserStore],  // Component-scoped
  template: `
    <ng-container *ngIf="vm$ | async as vm">
      <div *ngIf="vm.loading">Loading...</div>
      <ul>
        <li *ngFor="let user of vm.users">{{ user.name }}</li>
      </ul>
    </ng-container>
  `
})
export class UserListComponent implements OnInit {
  vm$ = this.store.vm$;
  
  constructor(private store: UserStore) {}
  
  ngOnInit() {
    this.store.loadUsers();
  }
}
```

---

## NgRx Store

### NgRx Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                           NgRx ARCHITECTURE                              │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│                        ┌─────────────────┐                              │
│                        │    COMPONENT    │                              │
│                        │    (View)       │                              │
│                        └────────┬────────┘                              │
│                                 │                                        │
│              ┌──────────────────┼──────────────────┐                    │
│              │ dispatch(action) │ select(selector) │                    │
│              ▼                  │                  │                    │
│     ┌────────────────┐          │         ┌────────────────┐            │
│     │    ACTIONS     │          │         │   SELECTORS    │            │
│     │ (Events)       │          │         │ (Queries)      │            │
│     └────────┬───────┘          │         └────────▲───────┘            │
│              │                  │                  │                    │
│              ▼                  │                  │                    │
│     ┌────────────────┐          │         ┌────────────────┐            │
│     │   REDUCERS     │──────────┴────────▶│     STORE      │            │
│     │ (State changes)│                    │   (State)      │            │
│     └────────────────┘                    └────────────────┘            │
│              │                                                          │
│              ▼                                                          │
│     ┌────────────────┐                                                  │
│     │   EFFECTS      │                                                  │
│     │ (Side effects) │                                                  │
│     └────────────────┘                                                  │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### Complete NgRx Example

```typescript
// user.actions.ts
import { createAction, props } from '@ngrx/store';

export const loadUsers = createAction('[User] Load Users');
export const loadUsersSuccess = createAction(
  '[User] Load Users Success',
  props<{ users: User[] }>()
);
export const loadUsersFailure = createAction(
  '[User] Load Users Failure',
  props<{ error: string }>()
);
export const selectUser = createAction(
  '[User] Select User',
  props<{ userId: number }>()
);
export const addUser = createAction(
  '[User] Add User',
  props<{ user: User }>()
);
export const deleteUser = createAction(
  '[User] Delete User',
  props<{ userId: number }>()
);

// user.reducer.ts
import { createReducer, on } from '@ngrx/store';

export interface UserState {
  users: User[];
  selectedUserId: number | null;
  loading: boolean;
  error: string | null;
}

export const initialState: UserState = {
  users: [],
  selectedUserId: null,
  loading: false,
  error: null
};

export const userReducer = createReducer(
  initialState,
  
  on(loadUsers, state => ({
    ...state,
    loading: true,
    error: null
  })),
  
  on(loadUsersSuccess, (state, { users }) => ({
    ...state,
    users,
    loading: false
  })),
  
  on(loadUsersFailure, (state, { error }) => ({
    ...state,
    loading: false,
    error
  })),
  
  on(selectUser, (state, { userId }) => ({
    ...state,
    selectedUserId: userId
  })),
  
  on(addUser, (state, { user }) => ({
    ...state,
    users: [...state.users, user]
  })),
  
  on(deleteUser, (state, { userId }) => ({
    ...state,
    users: state.users.filter(u => u.id !== userId)
  }))
);

// user.selectors.ts
import { createFeatureSelector, createSelector } from '@ngrx/store';

export const selectUserState = createFeatureSelector<UserState>('users');

export const selectAllUsers = createSelector(
  selectUserState,
  state => state.users
);

export const selectUsersLoading = createSelector(
  selectUserState,
  state => state.loading
);

export const selectUsersError = createSelector(
  selectUserState,
  state => state.error
);

export const selectSelectedUserId = createSelector(
  selectUserState,
  state => state.selectedUserId
);

export const selectSelectedUser = createSelector(
  selectAllUsers,
  selectSelectedUserId,
  (users, selectedId) => users.find(u => u.id === selectedId) || null
);

// Parameterized selector
export const selectUserById = (userId: number) => createSelector(
  selectAllUsers,
  users => users.find(u => u.id === userId)
);

// user.effects.ts
import { Injectable } from '@angular/core';
import { Actions, createEffect, ofType } from '@ngrx/effects';

@Injectable()
export class UserEffects {
  
  loadUsers$ = createEffect(() =>
    this.actions$.pipe(
      ofType(loadUsers),
      switchMap(() =>
        this.userService.getUsers().pipe(
          map(users => loadUsersSuccess({ users })),
          catchError(error => of(loadUsersFailure({ error: error.message })))
        )
      )
    )
  );
  
  // Effect that doesn't dispatch action
  logActions$ = createEffect(() =>
    this.actions$.pipe(
      tap(action => console.log('Action:', action.type))
    ),
    { dispatch: false }
  );
  
  constructor(
    private actions$: Actions,
    private userService: UserService
  ) {}
}

// app.module.ts
@NgModule({
  imports: [
    StoreModule.forRoot({ users: userReducer }),
    EffectsModule.forRoot([UserEffects]),
    StoreDevtoolsModule.instrument({ maxAge: 25 })  // Dev tools
  ]
})
export class AppModule {}

// Component
@Component({
  template: `
    <div *ngIf="loading$ | async">Loading...</div>
    <div *ngIf="error$ | async as error" class="error">{{ error }}</div>
    
    <ul>
      <li *ngFor="let user of users$ | async"
          [class.selected]="user.id === (selectedUserId$ | async)"
          (click)="onSelectUser(user.id)">
        {{ user.name }}
        <button (click)="onDeleteUser(user.id)">Delete</button>
      </li>
    </ul>
  `
})
export class UserListComponent implements OnInit {
  users$ = this.store.select(selectAllUsers);
  loading$ = this.store.select(selectUsersLoading);
  error$ = this.store.select(selectUsersError);
  selectedUserId$ = this.store.select(selectSelectedUserId);
  
  constructor(private store: Store) {}
  
  ngOnInit() {
    this.store.dispatch(loadUsers());
  }
  
  onSelectUser(userId: number) {
    this.store.dispatch(selectUser({ userId }));
  }
  
  onDeleteUser(userId: number) {
    this.store.dispatch(deleteUser({ userId }));
  }
}
```

---

## Angular Signals (17+)

### What are Signals?

Signals are a new reactive primitive in Angular 16+ that provide:
- **Synchronous** access to values
- **Fine-grained** change detection
- **Simple** API compared to RxJS for many cases

```typescript
import { signal, computed, effect } from '@angular/core';

// Create a signal
const count = signal(0);

// Read value
console.log(count());  // 0

// Update value
count.set(5);
count.update(n => n + 1);

// Computed signals (derived state)
const doubled = computed(() => count() * 2);
console.log(doubled());  // 12

// Effects (side effects)
effect(() => {
  console.log('Count changed:', count());
});
```

### Signals in Components

```typescript
@Component({
  selector: 'app-counter',
  standalone: true,
  template: `
    <h2>Count: {{ count() }}</h2>
    <p>Doubled: {{ doubled() }}</p>
    
    <button (click)="increment()">+</button>
    <button (click)="decrement()">-</button>
    <button (click)="reset()">Reset</button>
  `
})
export class CounterComponent {
  // Writable signal
  count = signal(0);
  
  // Computed signal (read-only, derived)
  doubled = computed(() => this.count() * 2);
  isPositive = computed(() => this.count() > 0);
  
  increment() {
    this.count.update(n => n + 1);
  }
  
  decrement() {
    this.count.update(n => n - 1);
  }
  
  reset() {
    this.count.set(0);
  }
}
```

### Signal Inputs (Angular 17.1+)

```typescript
@Component({
  selector: 'app-user',
  template: `<div>{{ user().name }}</div>`
})
export class UserComponent {
  // Signal-based input
  user = input.required<User>();
  
  // Optional with default
  showAvatar = input(true);
  
  // Transform input
  userId = input(0, { transform: (v: string) => parseInt(v, 10) });
  
  // Computed from input
  fullName = computed(() => 
    `${this.user().firstName} ${this.user().lastName}`
  );
}

// Usage
<app-user [user]="currentUser" [showAvatar]="false" />
```

### Signal Outputs (Angular 17.3+)

```typescript
@Component({
  selector: 'app-counter',
  template: `<button (click)="increment()">+</button>`
})
export class CounterComponent {
  // Signal-based output
  countChange = output<number>();
  
  // With options
  closed = output({ alias: 'onClose' });
  
  private count = signal(0);
  
  increment() {
    this.count.update(n => n + 1);
    this.countChange.emit(this.count());
  }
}

// Usage
<app-counter (countChange)="onCountChange($event)" />
```

### Converting Between Signals and Observables

```typescript
import { toSignal, toObservable } from '@angular/core/rxjs-interop';

@Component({...})
export class HybridComponent {
  // Observable → Signal
  users = toSignal(
    this.userService.getUsers(),
    { initialValue: [] }
  );
  
  // Signal → Observable
  searchTerm = signal('');
  searchTerm$ = toObservable(this.searchTerm);
  
  // Use in reactive pipeline
  searchResults$ = this.searchTerm$.pipe(
    debounceTime(300),
    switchMap(term => this.searchService.search(term))
  );
  
  // Back to signal for template
  searchResults = toSignal(this.searchResults$, { initialValue: [] });
  
  constructor(
    private userService: UserService,
    private searchService: SearchService
  ) {}
}
```

---

## Interview Questions & Answers

### Q1: What's the difference between switchMap, mergeMap, concatMap, and exhaustMap?

**Answer:**

| Operator | Behavior | Use Case |
|----------|----------|----------|
| **switchMap** | Cancels previous inner observable | Search, route changes |
| **mergeMap** | Runs all concurrently | Parallel requests |
| **concatMap** | Runs sequentially in order | Ordered operations |
| **exhaustMap** | Ignores new while busy | Form submissions |

```typescript
// switchMap - Type-ahead search (cancel previous)
searchInput$.pipe(
  switchMap(term => this.search(term))  // New search cancels old
);

// mergeMap - Load all user details (parallel)
userIds$.pipe(
  mergeMap(id => this.loadUser(id), 5)  // 5 concurrent max
);

// concatMap - Save queue (maintain order)
saveActions$.pipe(
  concatMap(action => this.save(action))  // Wait for each
);

// exhaustMap - Submit button (ignore rapid clicks)
submitBtn$.pipe(
  exhaustMap(() => this.submit())  // Ignore while submitting
);
```

---

### Q2: How do you prevent memory leaks in Angular?

**Answer:**

**Memory leaks happen when subscriptions aren't cleaned up.**

**Solutions:**

1. **Async Pipe** (automatic)
```typescript
<div *ngIf="user$ | async as user">{{ user.name }}</div>
```

2. **takeUntil Pattern**
```typescript
private destroy$ = new Subject<void>();

ngOnInit() {
  source$.pipe(takeUntil(this.destroy$)).subscribe();
}

ngOnDestroy() {
  this.destroy$.next();
  this.destroy$.complete();
}
```

3. **takeUntilDestroyed (Angular 16+)**
```typescript
data$ = source$.pipe(takeUntilDestroyed());
```

4. **take(1) / first() for one-time requests**
```typescript
this.http.get('/api').pipe(take(1)).subscribe();
```

---

### Q3: When would you use NgRx vs a simple service with BehaviorSubject?

**Answer:**

**Use BehaviorSubject service when:**
- Small to medium apps
- Simple state requirements
- Team not familiar with Redux patterns
- Prototyping / MVPs

**Use NgRx when:**
- Large applications with complex state
- Multiple developers need predictable patterns
- Need time-travel debugging
- Complex async flows
- State needs to be serializable

```typescript
// Simple service - easier for small apps
@Injectable({ providedIn: 'root' })
export class CartService {
  private items$ = new BehaviorSubject<CartItem[]>([]);
  
  addItem(item: CartItem) {
    this.items$.next([...this.items$.value, item]);
  }
}

// NgRx - better for complex apps
// Actions, reducers, effects, selectors...
// More boilerplate but more predictable and testable
```

---

### Q4: Explain Angular Signals and when to use them vs RxJS.

**Answer:**

**Signals** (Angular 16+):
- Synchronous value access
- Simple API (signal, computed, effect)
- Better for UI state
- Fine-grained change detection

**RxJS**:
- Asynchronous streams
- Powerful operators
- Better for async operations, HTTP, events
- Complex data transformations

```typescript
// Use Signals for:
count = signal(0);                    // Simple state
doubled = computed(() => count() * 2); // Derived state
isValid = computed(() => form().valid); // UI state

// Use RxJS for:
searchResults$ = searchTerm$.pipe(
  debounceTime(300),
  switchMap(term => this.http.get(`/search?q=${term}`))
);

// They work together:
users = toSignal(this.http.get<User[]>('/api/users'));
```

---

### Q5: What is the difference between Subject, BehaviorSubject, and ReplaySubject?

**Answer:**

| Subject Type | Initial Value | New Subscriber Receives |
|--------------|---------------|------------------------|
| **Subject** | None | Only future values |
| **BehaviorSubject** | Required | Last value + future |
| **ReplaySubject** | None | Last N values + future |
| **AsyncSubject** | None | Only last value on complete |

```typescript
// Subject - Events, notifications
const clicks$ = new Subject<MouseEvent>();

// BehaviorSubject - Current state
const currentUser$ = new BehaviorSubject<User | null>(null);
currentUser$.value;  // Sync access

// ReplaySubject - Cache recent values
const messages$ = new ReplaySubject<Message>(10);  // Last 10

// AsyncSubject - Final result only
const calculation$ = new AsyncSubject<number>();
```

---

## Summary: RxJS & State Management Checklist

✅ **RxJS Essentials**
- Understand Observable vs Promise differences
- Master transformation operators (map, switchMap, etc.)
- Know when to use each flattening operator
- Handle errors properly with catchError + retry

✅ **Memory Management**
- Always unsubscribe (takeUntil, async pipe, take)
- Use takeUntilDestroyed in Angular 16+
- Prefer async pipe in templates

✅ **State Management**
- Start with services + BehaviorSubject
- Graduate to NgRx for complex apps
- Consider Signals for simple UI state

✅ **Best Practices**
- Use distinctUntilChanged to avoid duplicate emissions
- Use shareReplay for expensive observables
- Combine operators efficiently
- Keep effects focused and testable

---

**Key Takeaway:** RxJS is powerful but can be complex. Start with simple patterns, understand the operators deeply, and always clean up subscriptions. Choose the right state management approach based on your app's complexity.
