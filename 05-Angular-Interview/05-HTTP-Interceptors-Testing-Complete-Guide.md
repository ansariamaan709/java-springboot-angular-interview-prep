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
import { provideHttpClient, withInterceptors } from '@angular/common/http';

bootstrapApplication(AppComponent, {
  providers: [
    provideHttpClient(
      withInterceptors([authInterceptor, loggingInterceptor])
    )
  ]
});

// NgModule approach
import { HttpClientModule } from '@angular/common/http';

@NgModule({
  imports: [HttpClientModule]
})
export class AppModule {}
```

### Basic HTTP Operations

```typescript
import { HttpClient, HttpHeaders, HttpParams } from '@angular/common/http';

@Injectable({ providedIn: 'root' })
export class UserService {
  private apiUrl = '/api/users';
  
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
@Injectable({ providedIn: 'root' })
export class ApiService {
  constructor(private http: HttpClient) {}
  
  // With headers
  getWithHeaders(): Observable<User[]> {
    const headers = new HttpHeaders({
      'Content-Type': 'application/json',
      'Authorization': 'Bearer token123',
      'X-Custom-Header': 'value'
    });
    
    return this.http.get<User[]>('/api/users', { headers });
  }
  
  // With query params
  searchUsers(term: string, page: number): Observable<User[]> {
    const params = new HttpParams()
      .set('search', term)
      .set('page', page.toString())
      .set('limit', '10');
    
    return this.http.get<User[]>('/api/users', { params });
    // URL: /api/users?search=john&page=1&limit=10
  }
  
  // Observe response (access headers, status)
  getWithFullResponse(): Observable<HttpResponse<User[]>> {
    return this.http.get<User[]>('/api/users', { 
      observe: 'response' 
    });
  }
  
  // Observe events (for upload progress)
  uploadWithProgress(file: File): Observable<HttpEvent<any>> {
    const formData = new FormData();
    formData.append('file', file);
    
    return this.http.post('/api/upload', formData, {
      observe: 'events',
      reportProgress: true
    });
  }
  
  // Response type
  downloadFile(id: string): Observable<Blob> {
    return this.http.get(`/api/files/${id}`, {
      responseType: 'blob'
    });
  }
  
  getTextFile(): Observable<string> {
    return this.http.get('/api/readme', {
      responseType: 'text'
    });
  }
}
```

### File Upload with Progress

```typescript
@Component({
  template: `
    <input type="file" (change)="onFileSelected($event)">
    <button (click)="upload()" [disabled]="!selectedFile">Upload</button>
    
    <div *ngIf="uploadProgress > 0" class="progress">
      <div [style.width.%]="uploadProgress">{{ uploadProgress }}%</div>
    </div>
  `
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
    formData.append('file', this.selectedFile);
    
    this.http.post('/api/upload', formData, {
      observe: 'events',
      reportProgress: true
    }).subscribe(event => {
      if (event.type === HttpEventType.UploadProgress) {
        this.uploadProgress = Math.round(100 * event.loaded / (event.total || 1));
      } else if (event.type === HttpEventType.Response) {
        console.log('Upload complete', event.body);
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
import { HttpErrorResponse } from '@angular/common/http';
import { throwError, retry, catchError } from 'rxjs';

@Injectable({ providedIn: 'root' })
export class UserService {
  constructor(private http: HttpClient) {}
  
  getUsers(): Observable<User[]> {
    return this.http.get<User[]>('/api/users').pipe(
      retry(2),  // Retry up to 2 times
      catchError(this.handleError)
    );
  }
  
  private handleError(error: HttpErrorResponse): Observable<never> {
    let errorMessage = 'An error occurred';
    
    if (error.status === 0) {
      // Network error
      errorMessage = 'Network error. Please check your connection.';
    } else if (error.error instanceof ErrorEvent) {
      // Client-side error
      errorMessage = `Client error: ${error.error.message}`;
    } else {
      // Server-side error
      switch (error.status) {
        case 400:
          errorMessage = error.error?.message || 'Bad request';
          break;
        case 401:
          errorMessage = 'Unauthorized. Please login again.';
          break;
        case 403:
          errorMessage = 'Forbidden. You don\'t have permission.';
          break;
        case 404:
          errorMessage = 'Resource not found.';
          break;
        case 422:
          errorMessage = 'Validation error.';
          break;
        case 500:
          errorMessage = 'Internal server error.';
          break;
        default:
          errorMessage = `Error ${error.status}: ${error.statusText}`;
      }
    }
    
    console.error('HTTP Error:', error);
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
  `
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
      }
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
┌──────────────────────────────────────────────────────────────────────────┐
│                       HTTP INTERCEPTOR CHAIN                              │
├──────────────────────────────────────────────────────────────────────────┤
│                                                                           │
│  Request Flow:                                                           │
│  ┌─────────┐    ┌─────────────┐    ┌─────────────┐    ┌──────────┐      │
│  │ Request │───▶│ Interceptor │───▶│ Interceptor │───▶│  Server  │      │
│  │         │    │     1       │    │     2       │    │          │      │
│  └─────────┘    └─────────────┘    └─────────────┘    └──────────┘      │
│                                                                           │
│  Response Flow:                                                          │
│  ┌─────────┐    ┌─────────────┐    ┌─────────────┐    ┌──────────┐      │
│  │Response │◀───│ Interceptor │◀───│ Interceptor │◀───│  Server  │      │
│  │         │    │     1       │    │     2       │    │          │      │
│  └─────────┘    └─────────────┘    └─────────────┘    └──────────┘      │
│                                                                           │
│  Common Use Cases:                                                       │
│  • Authentication (add tokens)                                           │
│  • Logging                                                               │
│  • Error handling                                                        │
│  • Caching                                                               │
│  • Loading indicators                                                    │
│                                                                           │
└──────────────────────────────────────────────────────────────────────────┘
```

### Functional Interceptors (Angular 15+)

```typescript
import { HttpInterceptorFn, HttpHandlerFn, HttpRequest } from '@angular/common/http';

// Auth interceptor
export const authInterceptor: HttpInterceptorFn = (req, next) => {
  const authService = inject(AuthService);
  const token = authService.getToken();
  
  if (token) {
    const clonedReq = req.clone({
      setHeaders: {
        Authorization: `Bearer ${token}`
      }
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
      }
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
        router.navigate(['/login']);
      }
      return throwError(() => error);
    })
  );
};

// Loading interceptor
export const loadingInterceptor: HttpInterceptorFn = (req, next) => {
  const loadingService = inject(LoadingService);
  
  loadingService.show();
  
  return next(req).pipe(
    finalize(() => loadingService.hide())
  );
};

// Register interceptors
bootstrapApplication(AppComponent, {
  providers: [
    provideHttpClient(
      withInterceptors([
        authInterceptor,
        loggingInterceptor,
        errorInterceptor,
        loadingInterceptor
      ])
    )
  ]
});
```

### Class-Based Interceptors (Traditional)

```typescript
import { HTTP_INTERCEPTORS, HttpInterceptor, HttpHandler } from '@angular/common/http';

@Injectable()
export class AuthInterceptor implements HttpInterceptor {
  constructor(private authService: AuthService) {}
  
  intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    const token = this.authService.getToken();
    
    if (token) {
      const clonedReq = req.clone({
        setHeaders: {
          Authorization: `Bearer ${token}`
        }
      });
      return next.handle(clonedReq);
    }
    
    return next.handle(req);
  }
}

@Injectable()
export class RetryInterceptor implements HttpInterceptor {
  intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    return next.handle(req).pipe(
      retry({
        count: 3,
        delay: (error, retryCount) => {
          if (error.status === 503) {  // Service unavailable
            return timer(1000 * retryCount);  // Exponential backoff
          }
          throw error;  // Don't retry other errors
        }
      })
    );
  }
}

// Register
@NgModule({
  providers: [
    { provide: HTTP_INTERCEPTORS, useClass: AuthInterceptor, multi: true },
    { provide: HTTP_INTERCEPTORS, useClass: RetryInterceptor, multi: true }
  ]
})
export class AppModule {}
```

### Token Refresh Interceptor

```typescript
export const tokenRefreshInterceptor: HttpInterceptorFn = (req, next) => {
  const authService = inject(AuthService);
  
  return next(req).pipe(
    catchError((error: HttpErrorResponse) => {
      if (error.status === 401 && !req.url.includes('/refresh')) {
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

---

## Caching Strategies

### Simple Caching Service

```typescript
@Injectable({ providedIn: 'root' })
export class CacheService {
  private cache = new Map<string, { data: any; timestamp: number }>();
  private defaultTTL = 5 * 60 * 1000;  // 5 minutes
  
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
@Injectable({ providedIn: 'root' })
export class UserService {
  constructor(
    private http: HttpClient,
    private cache: CacheService
  ) {}
  
  getUsers(): Observable<User[]> {
    const cached = this.cache.get<User[]>('users');
    if (cached) {
      return of(cached);
    }
    
    return this.http.get<User[]>('/api/users').pipe(
      tap(users => this.cache.set('users', users))
    );
  }
}
```

### Caching with shareReplay

```typescript
@Injectable({ providedIn: 'root' })
export class ConfigService {
  private config$: Observable<Config> | null = null;
  
  constructor(private http: HttpClient) {}
  
  getConfig(): Observable<Config> {
    if (!this.config$) {
      this.config$ = this.http.get<Config>('/api/config').pipe(
        shareReplay(1)  // Cache last emitted value
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
const CACHEABLE_URLS = ['/api/config', '/api/categories', '/api/countries'];

export const cacheInterceptor: HttpInterceptorFn = (req, next) => {
  // Only cache GET requests
  if (req.method !== 'GET') {
    return next(req);
  }
  
  // Check if URL is cacheable
  if (!CACHEABLE_URLS.some(url => req.url.includes(url))) {
    return next(req);
  }
  
  const cacheService = inject(CacheService);
  const cached = cacheService.get(req.url);
  
  if (cached) {
    return of(new HttpResponse({ body: cached }));
  }
  
  return next(req).pipe(
    tap(event => {
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
import { ComponentFixture, TestBed, fakeAsync, tick } from '@angular/core/testing';
import { By } from '@angular/platform-browser';

describe('UserListComponent', () => {
  let component: UserListComponent;
  let fixture: ComponentFixture<UserListComponent>;
  let userServiceSpy: jasmine.SpyObj<UserService>;
  
  beforeEach(async () => {
    // Create spy object
    const spy = jasmine.createSpyObj('UserService', ['getUsers', 'deleteUser']);
    
    await TestBed.configureTestingModule({
      imports: [UserListComponent],  // Standalone component
      providers: [
        { provide: UserService, useValue: spy }
      ]
    }).compileComponents();
    
    fixture = TestBed.createComponent(UserListComponent);
    component = fixture.componentInstance;
    userServiceSpy = TestBed.inject(UserService) as jasmine.SpyObj<UserService>;
  });
  
  it('should create', () => {
    expect(component).toBeTruthy();
  });
  
  it('should load users on init', () => {
    const mockUsers: User[] = [
      { id: 1, name: 'John' },
      { id: 2, name: 'Jane' }
    ];
    userServiceSpy.getUsers.and.returnValue(of(mockUsers));
    
    fixture.detectChanges();  // Triggers ngOnInit
    
    expect(userServiceSpy.getUsers).toHaveBeenCalled();
    expect(component.users).toEqual(mockUsers);
  });
  
  it('should display users in template', () => {
    const mockUsers: User[] = [
      { id: 1, name: 'John' },
      { id: 2, name: 'Jane' }
    ];
    userServiceSpy.getUsers.and.returnValue(of(mockUsers));
    
    fixture.detectChanges();
    
    const userElements = fixture.debugElement.queryAll(By.css('.user-item'));
    expect(userElements.length).toBe(2);
    expect(userElements[0].nativeElement.textContent).toContain('John');
  });
  
  it('should show loading indicator', () => {
    userServiceSpy.getUsers.and.returnValue(of([]).pipe(delay(100)));
    
    fixture.detectChanges();
    
    const loading = fixture.debugElement.query(By.css('.loading'));
    expect(loading).toBeTruthy();
  });
  
  it('should handle error', () => {
    userServiceSpy.getUsers.and.returnValue(
      throwError(() => new Error('Server error'))
    );
    
    fixture.detectChanges();
    
    expect(component.error).toBe('Server error');
    const errorEl = fixture.debugElement.query(By.css('.error'));
    expect(errorEl.nativeElement.textContent).toContain('Server error');
  });
});
```

### Testing Async Operations

```typescript
describe('AsyncComponent', () => {
  let component: AsyncComponent;
  let fixture: ComponentFixture<AsyncComponent>;
  
  beforeEach(async () => {
    await TestBed.configureTestingModule({
      imports: [AsyncComponent]
    }).compileComponents();
    
    fixture = TestBed.createComponent(AsyncComponent);
    component = fixture.componentInstance;
  });
  
  // Using fakeAsync + tick
  it('should update after delay (fakeAsync)', fakeAsync(() => {
    component.delayedAction();
    
    expect(component.status).toBe('pending');
    
    tick(1000);  // Advance time by 1 second
    
    expect(component.status).toBe('complete');
  }));
  
  // Using async/await with fixture.whenStable()
  it('should update after async operation (async)', async () => {
    component.asyncAction();
    
    await fixture.whenStable();
    fixture.detectChanges();
    
    expect(component.result).toBeDefined();
  });
  
  // Using done callback
  it('should emit value (done callback)', (done) => {
    component.output.subscribe(value => {
      expect(value).toBe('emitted');
      done();
    });
    
    component.emitValue();
  });
});
```

### Testing Input/Output

```typescript
@Component({
  selector: 'app-counter',
  template: `
    <button (click)="decrement()">-</button>
    <span class="count">{{ count }}</span>
    <button (click)="increment()">+</button>
  `
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
    component.count = 10;
    fixture.detectChanges();
    
    const countEl = fixture.debugElement.query(By.css('.count'));
    expect(countEl.nativeElement.textContent).toContain('10');
  });
  
  it('should emit on increment', () => {
    spyOn(component.countChange, 'emit');
    
    component.count = 5;
    component.increment();
    
    expect(component.countChange.emit).toHaveBeenCalledWith(6);
  });
  
  it('should increment on button click', () => {
    component.count = 0;
    fixture.detectChanges();
    
    const incrementBtn = fixture.debugElement.queryAll(By.css('button'))[1];
    incrementBtn.triggerEventHandler('click', null);
    
    expect(component.count).toBe(1);
  });
});
```

---

## Testing Services

### Basic Service Testing

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
    // Verify no outstanding requests
    httpMock.verify();
  });
  
  it('should fetch users', () => {
    const mockUsers: User[] = [
      { id: 1, name: 'John' },
      { id: 2, name: 'Jane' }
    ];
    
    service.getUsers().subscribe(users => {
      expect(users.length).toBe(2);
      expect(users).toEqual(mockUsers);
    });
    
    const req = httpMock.expectOne('/api/users');
    expect(req.request.method).toBe('GET');
    req.flush(mockUsers);
  });
  
  it('should create user', () => {
    const newUser = { name: 'Bob', email: 'bob@test.com' };
    const createdUser = { id: 3, ...newUser };
    
    service.createUser(newUser).subscribe(user => {
      expect(user).toEqual(createdUser);
    });
    
    const req = httpMock.expectOne('/api/users');
    expect(req.request.method).toBe('POST');
    expect(req.request.body).toEqual(newUser);
    req.flush(createdUser);
  });
  
  it('should handle error', () => {
    service.getUsers().subscribe({
      next: () => fail('should have failed'),
      error: (error) => {
        expect(error.message).toContain('Server error');
      }
    });
    
    const req = httpMock.expectOne('/api/users');
    req.flush('Server error', { status: 500, statusText: 'Server Error' });
  });
});
```

### Testing with Dependencies

```typescript
describe('AuthService', () => {
  let service: AuthService;
  let httpMock: HttpTestingController;
  let routerSpy: jasmine.SpyObj<Router>;
  let storageSpy: jasmine.SpyObj<StorageService>;
  
  beforeEach(() => {
    routerSpy = jasmine.createSpyObj('Router', ['navigate']);
    storageSpy = jasmine.createSpyObj('StorageService', ['get', 'set', 'remove']);
    
    TestBed.configureTestingModule({
      imports: [HttpClientTestingModule],
      providers: [
        AuthService,
        { provide: Router, useValue: routerSpy },
        { provide: StorageService, useValue: storageSpy }
      ]
    });
    
    service = TestBed.inject(AuthService);
    httpMock = TestBed.inject(HttpTestingController);
  });
  
  it('should login and store token', () => {
    const credentials = { email: 'test@test.com', password: 'password' };
    const response = { token: 'jwt-token', user: { id: 1, name: 'Test' } };
    
    service.login(credentials).subscribe(result => {
      expect(result.token).toBe('jwt-token');
    });
    
    const req = httpMock.expectOne('/api/auth/login');
    req.flush(response);
    
    expect(storageSpy.set).toHaveBeenCalledWith('token', 'jwt-token');
  });
  
  it('should logout and navigate to login', () => {
    service.logout();
    
    expect(storageSpy.remove).toHaveBeenCalledWith('token');
    expect(routerSpy.navigate).toHaveBeenCalledWith(['/login']);
  });
});
```

---

## Testing HTTP Calls

### HttpTestingController

```typescript
import { HttpClientTestingModule, HttpTestingController } from '@angular/common/http/testing';

describe('ApiService', () => {
  let service: ApiService;
  let httpMock: HttpTestingController;
  
  beforeEach(() => {
    TestBed.configureTestingModule({
      imports: [HttpClientTestingModule],
      providers: [ApiService]
    });
    
    service = TestBed.inject(ApiService);
    httpMock = TestBed.inject(HttpTestingController);
  });
  
  afterEach(() => {
    httpMock.verify();  // Ensure no outstanding requests
  });
  
  // Test with query params
  it('should search with params', () => {
    service.search('test', 1, 10).subscribe();
    
    const req = httpMock.expectOne(
      (request) => request.url === '/api/search' &&
                   request.params.get('q') === 'test' &&
                   request.params.get('page') === '1'
    );
    
    expect(req.request.method).toBe('GET');
    req.flush({ results: [] });
  });
  
  // Test with headers
  it('should send custom headers', () => {
    service.fetchWithAuth().subscribe();
    
    const req = httpMock.expectOne('/api/protected');
    expect(req.request.headers.get('Authorization')).toBeTruthy();
    req.flush({});
  });
  
  // Test multiple requests
  it('should handle multiple requests', () => {
    service.getUsers().subscribe();
    service.getProducts().subscribe();
    
    const requests = httpMock.match((req) => true);
    expect(requests.length).toBe(2);
    
    requests.forEach(req => req.flush([]));
  });
  
  // Test error responses
  it('should handle 404', () => {
    service.getUser(999).subscribe({
      error: (error) => {
        expect(error.status).toBe(404);
      }
    });
    
    const req = httpMock.expectOne('/api/users/999');
    req.flush('Not found', { status: 404, statusText: 'Not Found' });
  });
  
  // Test network error
  it('should handle network error', () => {
    service.getUsers().subscribe({
      error: (error) => {
        expect(error.message).toContain('Network');
      }
    });
    
    const req = httpMock.expectOne('/api/users');
    req.error(new ProgressEvent('Network error'));
  });
});
```

### Testing Interceptors

```typescript
describe('AuthInterceptor', () => {
  let httpMock: HttpTestingController;
  let http: HttpClient;
  let authServiceSpy: jasmine.SpyObj<AuthService>;
  
  beforeEach(() => {
    authServiceSpy = jasmine.createSpyObj('AuthService', ['getToken']);
    
    TestBed.configureTestingModule({
      providers: [
        provideHttpClient(withInterceptors([authInterceptor])),
        provideHttpClientTesting(),
        { provide: AuthService, useValue: authServiceSpy }
      ]
    });
    
    http = TestBed.inject(HttpClient);
    httpMock = TestBed.inject(HttpTestingController);
  });
  
  it('should add auth header when token exists', () => {
    authServiceSpy.getToken.and.returnValue('test-token');
    
    http.get('/api/data').subscribe();
    
    const req = httpMock.expectOne('/api/data');
    expect(req.request.headers.get('Authorization')).toBe('Bearer test-token');
    req.flush({});
  });
  
  it('should not add header when no token', () => {
    authServiceSpy.getToken.and.returnValue(null);
    
    http.get('/api/data').subscribe();
    
    const req = httpMock.expectOne('/api/data');
    expect(req.request.headers.has('Authorization')).toBeFalse();
    req.flush({});
  });
});
```

---

## Component Testing Deep Dive

### Testing with Router

```typescript
import { RouterTestingModule } from '@angular/router/testing';
import { Router } from '@angular/router';

describe('NavigationComponent', () => {
  let component: NavigationComponent;
  let fixture: ComponentFixture<NavigationComponent>;
  let router: Router;
  
  beforeEach(async () => {
    await TestBed.configureTestingModule({
      imports: [
        NavigationComponent,
        RouterTestingModule.withRoutes([
          { path: 'home', component: DummyComponent },
          { path: 'users', component: DummyComponent }
        ])
      ]
    }).compileComponents();
    
    fixture = TestBed.createComponent(NavigationComponent);
    component = fixture.componentInstance;
    router = TestBed.inject(Router);
    
    fixture.detectChanges();
  });
  
  it('should navigate to users', fakeAsync(() => {
    const navigateSpy = spyOn(router, 'navigate');
    
    component.goToUsers();
    
    expect(navigateSpy).toHaveBeenCalledWith(['/users']);
  }));
  
  it('should navigate via routerLink', fakeAsync(() => {
    const link = fixture.debugElement.query(By.css('a[routerLink="/users"]'));
    link.nativeElement.click();
    
    tick();
    
    expect(router.url).toBe('/users');
  }));
});
```

### Testing Forms

```typescript
describe('LoginFormComponent', () => {
  let component: LoginFormComponent;
  let fixture: ComponentFixture<LoginFormComponent>;
  
  beforeEach(async () => {
    await TestBed.configureTestingModule({
      imports: [LoginFormComponent, ReactiveFormsModule]
    }).compileComponents();
    
    fixture = TestBed.createComponent(LoginFormComponent);
    component = fixture.componentInstance;
    fixture.detectChanges();
  });
  
  it('should create form with controls', () => {
    expect(component.loginForm.contains('email')).toBeTruthy();
    expect(component.loginForm.contains('password')).toBeTruthy();
  });
  
  it('should require email', () => {
    const email = component.loginForm.get('email');
    email?.setValue('');
    
    expect(email?.valid).toBeFalsy();
    expect(email?.errors?.['required']).toBeTruthy();
  });
  
  it('should validate email format', () => {
    const email = component.loginForm.get('email');
    email?.setValue('invalid-email');
    
    expect(email?.errors?.['email']).toBeTruthy();
  });
  
  it('should be valid with correct data', () => {
    component.loginForm.patchValue({
      email: 'test@test.com',
      password: 'password123'
    });
    
    expect(component.loginForm.valid).toBeTruthy();
  });
  
  it('should submit form', () => {
    spyOn(component, 'onSubmit');
    
    component.loginForm.patchValue({
      email: 'test@test.com',
      password: 'password123'
    });
    
    const form = fixture.debugElement.query(By.css('form'));
    form.triggerEventHandler('ngSubmit', null);
    
    expect(component.onSubmit).toHaveBeenCalled();
  });
  
  it('should disable submit button when invalid', () => {
    const submitBtn = fixture.debugElement.query(By.css('button[type="submit"]'));
    
    expect(submitBtn.nativeElement.disabled).toBeTruthy();
    
    component.loginForm.patchValue({
      email: 'test@test.com',
      password: 'password123'
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
import { test, expect } from '@playwright/test';

test.describe('Login Page', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('/login');
  });
  
  test('should display login form', async ({ page }) => {
    await expect(page.locator('h1')).toHaveText('Login');
    await expect(page.locator('input[name="email"]')).toBeVisible();
    await expect(page.locator('input[name="password"]')).toBeVisible();
  });
  
  test('should show validation errors', async ({ page }) => {
    await page.click('button[type="submit"]');
    
    await expect(page.locator('.error')).toContainText('Email is required');
  });
  
  test('should login successfully', async ({ page }) => {
    await page.fill('input[name="email"]', 'user@test.com');
    await page.fill('input[name="password"]', 'password123');
    await page.click('button[type="submit"]');
    
    await expect(page).toHaveURL('/dashboard');
    await expect(page.locator('.welcome')).toContainText('Welcome');
  });
  
  test('should handle invalid credentials', async ({ page }) => {
    await page.fill('input[name="email"]', 'wrong@test.com');
    await page.fill('input[name="password"]', 'wrongpassword');
    await page.click('button[type="submit"]');
    
    await expect(page.locator('.error-message')).toContainText('Invalid credentials');
  });
});
```

### Cypress Alternative

```typescript
// cypress/e2e/login.cy.ts
describe('Login Page', () => {
  beforeEach(() => {
    cy.visit('/login');
  });
  
  it('should login successfully', () => {
    cy.get('input[name="email"]').type('user@test.com');
    cy.get('input[name="password"]').type('password123');
    cy.get('button[type="submit"]').click();
    
    cy.url().should('include', '/dashboard');
    cy.contains('Welcome').should('be.visible');
  });
  
  it('should intercept API calls', () => {
    cy.intercept('POST', '/api/auth/login', {
      statusCode: 200,
      body: { token: 'fake-token', user: { name: 'Test' } }
    }).as('loginRequest');
    
    cy.get('input[name="email"]').type('user@test.com');
    cy.get('input[name="password"]').type('password123');
    cy.get('button[type="submit"]').click();
    
    cy.wait('@loginRequest');
    cy.url().should('include', '/dashboard');
  });
});
```

---

## Interview Questions & Answers

### Q1: What are HTTP Interceptors and when would you use them?

**Answer:**

HTTP Interceptors are middleware that intercept HTTP requests/responses. They're used for:

- **Authentication**: Add auth tokens to requests
- **Logging**: Log requests/responses
- **Error Handling**: Global error handling
- **Caching**: Cache responses
- **Loading Indicators**: Show/hide loading

```typescript
export const authInterceptor: HttpInterceptorFn = (req, next) => {
  const token = inject(AuthService).getToken();
  
  if (token) {
    req = req.clone({
      setHeaders: { Authorization: `Bearer ${token}` }
    });
  }
  
  return next(req);
};
```

**Key points:**
- Requests are immutable (use `clone()`)
- Multiple interceptors run in order
- Can modify request AND response
- Must call `next()` to continue chain

---

### Q2: How do you test HTTP calls in Angular?

**Answer:**

Use `HttpClientTestingModule` with `HttpTestingController`:

```typescript
beforeEach(() => {
  TestBed.configureTestingModule({
    imports: [HttpClientTestingModule],
    providers: [MyService]
  });
  httpMock = TestBed.inject(HttpTestingController);
});

it('should fetch data', () => {
  service.getData().subscribe(data => {
    expect(data).toEqual(mockData);
  });
  
  const req = httpMock.expectOne('/api/data');
  expect(req.request.method).toBe('GET');
  req.flush(mockData);  // Provide mock response
});

afterEach(() => {
  httpMock.verify();  // Ensure no outstanding requests
});
```

**Key methods:**
- `expectOne()` - Expect single request
- `expectNone()` - Expect no requests
- `match()` - Match multiple requests
- `flush()` - Provide response
- `verify()` - Verify no outstanding requests

---

### Q3: What's the difference between unit testing and E2E testing?

**Answer:**

| Aspect | Unit Tests | E2E Tests |
|--------|-----------|-----------|
| **Scope** | Single unit (component/service) | Full application flow |
| **Speed** | Fast | Slow |
| **Dependencies** | Mocked | Real |
| **Tools** | Jasmine/Jest, Karma | Playwright, Cypress |
| **Purpose** | Test isolated logic | Test user journeys |
| **When** | Every commit | Before release |

```typescript
// Unit test - Mock everything
it('should show user name', () => {
  userServiceSpy.getUser.and.returnValue(of({ name: 'John' }));
  fixture.detectChanges();
  expect(component.userName).toBe('John');
});

// E2E test - Real browser
test('should display user profile', async ({ page }) => {
  await page.goto('/profile');
  await expect(page.locator('.user-name')).toHaveText('John');
});
```

---

### Q4: How do you handle errors in HTTP requests?

**Answer:**

**Service Level:**
```typescript
getUsers(): Observable<User[]> {
  return this.http.get<User[]>('/api/users').pipe(
    retry(2),
    catchError(this.handleError)
  );
}

private handleError(error: HttpErrorResponse): Observable<never> {
  let message = 'An error occurred';
  
  if (error.status === 0) {
    message = 'Network error';
  } else if (error.status === 401) {
    message = 'Unauthorized';
  } else if (error.status === 500) {
    message = 'Server error';
  }
  
  return throwError(() => new Error(message));
}
```

**Component Level:**
```typescript
loadUsers() {
  this.loading = true;
  this.error = null;
  
  this.userService.getUsers().subscribe({
    next: (users) => { this.users = users; this.loading = false; },
    error: (err) => { this.error = err.message; this.loading = false; }
  });
}
```

**Global Level (Interceptor):**
```typescript
export const errorInterceptor: HttpInterceptorFn = (req, next) => {
  return next(req).pipe(
    catchError((error: HttpErrorResponse) => {
      if (error.status === 401) {
        inject(AuthService).logout();
      }
      return throwError(() => error);
    })
  );
};
```

---

### Q5: What testing patterns do you follow for Angular components?

**Answer:**

**1. Arrange-Act-Assert (AAA):**
```typescript
it('should increment count', () => {
  // Arrange
  component.count = 0;
  
  // Act
  component.increment();
  
  // Assert
  expect(component.count).toBe(1);
});
```

**2. Test user behavior, not implementation:**
```typescript
// Bad - tests implementation
it('should call service method', () => {
  component.loadUsers();
  expect(serviceSpy.getUsers).toHaveBeenCalled();
});

// Good - tests behavior
it('should display users after loading', () => {
  serviceSpy.getUsers.and.returnValue(of(mockUsers));
  fixture.detectChanges();
  
  const userElements = fixture.debugElement.queryAll(By.css('.user'));
  expect(userElements.length).toBe(mockUsers.length);
});
```

**3. Test edge cases:**
```typescript
it('should handle empty list', () => {
  serviceSpy.getUsers.and.returnValue(of([]));
  fixture.detectChanges();
  
  expect(fixture.debugElement.query(By.css('.empty-state'))).toBeTruthy();
});

it('should handle error', () => {
  serviceSpy.getUsers.and.returnValue(throwError(() => new Error('Failed')));
  fixture.detectChanges();
  
  expect(component.error).toBeTruthy();
});
```

---

## Summary Checklist

✅ **HttpClient**
- GET, POST, PUT, PATCH, DELETE
- Headers, query params, options
- File upload with progress

✅ **Interceptors**
- Functional vs class-based
- Auth, logging, error handling
- Token refresh pattern

✅ **Error Handling**
- Service-level catchError
- Component-level handling
- Global interceptor handling

✅ **Testing**
- Component testing with TestBed
- Service testing with HttpTestingController
- Mocking dependencies with spies

✅ **E2E Testing**
- Playwright/Cypress basics
- Page object pattern
- API mocking

---

**Key Takeaway:** Interceptors are essential for enterprise apps - use them for auth and error handling. Always test HTTP calls with `HttpTestingController` and verify no outstanding requests. Prefer testing user behavior over implementation details.
