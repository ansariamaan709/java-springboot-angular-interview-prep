# Angular HTTP, Interceptors & Testing - Complete Interview Guide

## Table of Contents

1. [HttpClient Fundamentals](#httpclient-fundamentals)
2. [HTTP Methods & Options](#http-methods--options)
3. [Error Handling](#error-handling)
4. [HTTP Interceptors](#http-interceptors)
5. [Caching Strategies](#caching-strategies)
6. [Unit Testing Components](#unit-testing-components)
7. [Testing Services](#testing-services)
8. [Testing HTTP Calls](#testing-http-calls)
9. [Component Testing Deep Dive](#component-testing-deep-dive)
10. [E2E Testing Overview](#e2e-testing-overview)
11. [Interview Questions & Answers](#interview-questions--answers)

---

## HttpClient Fundamentals

### Setup

```typescript
// Standalone (Angular 15+)
import { provideHttpClient, withInterceptors } from "@angular/common/http";

bootstrapApplication(AppComponent, {
  providers: [
    provideHttpClient(withInterceptors([authInterceptor, loggingInterceptor])),
  ],
});

// NgModule approach
import { HttpClientModule } from "@angular/common/http";

@NgModule({
  imports: [HttpClientModule],
})
export class AppModule {}
```

### Basic HTTP Operations

```typescript
import { HttpClient, HttpHeaders, HttpParams } from "@angular/common/http";

@Injectable({ providedIn: "root" })
export class UserService {
  private apiUrl = "/api/users";

  constructor(private http: HttpClient) {}

  // GET - Retrieve data
  getUsers(): Observable<User[]> {
    return this.http.get<User[]>(this.apiUrl);
  }

  getUser(id: number): Observable<User> {
    return this.http.get<User>(`${this.apiUrl}/${id}`);
  }

  // POST - Create data
  createUser(user: Partial<User>): Observable<User> {
    return this.http.post<User>(this.apiUrl, user);
  }

  // PUT - Full update
  updateUser(id: number, user: User): Observable<User> {
    return this.http.put<User>(`${this.apiUrl}/${id}`, user);
  }

  // PATCH - Partial update
  patchUser(id: number, changes: Partial<User>): Observable<User> {
    return this.http.patch<User>(`${this.apiUrl}/${id}`, changes);
  }

  // DELETE - Remove data
  deleteUser(id: number): Observable<void> {
    return this.http.delete<void>(`${this.apiUrl}/${id}`);
  }
}
```

---

## HTTP Methods & Options

### HTTP Options

```typescript
@Injectable({ providedIn: "root" })
export class ApiService {
  constructor(private http: HttpClient) {}

  // With headers
  getWithHeaders(): Observable<User[]> {
    const headers = new HttpHeaders({
      "Content-Type": "application/json",
      Authorization: "Bearer token123",
      "X-Custom-Header": "value",
    });

    return this.http.get<User[]>("/api/users", { headers });
  }

  // With query params
  searchUsers(term: string, page: number): Observable<User[]> {
    const params = new HttpParams()
      .set("search", term)
      .set("page", page.toString())
      .set("limit", "10");

    return this.http.get<User[]>("/api/users", { params });
    // URL: /api/users?search=john&page=1&limit=10
  }

  // Observe response (access headers, status)
  getWithFullResponse(): Observable<HttpResponse<User[]>> {
    return this.http.get<User[]>("/api/users", {
      observe: "response",
    });
  }

  // Observe events (for upload progress)
  uploadWithProgress(file: File): Observable<HttpEvent<any>> {
    const formData = new FormData();
    formData.append("file", file);

    return this.http.post("/api/upload", formData, {
      observe: "events",
      reportProgress: true,
    });
  }

  // Response type
  downloadFile(id: string): Observable<Blob> {
    return this.http.get(`/api/files/${id}`, {
      responseType: "blob",
    });
  }

  getTextFile(): Observable<string> {
    return this.http.get("/api/readme", {
      responseType: "text",
    });
  }
}
```

### File Upload with Progress

```typescript
@Component({
  template: `
    <input type="file" (change)="onFileSelected($event)" />
    <button (click)="upload()" [disabled]="!selectedFile">Upload</button>

    <div *ngIf="uploadProgress > 0" class="progress">
      <div [style.width.%]="uploadProgress">{{ uploadProgress }}%</div>
    </div>
  `,
})
export class UploadComponent {
  selectedFile: File | null = null;
  uploadProgress = 0;

  constructor(private http: HttpClient) {}

  onFileSelected(event: Event) {
    const input = event.target as HTMLInputElement;
    this.selectedFile = input.files?.[0] || null;
  }

  upload() {
    if (!this.selectedFile) return;

    const formData = new FormData();
    formData.append("file", this.selectedFile);

    this.http
      .post("/api/upload", formData, {
        observe: "events",
        reportProgress: true,
      })
      .subscribe((event) => {
        if (event.type === HttpEventType.UploadProgress) {
          this.uploadProgress = Math.round(
            (100 * event.loaded) / (event.total || 1)
          );
        } else if (event.type === HttpEventType.Response) {
          console.log("Upload complete", event.body);
          this.uploadProgress = 0;
        }
      });
  }
}
```

---

## Error Handling

### Comprehensive Error Handling

```typescript
import { HttpErrorResponse } from "@angular/common/http";
import { throwError, retry, catchError } from "rxjs";

@Injectable({ providedIn: "root" })
export class UserService {
  constructor(private http: HttpClient) {}

  getUsers(): Observable<User[]> {
    return this.http.get<User[]>("/api/users").pipe(
      retry(2), // Retry up to 2 times
      catchError(this.handleError)
    );
  }

  private handleError(error: HttpErrorResponse): Observable<never> {
    let errorMessage = "An error occurred";

    if (error.status === 0) {
      // Network error
      errorMessage = "Network error. Please check your connection.";
    } else if (error.error instanceof ErrorEvent) {
      // Client-side error
      errorMessage = `Client error: ${error.error.message}`;
    } else {
      // Server-side error
      switch (error.status) {
        case 400:
          errorMessage = error.error?.message || "Bad request";
          break;
        case 401:
          errorMessage = "Unauthorized. Please login again.";
          break;
        case 403:
          errorMessage = "Forbidden. You don't have permission.";
          break;
        case 404:
          errorMessage = "Resource not found.";
          break;
        case 422:
          errorMessage = "Validation error.";
          break;
        case 500:
          errorMessage = "Internal server error.";
          break;
        default:
          errorMessage = `Error ${error.status}: ${error.statusText}`;
      }
    }

    console.error("HTTP Error:", error);
    return throwError(() => new Error(errorMessage));
  }
}
```

### Error Handling in Components

```typescript
@Component({
  template: `
    <div *ngIf="loading" class="spinner">Loading...</div>

    <div *ngIf="error" class="error">
      {{ error }}
      <button (click)="retry()">Retry</button>
    </div>

    <ul *ngIf="users.length">
      <li *ngFor="let user of users">{{ user.name }}</li>
    </ul>
  `,
})
export class UserListComponent implements OnInit {
  users: User[] = [];
  loading = false;
  error: string | null = null;

  constructor(private userService: UserService) {}

  ngOnInit() {
    this.loadUsers();
  }

  loadUsers() {
    this.loading = true;
    this.error = null;

    this.userService.getUsers().subscribe({
      next: (users) => {
        this.users = users;
        this.loading = false;
      },
      error: (err) => {
        this.error = err.message;
        this.loading = false;
      },
    });
  }

  retry() {
    this.loadUsers();
  }
}
```

---

## HTTP Interceptors

### Interceptor Architecture

```
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚                       HTTP INTERCEPTOR CHAIN                              â”‚
â”œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¤
â”‚                                                                           â”‚
â”‚  Request Flow:                                                           â”‚
â”‚  â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”    â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”    â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”    â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”      â”‚
â”‚  â”‚ Request â”‚â”€â”€â”€â–¶â”‚ Interceptor â”‚â”€â”€â”€â–¶â”‚ Interceptor â”‚â”€â”€â”€â–¶â”‚  Server  â”‚      â”‚
â”‚  â”‚         â”‚    â”‚     1       â”‚    â”‚     2       â”‚    â”‚          â”‚      â”‚
â”‚  â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜    â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜    â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜    â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜      â”‚
â”‚                                                                           â”‚
â”‚  Response Flow:                                                          â”‚
â”‚  â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”    â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”    â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”    â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”      â”‚
â”‚  â”‚Response â”‚â—€â”€â”€â”€â”‚ Interceptor â”‚â—€â”€â”€â”€â”‚ Interceptor â”‚â—€â”€â”€â”€â”‚  Server  â”‚      â”‚
â”‚  â”‚         â”‚    â”‚     1       â”‚    â”‚     2       â”‚    â”‚          â”‚      â”‚
â”‚  â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜    â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜    â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜    â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜      â”‚
â”‚                                                                           â”‚
â”‚  Common Use Cases:                                                       â”‚
â”‚  â€¢ Authentication (add tokens)                                           â”‚
â”‚  â€¢ Logging                                                               â”‚
â”‚  â€¢ Error handling                                                        â”‚
â”‚  â€¢ Caching                                                               â”‚
â”‚  â€¢ Loading indicators                                                    â”‚
â”‚                                                                           â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

### Functional Interceptors (Angular 15+)

```typescript
import {
  HttpInterceptorFn,
  HttpHandlerFn,
  HttpRequest,
} from "@angular/common/http";

// Auth interceptor
export const authInterceptor: HttpInterceptorFn = (req, next) => {
  const authService = inject(AuthService);
  const token = authService.getToken();

  if (token) {
    const clonedReq = req.clone({
      setHeaders: {
        Authorization: `Bearer ${token}`,
      },
    });
    return next(clonedReq);
  }

  return next(req);
};

// Logging interceptor
export const loggingInterceptor: HttpInterceptorFn = (req, next) => {
  const started = Date.now();

  return next(req).pipe(
    tap({
      next: (event) => {
        if (event.type === HttpEventType.Response) {
          const elapsed = Date.now() - started;
          console.log(`${req.method} ${req.url} - ${elapsed}ms`);
        }
      },
      error: (error) => {
        const elapsed = Date.now() - started;
        console.error(`${req.method} ${req.url} failed after ${elapsed}ms`);
      },
    })
  );
};

// Error handling interceptor
export const errorInterceptor: HttpInterceptorFn = (req, next) => {
  const router = inject(Router);
  const authService = inject(AuthService);

  return next(req).pipe(
    catchError((error: HttpErrorResponse) => {
      if (error.status === 401) {
        authService.logout();
        router.navigate(["/login"]);
      }
      return throwError(() => error);
    })
  );
};

// Loading interceptor
export const loadingInterceptor: HttpInterceptorFn = (req, next) => {
  const loadingService = inject(LoadingService);

  loadingService.show();

  return next(req).pipe(finalize(() => loadingService.hide()));
};

// Register interceptors
bootstrapApplication(AppComponent, {
  providers: [
    provideHttpClient(
      withInterceptors([
        authInterceptor,
        loggingInterceptor,
        errorInterceptor,
        loadingInterceptor,
      ])
    ),
  ],
});
```

### Class-Based Interceptors (Traditional)

```typescript
import {
  HTTP_INTERCEPTORS,
  HttpInterceptor,
  HttpHandler,
} from "@angular/common/http";

@Injectable()
export class AuthInterceptor implements HttpInterceptor {
  constructor(private authService: AuthService) {}

  intercept(
    req: HttpRequest<any>,
    next: HttpHandler
  ): Observable<HttpEvent<any>> {
    const token = this.authService.getToken();

    if (token) {
      const clonedReq = req.clone({
        setHeaders: {
          Authorization: `Bearer ${token}`,
        },
      });
      return next.handle(clonedReq);
    }

    return next.handle(req);
  }
}

@Injectable()
export class RetryInterceptor implements HttpInterceptor {
  intercept(
    req: HttpRequest<any>,
    next: HttpHandler
  ): Observable<HttpEvent<any>> {
    return next.handle(req).pipe(
      retry({
        count: 3,
        delay: (error, retryCount) => {
          if (error.status === 503) {
            // Service unavailable
            return timer(1000 * retryCount); // Exponential backoff
          }
          throw error; // Don't retry other errors
        },
      })
    );
  }
}

// Register
@NgModule({
  providers: [
    { provide: HTTP_INTERCEPTORS, useClass: AuthInterceptor, multi: true },
    { provide: HTTP_INTERCEPTORS, useClass: RetryInterceptor, multi: true },
  ],
})
export class AppModule {}
```

### Token Refresh Interceptor

```typescript
export const tokenRefreshInterceptor: HttpInterceptorFn = (req, next) => {
  const authService = inject(AuthService);

  return next(req).pipe(
    catchError((error: HttpErrorResponse) => {
      if (error.status === 401 && !req.url.includes("/refresh")) {
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

---

## Caching Strategies

### Simple Caching Service

```typescript
@Injectable({ providedIn: "root" })
export class CacheService {
  private cache = new Map<string, { data: any; timestamp: number }>();
  private defaultTTL = 5 * 60 * 1000; // 5 minutes

  get<T>(key: string): T | null {
    const cached = this.cache.get(key);

    if (cached && Date.now() - cached.timestamp < this.defaultTTL) {
      return cached.data as T;
    }

    this.cache.delete(key);
    return null;
  }

  set(key: string, data: any): void {
    this.cache.set(key, { data, timestamp: Date.now() });
  }

  clear(key?: string): void {
    if (key) {
      this.cache.delete(key);
    } else {
      this.cache.clear();
    }
  }
}

// Usage in service
@Injectable({ providedIn: "root" })
export class UserService {
  constructor(private http: HttpClient, private cache: CacheService) {}

  getUsers(): Observable<User[]> {
    const cached = this.cache.get<User[]>("users");
    if (cached) {
      return of(cached);
    }

    return this.http
      .get<User[]>("/api/users")
      .pipe(tap((users) => this.cache.set("users", users)));
  }
}
```

### Caching with shareReplay

```typescript
@Injectable({ providedIn: "root" })
export class ConfigService {
  private config$: Observable<Config> | null = null;

  constructor(private http: HttpClient) {}

  getConfig(): Observable<Config> {
    if (!this.config$) {
      this.config$ = this.http.get<Config>("/api/config").pipe(
        shareReplay(1) // Cache last emitted value
      );
    }
    return this.config$;
  }

  clearCache(): void {
    this.config$ = null;
  }
}
```

### Caching Interceptor

```typescript
const CACHEABLE_URLS = ["/api/config", "/api/categories", "/api/countries"];

export const cacheInterceptor: HttpInterceptorFn = (req, next) => {
  // Only cache GET requests
  if (req.method !== "GET") {
    return next(req);
  }

  // Check if URL is cacheable
  if (!CACHEABLE_URLS.some((url) => req.url.includes(url))) {
    return next(req);
  }

  const cacheService = inject(CacheService);
  const cached = cacheService.get(req.url);

  if (cached) {
    return of(new HttpResponse({ body: cached }));
  }

  return next(req).pipe(
    tap((event) => {
      if (event instanceof HttpResponse) {
        cacheService.set(req.url, event.body);
      }
    })
  );
};
```

---

## Unit Testing Components

### Testing Setup

```typescript
import {
  ComponentFixture,
  TestBed,
  fakeAsync,
  tick,
} from "@angular/core/testing";
import { By } from "@angular/platform-browser";

describe("UserListComponent", () => {
  let component: UserListComponent;
  let fixture: ComponentFixture<UserListComponent>;
  let userServiceSpy: jasmine.SpyObj<UserService>;

  beforeEach(async () => {
    // Create spy object
    const spy = jasmine.createSpyObj("UserService", ["getUsers", "deleteUser"]);

    await TestBed.configureTestingModule({
      imports: [UserListComponent], // Standalone component
      providers: [{ provide: UserService, useValue: spy }],
    }).compileComponents();

    fixture = TestBed.createComponent(UserListComponent);
    component = fixture.componentInstance;
    userServiceSpy = TestBed.inject(UserService) as jasmine.SpyObj<UserService>;
  });

  it("should create", () => {
    expect(component).toBeTruthy();
  });

  it("should load users on init", () => {
    const mockUsers: User[] = [
      { id: 1, name: "John" },
      { id: 2, name: "Jane" },
    ];
    userServiceSpy.getUsers.and.returnValue(of(mockUsers));

    fixture.detectChanges(); // Triggers ngOnInit

    expect(userServiceSpy.getUsers).toHaveBeenCalled();
    expect(component.users).toEqual(mockUsers);
  });

  it("should display users in template", () => {
    const mockUsers: User[] = [
      { id: 1, name: "John" },
      { id: 2, name: "Jane" },
    ];
    userServiceSpy.getUsers.and.returnValue(of(mockUsers));

    fixture.detectChanges();

    const userElements = fixture.debugElement.queryAll(By.css(".user-item"));
    expect(userElements.length).toBe(2);
    expect(userElements[0].nativeElement.textContent).toContain("John");
  });

  it("should show loading indicator", () => {
    userServiceSpy.getUsers.and.returnValue(of([]).pipe(delay(100)));

    fixture.detectChanges();

    const loading = fixture.debugElement.query(By.css(".loading"));
    expect(loading).toBeTruthy();
  });

  it("should handle error", () => {
    userServiceSpy.getUsers.and.returnValue(
      throwError(() => new Error("Server error"))
    );

    fixture.detectChanges();

    expect(component.error).toBe("Server error");
    const errorEl = fixture.debugElement.query(By.css(".error"));
    expect(errorEl.nativeElement.textContent).toContain("Server error");
  });
});
```

### Testing Async Operations

```typescript
describe("AsyncComponent", () => {
  let component: AsyncComponent;
  let fixture: ComponentFixture<AsyncComponent>;

  beforeEach(async () => {
    await TestBed.configureTestingModule({
      imports: [AsyncComponent],
    }).compileComponents();

    fixture = TestBed.createComponent(AsyncComponent);
    component = fixture.componentInstance;
  });

  // Using fakeAsync + tick
  it("should update after delay (fakeAsync)", fakeAsync(() => {
    component.delayedAction();

    expect(component.status).toBe("pending");

    tick(1000); // Advance time by 1 second

    expect(component.status).toBe("complete");
  }));

  // Using async/await with fixture.whenStable()
  it("should update after async operation (async)", async () => {
    component.asyncAction();

    await fixture.whenStable();
    fixture.detectChanges();

    expect(component.result).toBeDefined();
  });

  // Using done callback
  it("should emit value (done callback)", (done) => {
    component.output.subscribe((value) => {
      expect(value).toBe("emitted");
      done();
    });

    component.emitValue();
  });
});
```

### Testing Input/Output

```typescript
@Component({
  selector: "app-counter",
  template: `
    <button (click)="decrement()">-</button>
    <span class="count">{{ count }}</span>
    <button (click)="increment()">+</button>
  `,
})
export class CounterComponent {
  @Input() count = 0;
  @Output() countChange = new EventEmitter<number>();

  increment() {
    this.count++;
    this.countChange.emit(this.count);
  }

  decrement() {
    this.count--;
    this.countChange.emit(this.count);
  }
}

describe("CounterComponent", () => {
  let component: CounterComponent;
  let fixture: ComponentFixture<CounterComponent>;

  beforeEach(async () => {
    await TestBed.configureTestingModule({
      imports: [CounterComponent],
    }).compileComponents();

    fixture = TestBed.createComponent(CounterComponent);
    component = fixture.componentInstance;
  });

  it("should accept input", () => {
    component.count = 10;
    fixture.detectChanges();

    const countEl = fixture.debugElement.query(By.css(".count"));
    expect(countEl.nativeElement.textContent).toContain("10");
  });

  it("should emit on increment", () => {
    spyOn(component.countChange, "emit");

    component.count = 5;
    component.increment();

    expect(component.countChange.emit).toHaveBeenCalledWith(6);
  });

  it("should increment on button click", () => {
    component.count = 0;
    fixture.detectChanges();

    const incrementBtn = fixture.debugElement.queryAll(By.css("button"))[1];
    incrementBtn.triggerEventHandler("click", null);

    expect(component.count).toBe(1);
  });
});
```

---

## Testing Services

### Basic Service Testing

```typescript
describe("UserService", () => {
  let service: UserService;
  let httpMock: HttpTestingController;

  beforeEach(() => {
    TestBed.configureTestingModule({
      imports: [HttpClientTestingModule],
      providers: [UserService],
    });

    service = TestBed.inject(UserService);
    httpMock = TestBed.inject(HttpTestingController);
  });

  afterEach(() => {
    // Verify no outstanding requests
    httpMock.verify();
  });

  it("should fetch users", () => {
    const mockUsers: User[] = [
      { id: 1, name: "John" },
      { id: 2, name: "Jane" },
    ];

    service.getUsers().subscribe((users) => {
      expect(users.length).toBe(2);
      expect(users).toEqual(mockUsers);
    });

    const req = httpMock.expectOne("/api/users");
    expect(req.request.method).toBe("GET");
    req.flush(mockUsers);
  });

  it("should create user", () => {
    const newUser = { name: "Bob", email: "bob@test.com" };
    const createdUser = { id: 3, ...newUser };

    service.createUser(newUser).subscribe((user) => {
      expect(user).toEqual(createdUser);
    });

    const req = httpMock.expectOne("/api/users");
    expect(req.request.method).toBe("POST");
    expect(req.request.body).toEqual(newUser);
    req.flush(createdUser);
  });

  it("should handle error", () => {
    service.getUsers().subscribe({
      next: () => fail("should have failed"),
      error: (error) => {
        expect(error.message).toContain("Server error");
      },
    });

    const req = httpMock.expectOne("/api/users");
    req.flush("Server error", { status: 500, statusText: "Server Error" });
  });
});
```

### Testing with Dependencies

```typescript
describe("AuthService", () => {
  let service: AuthService;
  let httpMock: HttpTestingController;
  let routerSpy: jasmine.SpyObj<Router>;
  let storageSpy: jasmine.SpyObj<StorageService>;

  beforeEach(() => {
    routerSpy = jasmine.createSpyObj("Router", ["navigate"]);
    storageSpy = jasmine.createSpyObj("StorageService", [
      "get",
      "set",
      "remove",
    ]);

    TestBed.configureTestingModule({
      imports: [HttpClientTestingModule],
      providers: [
        AuthService,
        { provide: Router, useValue: routerSpy },
        { provide: StorageService, useValue: storageSpy },
      ],
    });

    service = TestBed.inject(AuthService);
    httpMock = TestBed.inject(HttpTestingController);
  });

  it("should login and store token", () => {
    const credentials = { email: "test@test.com", password: "password" };
    const response = { token: "jwt-token", user: { id: 1, name: "Test" } };

    service.login(credentials).subscribe((result) => {
      expect(result.token).toBe("jwt-token");
    });

    const req = httpMock.expectOne("/api/auth/login");
    req.flush(response);

    expect(storageSpy.set).toHaveBeenCalledWith("token", "jwt-token");
  });

  it("should logout and navigate to login", () => {
    service.logout();

    expect(storageSpy.remove).toHaveBeenCalledWith("token");
    expect(routerSpy.navigate).toHaveBeenCalledWith(["/login"]);
  });
});
```

---

## Testing HTTP Calls

### HttpTestingController

```typescript
import {
  HttpClientTestingModule,
  HttpTestingController,
} from "@angular/common/http/testing";

describe("ApiService", () => {
  let service: ApiService;
  let httpMock: HttpTestingController;

  beforeEach(() => {
    TestBed.configureTestingModule({
      imports: [HttpClientTestingModule],
      providers: [ApiService],
    });

    service = TestBed.inject(ApiService);
    httpMock = TestBed.inject(HttpTestingController);
  });

  afterEach(() => {
    httpMock.verify(); // Ensure no outstanding requests
  });

  // Test with query params
  it("should search with params", () => {
    service.search("test", 1, 10).subscribe();

    const req = httpMock.expectOne(
      (request) =>
        request.url === "/api/search" &&
        request.params.get("q") === "test" &&
        request.params.get("page") === "1"
    );

    expect(req.request.method).toBe("GET");
    req.flush({ results: [] });
  });

  // Test with headers
  it("should send custom headers", () => {
    service.fetchWithAuth().subscribe();

    const req = httpMock.expectOne("/api/protected");
    expect(req.request.headers.get("Authorization")).toBeTruthy();
    req.flush({});
  });

  // Test multiple requests
  it("should handle multiple requests", () => {
    service.getUsers().subscribe();
    service.getProducts().subscribe();

    const requests = httpMock.match((req) => true);
    expect(requests.length).toBe(2);

    requests.forEach((req) => req.flush([]));
  });

  // Test error responses
  it("should handle 404", () => {
    service.getUser(999).subscribe({
      error: (error) => {
        expect(error.status).toBe(404);
      },
    });

    const req = httpMock.expectOne("/api/users/999");
    req.flush("Not found", { status: 404, statusText: "Not Found" });
  });

  // Test network error
  it("should handle network error", () => {
    service.getUsers().subscribe({
      error: (error) => {
        expect(error.message).toContain("Network");
      },
    });

    const req = httpMock.expectOne("/api/users");
    req.error(new ProgressEvent("Network error"));
  });
});
```

### Testing Interceptors

```typescript
describe("AuthInterceptor", () => {
  let httpMock: HttpTestingController;
  let http: HttpClient;
  let authServiceSpy: jasmine.SpyObj<AuthService>;

  beforeEach(() => {
    authServiceSpy = jasmine.createSpyObj("AuthService", ["getToken"]);

    TestBed.configureTestingModule({
      providers: [
        provideHttpClient(withInterceptors([authInterceptor])),
        provideHttpClientTesting(),
        { provide: AuthService, useValue: authServiceSpy },
      ],
    });

    http = TestBed.inject(HttpClient);
    httpMock = TestBed.inject(HttpTestingController);
  });

  it("should add auth header when token exists", () => {
    authServiceSpy.getToken.and.returnValue("test-token");

    http.get("/api/data").subscribe();

    const req = httpMock.expectOne("/api/data");
    expect(req.request.headers.get("Authorization")).toBe("Bearer test-token");
    req.flush({});
  });

  it("should not add header when no token", () => {
    authServiceSpy.getToken.and.returnValue(null);

    http.get("/api/data").subscribe();

    const req = httpMock.expectOne("/api/data");
    expect(req.request.headers.has("Authorization")).toBeFalse();
    req.flush({});
  });
});
```

---

## Component Testing Deep Dive

### Testing with Router

```typescript
import { RouterTestingModule } from "@angular/router/testing";
import { Router } from "@angular/router";

describe("NavigationComponent", () => {
  let component: NavigationComponent;
  let fixture: ComponentFixture<NavigationComponent>;
  let router: Router;

  beforeEach(async () => {
    await TestBed.configureTestingModule({
      imports: [
        NavigationComponent,
        RouterTestingModule.withRoutes([
          { path: "home", component: DummyComponent },
          { path: "users", component: DummyComponent },
        ]),
      ],
    }).compileComponents();

    fixture = TestBed.createComponent(NavigationComponent);
    component = fixture.componentInstance;
    router = TestBed.inject(Router);

    fixture.detectChanges();
  });

  it("should navigate to users", fakeAsync(() => {
    const navigateSpy = spyOn(router, "navigate");

    component.goToUsers();

    expect(navigateSpy).toHaveBeenCalledWith(["/users"]);
  }));

  it("should navigate via routerLink", fakeAsync(() => {
    const link = fixture.debugElement.query(By.css('a[routerLink="/users"]'));
    link.nativeElement.click();

    tick();

    expect(router.url).toBe("/users");
  }));
});
```

### Testing Forms

```typescript
describe("LoginFormComponent", () => {
  let component: LoginFormComponent;
  let fixture: ComponentFixture<LoginFormComponent>;

  beforeEach(async () => {
    await TestBed.configureTestingModule({
      imports: [LoginFormComponent, ReactiveFormsModule],
    }).compileComponents();

    fixture = TestBed.createComponent(LoginFormComponent);
    component = fixture.componentInstance;
    fixture.detectChanges();
  });

  it("should create form with controls", () => {
    expect(component.loginForm.contains("email")).toBeTruthy();
    expect(component.loginForm.contains("password")).toBeTruthy();
  });

  it("should require email", () => {
    const email = component.loginForm.get("email");
    email?.setValue("");

    expect(email?.valid).toBeFalsy();
    expect(email?.errors?.["required"]).toBeTruthy();
  });

  it("should validate email format", () => {
    const email = component.loginForm.get("email");
    email?.setValue("invalid-email");

    expect(email?.errors?.["email"]).toBeTruthy();
  });

  it("should be valid with correct data", () => {
    component.loginForm.patchValue({
      email: "test@test.com",
      password: "password123",
    });

    expect(component.loginForm.valid).toBeTruthy();
  });

  it("should submit form", () => {
    spyOn(component, "onSubmit");

    component.loginForm.patchValue({
      email: "test@test.com",
      password: "password123",
    });

    const form = fixture.debugElement.query(By.css("form"));
    form.triggerEventHandler("ngSubmit", null);

    expect(component.onSubmit).toHaveBeenCalled();
  });

  it("should disable submit button when invalid", () => {
    const submitBtn = fixture.debugElement.query(
      By.css('button[type="submit"]')
    );

    expect(submitBtn.nativeElement.disabled).toBeTruthy();

    component.loginForm.patchValue({
      email: "test@test.com",
      password: "password123",
    });
    fixture.detectChanges();

    expect(submitBtn.nativeElement.disabled).toBeFalsy();
  });
});
```

---

## E2E Testing Overview

### Playwright (Recommended)

```typescript
// e2e/login.spec.ts
import { test, expect } from "@playwright/test";

test.describe("Login Page", () => {
  test.beforeEach(async ({ page }) => {
    await page.goto("/login");
  });

  test("should display login form", async ({ page }) => {
    await expect(page.locator("h1")).toHaveText("Login");
    await expect(page.locator('input[name="email"]')).toBeVisible();
    await expect(page.locator('input[name="password"]')).toBeVisible();
  });

  test("should show validation errors", async ({ page }) => {
    await page.click('button[type="submit"]');

    await expect(page.locator(".error")).toContainText("Email is required");
  });

  test("should login successfully", async ({ page }) => {
    await page.fill('input[name="email"]', "user@test.com");
    await page.fill('input[name="password"]', "password123");
    await page.click('button[type="submit"]');

    await expect(page).toHaveURL("/dashboard");
    await expect(page.locator(".welcome")).toContainText("Welcome");
  });

  test("should handle invalid credentials", async ({ page }) => {
    await page.fill('input[name="email"]', "wrong@test.com");
    await page.fill('input[name="password"]', "wrongpassword");
    await page.click('button[type="submit"]');

    await expect(page.locator(".error-message")).toContainText(
      "Invalid credentials"
    );
  });
});
```

### Cypress Alternative

```typescript
// cypress/e2e/login.cy.ts
describe("Login Page", () => {
  beforeEach(() => {
    cy.visit("/login");
  });

  it("should login successfully", () => {
    cy.get('input[name="email"]').type("user@test.com");
    cy.get('input[name="password"]').type("password123");
    cy.get('button[type="submit"]').click();

    cy.url().should("include", "/dashboard");
    cy.contains("Welcome").should("be.visible");
  });

  it("should intercept API calls", () => {
    cy.intercept("POST", "/api/auth/login", {
      statusCode: 200,
      body: { token: "fake-token", user: { name: "Test" } },
    }).as("loginRequest");

    cy.get('input[name="email"]').type("user@test.com");
    cy.get('input[name="password"]').type("password123");
    cy.get('button[type="submit"]').click();

    cy.wait("@loginRequest");
    cy.url().should("include", "/dashboard");
  });
});
```

---

## Interview Questions & Answers

### Q1: What are HTTP Interceptors and when would you use them?

**Answer:**

Interceptors are middleware that can transform HTTP requests/responses globally.

```typescript
// Auth interceptor
export const authInterceptor: HttpInterceptorFn = (req, next) => {
  const authToken = inject(AuthService).getToken();
  
  if (authToken) {
    req = req.clone({
      setHeaders: { Authorization: `Bearer ${authToken}` }
    });
  }
  
  return next(req);
};

// Registration
bootstrapApplication(AppComponent, {
  providers: [
    provideHttpClient(withInterceptors([authInterceptor, loggingInterceptor]))
  ]
});
```

**Common use cases:**
- Authentication (add tokens)
- Logging and analytics
- Error handling (global)
- Caching responses
- Request/response transformation

---

### Q2: How do you test HTTP calls in Angular?

**Answer:**

```typescript
describe('UserService', () => {
  let service: UserService;
  let httpMock: HttpTestingController;
  
  beforeEach(() => {
    TestBed.configureTestingModule({
      imports: [HttpClientTestingModule],
      providers: [UserService]
    });
    
    service = TestBed.inject(UserService);
    httpMock = TestBed.inject(HttpTestingController);
  });
  
  afterEach(() => {
    httpMock.verify(); // Ensures no outstanding requests
  });
  
  it('should fetch users', () => {
    const mockUsers = [{ id: 1, name: 'John' }];
    
    service.getUsers().subscribe(users => {
      expect(users).toEqual(mockUsers);
    });
    
    const req = httpMock.expectOne('/api/users');
    expect(req.request.method).toBe('GET');
    req.flush(mockUsers);
  });
  
  it('should handle errors', () => {
    service.getUsers().subscribe({
      error: (err) => expect(err.status).toBe(404)
    });
    
    const req = httpMock.expectOne('/api/users');
    req.flush('Not found', { status: 404, statusText: 'Not Found' });
  });
});
```

---

### Q3: What's the difference between unit testing and E2E testing?

**Answer:**

| Aspect | Unit Testing | E2E Testing |
|--------|--------------|-------------|
| Scope | Individual units | Full application |
| Speed | Fast | Slow |
| Dependencies | Mocked | Real/simulated |
| Tools | Jasmine/Jest | Cypress/Playwright |
| Isolation | High | Low |
| Use case | Logic validation | User flow validation |

**When to use:**
- **Unit tests:** Business logic, services, pipes, utility functions
- **Integration tests:** Component interactions, HTTP calls
- **E2E tests:** Critical user journeys, smoke tests

---

### Q4: How do you handle errors in HTTP requests?

**Answer:**

```typescript
// Global error interceptor
export const errorInterceptor: HttpInterceptorFn = (req, next) => {
  return next(req).pipe(
    catchError((error: HttpErrorResponse) => {
      if (error.status === 401) {
        inject(AuthService).logout();
        inject(Router).navigate(['/login']);
      } else if (error.status === 503) {
        inject(NotificationService).showError('Service unavailable');
      }
      return throwError(() => error);
    })
  );
};

// Service-level error handling
@Injectable({ providedIn: 'root' })
export class UserService {
  getUser(id: string): Observable<User> {
    return this.http.get<User>(`/api/users/${id}`).pipe(
      retry({ count: 2, delay: 1000 }),
      catchError(this.handleError)
    );
  }
  
  private handleError(error: HttpErrorResponse): Observable<never> {
    let message = 'Unknown error occurred';
    if (error.error instanceof ErrorEvent) {
      message = error.error.message; // Client-side error
    } else {
      message = `Server error: ${error.status}`; // Server-side error
    }
    return throwError(() => new Error(message));
  }
}
```

---

### Q5: What testing patterns do you follow for Angular components?

**Answer:**

```typescript
describe('UserListComponent', () => {
  let component: UserListComponent;
  let fixture: ComponentFixture<UserListComponent>;
  let userService: jasmine.SpyObj<UserService>;
  
  beforeEach(async () => {
    const spy = jasmine.createSpyObj('UserService', ['getUsers']);
    
    await TestBed.configureTestingModule({
      imports: [UserListComponent],
      providers: [{ provide: UserService, useValue: spy }]
    }).compileComponents();
    
    userService = TestBed.inject(UserService) as jasmine.SpyObj<UserService>;
    fixture = TestBed.createComponent(UserListComponent);
    component = fixture.componentInstance;
  });
  
  it('should display users', () => {
    userService.getUsers.and.returnValue(of([{ id: 1, name: 'John' }]));
    fixture.detectChanges();
    
    const items = fixture.debugElement.queryAll(By.css('.user-item'));
    expect(items.length).toBe(1);
  });
  
  it('should show loading state', () => {
    userService.getUsers.and.returnValue(NEVER);
    fixture.detectChanges();
    
    const loader = fixture.debugElement.query(By.css('.loading'));
    expect(loader).toBeTruthy();
  });
});
```

---

### Q6: How do you implement request caching with interceptors?

**Answer:**

```typescript
const cache = new Map<string, HttpResponse<unknown>>();

export const cacheInterceptor: HttpInterceptorFn = (req, next) => {
  // Only cache GET requests
  if (req.method !== 'GET') {
    return next(req);
  }
  
  const cachedResponse = cache.get(req.url);
  if (cachedResponse) {
    return of(cachedResponse.clone());
  }
  
  return next(req).pipe(
    tap(event => {
      if (event instanceof HttpResponse) {
        cache.set(req.url, event.clone());
        
        // Clear cache after 5 minutes
        setTimeout(() => cache.delete(req.url), 5 * 60 * 1000);
      }
    })
  );
};
```

---

### Q7: How do you test components with async operations?

**Answer:**

```typescript
describe('AsyncComponent', () => {
  it('should handle async data with fakeAsync', fakeAsync(() => {
    const fixture = TestBed.createComponent(AsyncComponent);
    fixture.detectChanges();
    
    // Advance time for debounce
    tick(500);
    fixture.detectChanges();
    
    expect(fixture.nativeElement.textContent).toContain('Loaded');
  }));
  
  it('should handle async with waitForAsync', waitForAsync(() => {
    const fixture = TestBed.createComponent(AsyncComponent);
    fixture.detectChanges();
    
    fixture.whenStable().then(() => {
      fixture.detectChanges();
      expect(fixture.nativeElement.textContent).toContain('Loaded');
    });
  }));
  
  it('should handle observables', (done) => {
    service.getData().subscribe(data => {
      expect(data).toBeDefined();
      done();
    });
  });
});
```

---

### Q8: How do you implement retry logic for HTTP requests?

**Answer:**

```typescript
// Simple retry
this.http.get('/api/data').pipe(
  retry(3) // Retry 3 times immediately
);

// Retry with delay (exponential backoff)
this.http.get('/api/data').pipe(
  retry({
    count: 3,
    delay: (error, retryCount) => {
      const delay = Math.pow(2, retryCount) * 1000; // 2s, 4s, 8s
      console.log(`Retry ${retryCount} in ${delay}ms`);
      return timer(delay);
    }
  })
);

// Retry only specific errors
this.http.get('/api/data').pipe(
  retry({
    count: 3,
    delay: (error) => {
      if (error.status === 503) {
        return timer(2000); // Retry after 2s
      }
      return throwError(() => error); // Don't retry
    }
  })
);
```

---

### Q9: How do you test interceptors?

**Answer:**

```typescript
describe('AuthInterceptor', () => {
  let httpMock: HttpTestingController;
  let httpClient: HttpClient;
  let authService: jasmine.SpyObj<AuthService>;
  
  beforeEach(() => {
    const authSpy = jasmine.createSpyObj('AuthService', ['getToken']);
    
    TestBed.configureTestingModule({
      providers: [
        provideHttpClient(withInterceptors([authInterceptor])),
        provideHttpClientTesting(),
        { provide: AuthService, useValue: authSpy }
      ]
    });
    
    httpClient = TestBed.inject(HttpClient);
    httpMock = TestBed.inject(HttpTestingController);
    authService = TestBed.inject(AuthService) as jasmine.SpyObj<AuthService>;
  });
  
  it('should add auth header', () => {
    authService.getToken.and.returnValue('test-token');
    
    httpClient.get('/api/data').subscribe();
    
    const req = httpMock.expectOne('/api/data');
    expect(req.request.headers.get('Authorization')).toBe('Bearer test-token');
  });
  
  it('should not add header when no token', () => {
    authService.getToken.and.returnValue(null);
    
    httpClient.get('/api/data').subscribe();
    
    const req = httpMock.expectOne('/api/data');
    expect(req.request.headers.has('Authorization')).toBeFalse();
  });
});
```

---

### Q10: How do you implement loading indicators with interceptors?

**Answer:**

```typescript
@Injectable({ providedIn: 'root' })
export class LoadingService {
  private activeRequests = signal(0);
  isLoading = computed(() => this.activeRequests() > 0);
  
  start() { this.activeRequests.update(n => n + 1); }
  stop() { this.activeRequests.update(n => Math.max(0, n - 1)); }
}

export const loadingInterceptor: HttpInterceptorFn = (req, next) => {
  // Skip for certain requests
  if (req.headers.has('X-Skip-Loading')) {
    return next(req);
  }
  
  const loadingService = inject(LoadingService);
  loadingService.start();
  
  return next(req).pipe(
    finalize(() => loadingService.stop())
  );
};

// Usage in component
@Component({
  template: `
    <div class="loading-overlay" *ngIf="loadingService.isLoading()">
      <app-spinner></app-spinner>
    </div>
  `
})
export class AppComponent {
  loadingService = inject(LoadingService);
}
```

---

### Q11: How do you mock services in Angular tests?

**Answer:**

```typescript
// Method 1: Jasmine spy
const userServiceSpy = jasmine.createSpyObj('UserService', ['getUser', 'updateUser']);
userServiceSpy.getUser.and.returnValue(of({ id: 1, name: 'Test' }));

// Method 2: Manual mock class
class MockUserService {
  getUser = jasmine.createSpy().and.returnValue(of({ id: 1, name: 'Test' }));
}

// Method 3: Jest mock (if using Jest)
jest.mock('./user.service');
const mockUserService = UserService as jest.Mocked<typeof UserService>;

// TestBed configuration
TestBed.configureTestingModule({
  providers: [
    { provide: UserService, useValue: userServiceSpy },
    // OR
    { provide: UserService, useClass: MockUserService }
  ]
});
```

---

### Q12: How do you test components with @Input/@Output?

**Answer:**

```typescript
@Component({
  selector: 'app-counter',
  template: `
    <button (click)="decrement()">-</button>
    <span>{{ count }}</span>
    <button (click)="increment()">+</button>
  `
})
export class CounterComponent {
  @Input() count = 0;
  @Output() countChange = new EventEmitter<number>();
  
  increment() { this.countChange.emit(++this.count); }
  decrement() { this.countChange.emit(--this.count); }
}

// Tests
describe('CounterComponent', () => {
  let component: CounterComponent;
  let fixture: ComponentFixture<CounterComponent>;
  
  beforeEach(async () => {
    await TestBed.configureTestingModule({
      imports: [CounterComponent]
    }).compileComponents();
    
    fixture = TestBed.createComponent(CounterComponent);
    component = fixture.componentInstance;
  });
  
  it('should accept input', () => {
    component.count = 5;
    fixture.detectChanges();
    expect(fixture.nativeElement.querySelector('span').textContent).toBe('5');
  });
  
  it('should emit on increment', () => {
    spyOn(component.countChange, 'emit');
    component.count = 5;
    
    component.increment();
    
    expect(component.countChange.emit).toHaveBeenCalledWith(6);
  });
  
  it('should emit on button click', () => {
    let emittedValue: number | undefined;
    component.countChange.subscribe(v => emittedValue = v);
    
    const button = fixture.debugElement.queryAll(By.css('button'))[1];
    button.triggerEventHandler('click');
    
    expect(emittedValue).toBe(1);
  });
});
```

---

### Q13: How do you handle file uploads with HttpClient?

**Answer:**

```typescript
@Injectable({ providedIn: 'root' })
export class FileUploadService {
  upload(file: File): Observable<UploadProgress> {
    const formData = new FormData();
    formData.append('file', file);
    
    return this.http.post('/api/upload', formData, {
      reportProgress: true,
      observe: 'events'
    }).pipe(
      map(event => {
        switch (event.type) {
          case HttpEventType.UploadProgress:
            const progress = Math.round(100 * event.loaded / (event.total || 1));
            return { status: 'progress', progress };
          case HttpEventType.Response:
            return { status: 'complete', body: event.body };
          default:
            return { status: 'pending', progress: 0 };
        }
      }),
      catchError(error => of({ status: 'error', error }))
    );
  }
}

// Component usage
uploadFile(file: File) {
  this.uploadService.upload(file).subscribe(result => {
    if (result.status === 'progress') {
      this.progress = result.progress;
    } else if (result.status === 'complete') {
      this.uploadComplete = true;
    }
  });
}
```

---

### Q14: How do you test routing in Angular?

**Answer:**

```typescript
describe('Routing', () => {
  let router: Router;
  let location: Location;
  
  beforeEach(async () => {
    await TestBed.configureTestingModule({
      imports: [
        RouterTestingModule.withRoutes([
          { path: '', component: HomeComponent },
          { path: 'users', component: UserListComponent },
          { path: 'users/:id', component: UserDetailComponent }
        ])
      ]
    }).compileComponents();
    
    router = TestBed.inject(Router);
    location = TestBed.inject(Location);
  });
  
  it('should navigate to users', fakeAsync(() => {
    router.navigate(['/users']);
    tick();
    expect(location.path()).toBe('/users');
  }));
  
  it('should navigate with params', fakeAsync(() => {
    router.navigate(['/users', 123]);
    tick();
    expect(location.path()).toBe('/users/123');
  }));
});

// Testing guards
describe('AuthGuard', () => {
  it('should redirect to login if not authenticated', fakeAsync(() => {
    authService.isLoggedIn.and.returnValue(false);
    
    router.navigate(['/protected']);
    tick();
    
    expect(location.path()).toBe('/login');
  }));
});
```

---

### Q15: How do you implement request/response transformation?

**Answer:**

```typescript
// Transform request (add timestamp, convert to FormData)
export const requestTransformInterceptor: HttpInterceptorFn = (req, next) => {
  // Add timestamp to all requests
  const modifiedReq = req.clone({
    setHeaders: { 'X-Request-Time': Date.now().toString() }
  });
  
  return next(modifiedReq);
};

// Transform response (unwrap data, convert dates)
export const responseTransformInterceptor: HttpInterceptorFn = (req, next) => {
  return next(req).pipe(
    map(event => {
      if (event instanceof HttpResponse) {
        // Unwrap API response
        const body = event.body as any;
        if (body?.data) {
          return event.clone({ body: body.data });
        }
        
        // Convert date strings to Date objects
        return event.clone({
          body: convertDates(body)
        });
      }
      return event;
    })
  );
};

function convertDates(obj: any): any {
  if (!obj) return obj;
  for (const key of Object.keys(obj)) {
    if (typeof obj[key] === 'string' && isISODate(obj[key])) {
      obj[key] = new Date(obj[key]);
    }
  }
  return obj;
}
```

---
## Summary Checklist

âœ… **HttpClient**

- GET, POST, PUT, PATCH, DELETE
- Headers, query params, options
- File upload with progress

âœ… **Interceptors**

- Functional vs class-based
- Auth, logging, error handling
- Token refresh pattern

âœ… **Error Handling**

- Service-level catchError
- Component-level handling
- Global interceptor handling

âœ… **Testing**

- Component testing with TestBed
- Service testing with HttpTestingController
- Mocking dependencies with spies

âœ… **E2E Testing**

- Playwright/Cypress basics
- Page object pattern
- API mocking

---

**Key Takeaway:** Interceptors are essential for enterprise apps - use them for auth and error handling. Always test HTTP calls with `HttpTestingController` and verify no outstanding requests. Prefer testing user behavior over implementation details.
