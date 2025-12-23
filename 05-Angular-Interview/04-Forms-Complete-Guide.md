# Angular Forms - Complete Interview Guide

## Table of Contents

1. [Forms Overview](#forms-overview)
2. [Template-Driven Forms](#template-driven-forms)
3. [Reactive Forms](#reactive-forms)
4. [Form Validation](#form-validation)
5. [Custom Validators](#custom-validators)
6. [Async Validators](#async-validators)
7. [Dynamic Forms](#dynamic-forms)
8. [Form Arrays](#form-arrays)
9. [Typed Forms (Angular 14+)](#typed-forms-angular-14)
10. [Best Practices](#best-practices)
11. [Interview Questions & Answers](#interview-questions--answers)

---

## Forms Overview

### Template-Driven vs Reactive Forms

```
┌──────────────────────────────────────────────────────────────────────────┐
│              TEMPLATE-DRIVEN vs REACTIVE FORMS                           │
├──────────────────────────────────────────────────────────────────────────┤
│                                                                           │
│  TEMPLATE-DRIVEN                    │  REACTIVE                          │
│  ─────────────────                  │  ────────                          │
│                                     │                                     │
│  • Form model in template           │  • Form model in component class   │
│  • Two-way binding (ngModel)        │  • Programmatic control            │
│  • Less code for simple forms       │  • More testable                   │
│  • Async (relies on change det.)    │  • Synchronous                     │
│  • Hard to unit test                │  • Easy to unit test               │
│  • Good for simple forms            │  • Good for complex forms          │
│                                     │                                     │
│  FormsModule                        │  ReactiveFormsModule               │
│                                     │                                     │
│  <input [(ngModel)]="name">         │  <input [formControl]="nameCtrl">  │
│                                     │                                     │
└──────────────────────────────────────────────────────────────────────────┘
```

### When to Use Which

| Use Case | Recommended |
|----------|-------------|
| Simple login/contact forms | Template-Driven |
| Complex forms with validation | Reactive |
| Dynamic form fields | Reactive |
| Forms requiring unit tests | Reactive |
| Quick prototypes | Template-Driven |
| Enterprise applications | Reactive |

---

## Template-Driven Forms

### Basic Setup

```typescript
// Import FormsModule
import { FormsModule } from '@angular/forms';

@Component({
  standalone: true,
  imports: [FormsModule, CommonModule],
  template: `
    <form #loginForm="ngForm" (ngSubmit)="onSubmit(loginForm)">
      <div>
        <label for="email">Email:</label>
        <input 
          type="email" 
          id="email"
          name="email" 
          [(ngModel)]="user.email"
          required
          email
          #emailInput="ngModel"
        >
        <div *ngIf="emailInput.invalid && emailInput.touched" class="error">
          <span *ngIf="emailInput.errors?.['required']">Email is required</span>
          <span *ngIf="emailInput.errors?.['email']">Invalid email format</span>
        </div>
      </div>
      
      <div>
        <label for="password">Password:</label>
        <input 
          type="password" 
          id="password"
          name="password" 
          [(ngModel)]="user.password"
          required
          minlength="8"
          #passwordInput="ngModel"
        >
        <div *ngIf="passwordInput.invalid && passwordInput.touched" class="error">
          <span *ngIf="passwordInput.errors?.['required']">Password required</span>
          <span *ngIf="passwordInput.errors?.['minlength']">
            Min {{ passwordInput.errors?.['minlength'].requiredLength }} characters
          </span>
        </div>
      </div>
      
      <button type="submit" [disabled]="loginForm.invalid">Login</button>
      
      <!-- Debug -->
      <pre>Form Valid: {{ loginForm.valid }}</pre>
      <pre>Form Values: {{ loginForm.value | json }}</pre>
    </form>
  `
})
export class LoginComponent {
  user = {
    email: '',
    password: ''
  };
  
  onSubmit(form: NgForm) {
    if (form.valid) {
      console.log('Form submitted:', this.user);
    }
  }
}
```

### ngModel Directives

```html
<!-- Standalone ngModel (no form) -->
<input [(ngModel)]="searchTerm">

<!-- Within ngForm -->
<form #myForm="ngForm">
  <!-- Two-way binding with name attribute -->
  <input name="firstName" [(ngModel)]="person.firstName">
  
  <!-- ngModelGroup for nested objects -->
  <div ngModelGroup="address">
    <input name="street" [(ngModel)]="person.address.street">
    <input name="city" [(ngModel)]="person.address.city">
  </div>
</form>

<!-- Access form data -->
{{ myForm.value | json }}
<!-- Output: { firstName: 'John', address: { street: '123 Main', city: 'NYC' } } -->
```

### Template-Driven Validation

```html
<!-- Built-in validators -->
<input 
  name="username"
  [(ngModel)]="username"
  required                    <!-- Required field -->
  minlength="3"              <!-- Min length -->
  maxlength="20"             <!-- Max length -->
  pattern="[a-zA-Z0-9]+"     <!-- Regex pattern -->
  email                       <!-- Email format -->
  #usernameCtrl="ngModel"
>

<!-- Conditional validation -->
<input 
  name="phone"
  [(ngModel)]="phone"
  [required]="requirePhone"   <!-- Dynamic required -->
>

<!-- Custom validator directive -->
<input 
  name="password"
  [(ngModel)]="password"
  appPasswordStrength         <!-- Custom directive -->
>
```

---

## Reactive Forms

### Basic Setup

```typescript
import { ReactiveFormsModule, FormBuilder, FormGroup, Validators } from '@angular/forms';

@Component({
  standalone: true,
  imports: [ReactiveFormsModule, CommonModule],
  template: `
    <form [formGroup]="loginForm" (ngSubmit)="onSubmit()">
      <div>
        <label for="email">Email:</label>
        <input id="email" type="email" formControlName="email">
        <div *ngIf="email.invalid && email.touched" class="error">
          <span *ngIf="email.errors?.['required']">Email is required</span>
          <span *ngIf="email.errors?.['email']">Invalid email</span>
        </div>
      </div>
      
      <div>
        <label for="password">Password:</label>
        <input id="password" type="password" formControlName="password">
        <div *ngIf="password.invalid && password.touched" class="error">
          <span *ngIf="password.errors?.['required']">Password required</span>
          <span *ngIf="password.errors?.['minlength']">
            Min {{ password.errors?.['minlength'].requiredLength }} chars
          </span>
        </div>
      </div>
      
      <button type="submit" [disabled]="loginForm.invalid">Login</button>
    </form>
  `
})
export class LoginComponent implements OnInit {
  loginForm!: FormGroup;
  
  constructor(private fb: FormBuilder) {}
  
  ngOnInit() {
    this.loginForm = this.fb.group({
      email: ['', [Validators.required, Validators.email]],
      password: ['', [Validators.required, Validators.minLength(8)]]
    });
  }
  
  // Getters for easy access
  get email() { return this.loginForm.get('email')!; }
  get password() { return this.loginForm.get('password')!; }
  
  onSubmit() {
    if (this.loginForm.valid) {
      console.log('Form data:', this.loginForm.value);
    } else {
      // Mark all as touched to show errors
      this.loginForm.markAllAsTouched();
    }
  }
}
```

### FormControl, FormGroup, FormArray

```typescript
import { FormControl, FormGroup, FormArray, FormBuilder, Validators } from '@angular/forms';

// Individual FormControl
const nameControl = new FormControl('John', Validators.required);
console.log(nameControl.value);  // 'John'
console.log(nameControl.valid);  // true

// FormGroup
const userForm = new FormGroup({
  firstName: new FormControl('', Validators.required),
  lastName: new FormControl(''),
  email: new FormControl('', [Validators.required, Validators.email])
});

// Nested FormGroup
const profileForm = new FormGroup({
  name: new FormGroup({
    first: new FormControl(''),
    last: new FormControl('')
  }),
  email: new FormControl('')
});

// FormArray
const phoneNumbers = new FormArray([
  new FormControl('555-1234'),
  new FormControl('555-5678')
]);

// Using FormBuilder (recommended)
const form = this.fb.group({
  name: this.fb.group({
    first: ['', Validators.required],
    last: ['']
  }),
  email: ['', [Validators.required, Validators.email]],
  phones: this.fb.array([
    [''],
    ['']
  ])
});
```

### FormControl Methods

```typescript
const control = new FormControl('initial value');

// Value operations
control.setValue('new value');           // Set value (strict)
control.patchValue('partial');           // Patch value (lenient)
control.reset();                         // Reset to initial
control.reset('new initial');            // Reset with new initial

// State checks
control.valid;                           // Is valid?
control.invalid;                         // Is invalid?
control.pending;                         // Async validation pending?
control.disabled;                        // Is disabled?
control.enabled;                         // Is enabled?
control.pristine;                        // Not modified?
control.dirty;                           // Modified?
control.touched;                         // Blurred?
control.untouched;                       // Never blurred?

// State changes
control.markAsTouched();                 // Mark as touched
control.markAsUntouched();
control.markAsDirty();                   // Mark as dirty
control.markAsPristine();
control.disable();                       // Disable
control.enable();                        // Enable

// Validators
control.setValidators([Validators.required, Validators.min(0)]);
control.clearValidators();
control.updateValueAndValidity();        // Re-run validation

// Observables
control.valueChanges.subscribe(value => console.log(value));
control.statusChanges.subscribe(status => console.log(status));  // 'VALID' | 'INVALID' | 'PENDING' | 'DISABLED'
```

### FormGroup Methods

```typescript
const form = this.fb.group({
  name: ['John'],
  email: ['john@example.com'],
  address: this.fb.group({
    street: ['123 Main St'],
    city: ['NYC']
  })
});

// Value operations
form.setValue({                          // Must include ALL controls
  name: 'Jane',
  email: 'jane@example.com',
  address: { street: '456 Oak', city: 'LA' }
});

form.patchValue({                        // Can be partial
  name: 'Bob'
});

form.reset();                            // Reset all controls
form.reset({ name: 'Default' });         // Reset with values

// Access controls
form.get('name');                        // Get control
form.get('address.street');              // Nested access
form.controls['name'];                   // Direct access

// Add/remove controls dynamically
form.addControl('phone', new FormControl(''));
form.removeControl('phone');
form.setControl('email', new FormControl('new@email.com'));
form.contains('phone');                  // Check if exists

// Get value
form.value;                              // { name: 'John', email: '...', address: {...} }
form.getRawValue();                      // Includes disabled controls
```

---

## Form Validation

### Built-in Validators

```typescript
import { Validators } from '@angular/forms';

const form = this.fb.group({
  // Required
  name: ['', Validators.required],
  
  // Multiple validators
  email: ['', [
    Validators.required,
    Validators.email
  ]],
  
  // Length validators
  username: ['', [
    Validators.required,
    Validators.minLength(3),
    Validators.maxLength(20)
  ]],
  
  // Number validators
  age: [null, [
    Validators.required,
    Validators.min(18),
    Validators.max(120)
  ]],
  
  // Pattern validator
  phone: ['', Validators.pattern(/^\d{3}-\d{3}-\d{4}$/)],
  
  // Composed validators
  website: ['', Validators.compose([
    Validators.required,
    Validators.pattern(/^https?:\/\//)
  ])],
  
  // Required true (for checkboxes)
  acceptTerms: [false, Validators.requiredTrue]
});
```

### Displaying Validation Errors

```typescript
@Component({
  template: `
    <form [formGroup]="form">
      <div class="form-field">
        <label>Email</label>
        <input formControlName="email" type="email">
        
        <!-- Method 1: Individual error checks -->
        <div *ngIf="email.invalid && email.touched" class="errors">
          <div *ngIf="email.errors?.['required']">Email is required</div>
          <div *ngIf="email.errors?.['email']">Invalid email format</div>
        </div>
        
        <!-- Method 2: Using helper method -->
        <div *ngIf="getErrorMessage('email')" class="error">
          {{ getErrorMessage('email') }}
        </div>
      </div>
      
      <!-- Method 3: Error component -->
      <app-field-errors [control]="email"></app-field-errors>
    </form>
  `
})
export class FormComponent {
  form = this.fb.group({
    email: ['', [Validators.required, Validators.email]]
  });
  
  get email() { return this.form.get('email')!; }
  
  getErrorMessage(controlName: string): string | null {
    const control = this.form.get(controlName);
    if (!control || !control.errors || !control.touched) return null;
    
    const errorMessages: Record<string, string> = {
      required: 'This field is required',
      email: 'Please enter a valid email',
      minlength: `Minimum length is ${control.errors['minlength']?.requiredLength}`,
      maxlength: `Maximum length is ${control.errors['maxlength']?.requiredLength}`,
      min: `Minimum value is ${control.errors['min']?.min}`,
      max: `Maximum value is ${control.errors['max']?.max}`,
      pattern: 'Invalid format'
    };
    
    const firstError = Object.keys(control.errors)[0];
    return errorMessages[firstError] || 'Invalid value';
  }
}
```

### CSS Classes

```css
/* Angular automatically adds these classes */
.ng-valid { }      /* Control is valid */
.ng-invalid { }    /* Control is invalid */
.ng-pending { }    /* Async validation in progress */
.ng-pristine { }   /* Control not modified */
.ng-dirty { }      /* Control modified */
.ng-untouched { }  /* Control not blurred */
.ng-touched { }    /* Control has been blurred */

/* Common styling patterns */
input.ng-invalid.ng-touched {
  border-color: red;
}

input.ng-valid.ng-dirty {
  border-color: green;
}

.error-message {
  color: red;
  font-size: 0.875rem;
}
```

---

## Custom Validators

### Synchronous Validators

```typescript
import { AbstractControl, ValidationErrors, ValidatorFn } from '@angular/forms';

// Validator function
export function forbiddenNameValidator(forbidden: string): ValidatorFn {
  return (control: AbstractControl): ValidationErrors | null => {
    const value = control.value?.toLowerCase();
    return value === forbidden.toLowerCase() 
      ? { forbiddenName: { value: control.value } }
      : null;
  };
}

// Password strength validator
export function passwordStrengthValidator(): ValidatorFn {
  return (control: AbstractControl): ValidationErrors | null => {
    const value = control.value;
    if (!value) return null;
    
    const hasUpperCase = /[A-Z]/.test(value);
    const hasLowerCase = /[a-z]/.test(value);
    const hasNumber = /[0-9]/.test(value);
    const hasSpecial = /[!@#$%^&*]/.test(value);
    const minLength = value.length >= 8;
    
    const valid = hasUpperCase && hasLowerCase && hasNumber && hasSpecial && minLength;
    
    return valid ? null : {
      passwordStrength: {
        hasUpperCase,
        hasLowerCase,
        hasNumber,
        hasSpecial,
        minLength
      }
    };
  };
}

// Usage
const form = this.fb.group({
  username: ['', [Validators.required, forbiddenNameValidator('admin')]],
  password: ['', [Validators.required, passwordStrengthValidator()]]
});

// Display detailed password errors
<div *ngIf="password.errors?.['passwordStrength'] as errors">
  <div [class.valid]="errors.hasUpperCase">✓ Uppercase letter</div>
  <div [class.valid]="errors.hasLowerCase">✓ Lowercase letter</div>
  <div [class.valid]="errors.hasNumber">✓ Number</div>
  <div [class.valid]="errors.hasSpecial">✓ Special character</div>
  <div [class.valid]="errors.minLength">✓ Minimum 8 characters</div>
</div>
```

### Cross-Field Validators (Group Validators)

```typescript
// Password match validator
export function passwordMatchValidator(): ValidatorFn {
  return (group: AbstractControl): ValidationErrors | null => {
    const password = group.get('password')?.value;
    const confirmPassword = group.get('confirmPassword')?.value;
    
    return password === confirmPassword ? null : { passwordMismatch: true };
  };
}

// Date range validator
export function dateRangeValidator(): ValidatorFn {
  return (group: AbstractControl): ValidationErrors | null => {
    const startDate = group.get('startDate')?.value;
    const endDate = group.get('endDate')?.value;
    
    if (!startDate || !endDate) return null;
    
    return new Date(startDate) <= new Date(endDate) 
      ? null 
      : { dateRange: { message: 'End date must be after start date' } };
  };
}

// Usage
const form = this.fb.group({
  password: ['', Validators.required],
  confirmPassword: ['', Validators.required]
}, {
  validators: [passwordMatchValidator()]  // Group-level validator
});

// Template
<div *ngIf="form.errors?.['passwordMismatch']" class="error">
  Passwords do not match
</div>
```

### Validator Directive (Template-Driven)

```typescript
import { Directive } from '@angular/core';
import { NG_VALIDATORS, Validator, AbstractControl, ValidationErrors } from '@angular/forms';

@Directive({
  selector: '[appForbiddenName]',
  providers: [{
    provide: NG_VALIDATORS,
    useExisting: ForbiddenNameDirective,
    multi: true
  }],
  standalone: true
})
export class ForbiddenNameDirective implements Validator {
  @Input('appForbiddenName') forbiddenName = '';
  
  validate(control: AbstractControl): ValidationErrors | null {
    return control.value === this.forbiddenName 
      ? { forbiddenName: { value: control.value } }
      : null;
  }
}

// Usage in template
<input 
  name="username"
  [(ngModel)]="username"
  appForbiddenName="admin"
>
```

---

## Async Validators

### Creating Async Validators

```typescript
import { AsyncValidatorFn, AbstractControl, ValidationErrors } from '@angular/forms';
import { Observable, of, timer } from 'rxjs';
import { map, catchError, switchMap } from 'rxjs/operators';

// Async validator function
export function uniqueUsernameValidator(userService: UserService): AsyncValidatorFn {
  return (control: AbstractControl): Observable<ValidationErrors | null> => {
    if (!control.value) {
      return of(null);
    }
    
    // Debounce to avoid too many API calls
    return timer(300).pipe(
      switchMap(() => userService.checkUsername(control.value)),
      map(isTaken => isTaken ? { usernameTaken: true } : null),
      catchError(() => of(null))  // On error, don't block
    );
  };
}

// Email availability validator
export function emailAvailableValidator(authService: AuthService): AsyncValidatorFn {
  return (control: AbstractControl): Observable<ValidationErrors | null> => {
    return authService.checkEmail(control.value).pipe(
      map(response => response.available ? null : { emailTaken: true }),
      catchError(error => of({ serverError: true }))
    );
  };
}

// Usage
@Component({...})
export class RegisterComponent {
  form = this.fb.group({
    username: [
      '',
      [Validators.required, Validators.minLength(3)],  // Sync validators
      [uniqueUsernameValidator(this.userService)]       // Async validators
    ],
    email: [
      '',
      [Validators.required, Validators.email],
      [emailAvailableValidator(this.authService)]
    ]
  });
  
  constructor(
    private fb: FormBuilder,
    private userService: UserService,
    private authService: AuthService
  ) {}
}
```

### Handling Async Validation States

```html
<div class="form-field">
  <label>Username</label>
  <input formControlName="username">
  
  <!-- Pending state (validating...) -->
  <div *ngIf="username.pending" class="validating">
    <span class="spinner"></span> Checking availability...
  </div>
  
  <!-- Errors -->
  <div *ngIf="username.errors && username.touched" class="errors">
    <div *ngIf="username.errors['required']">Username required</div>
    <div *ngIf="username.errors['minlength']">Min 3 characters</div>
    <div *ngIf="username.errors['usernameTaken']">Username is taken</div>
    <div *ngIf="username.errors['serverError']">Unable to validate</div>
  </div>
  
  <!-- Success -->
  <div *ngIf="username.valid && username.dirty" class="success">
    ✓ Username available
  </div>
</div>

<button [disabled]="form.invalid || form.pending">Submit</button>
```

### Async Validator Class (Injectable)

```typescript
@Injectable({ providedIn: 'root' })
export class UniqueUsernameValidator implements AsyncValidator {
  constructor(private userService: UserService) {}
  
  validate(control: AbstractControl): Observable<ValidationErrors | null> {
    return timer(300).pipe(
      switchMap(() => this.userService.checkUsername(control.value)),
      map(isTaken => isTaken ? { usernameTaken: true } : null),
      catchError(() => of(null))
    );
  }
}

// Usage with DI
@Component({...})
export class RegisterComponent {
  constructor(
    private fb: FormBuilder,
    private uniqueUsernameValidator: UniqueUsernameValidator
  ) {}
  
  form = this.fb.group({
    username: [
      '',
      [Validators.required],
      [this.uniqueUsernameValidator.validate.bind(this.uniqueUsernameValidator)]
    ]
  });
}
```

---

## Dynamic Forms

### Dynamic Form Generation

```typescript
// form-config.ts
export interface FormFieldConfig {
  type: 'text' | 'email' | 'password' | 'number' | 'select' | 'checkbox' | 'textarea';
  name: string;
  label: string;
  placeholder?: string;
  value?: any;
  validators?: ValidatorFn[];
  asyncValidators?: AsyncValidatorFn[];
  options?: { value: any; label: string }[];  // For select
}

// dynamic-form.component.ts
@Component({
  selector: 'app-dynamic-form',
  template: `
    <form [formGroup]="form" (ngSubmit)="onSubmit()">
      <div *ngFor="let field of fields" class="form-field">
        <label [for]="field.name">{{ field.label }}</label>
        
        <ng-container [ngSwitch]="field.type">
          <!-- Text inputs -->
          <input 
            *ngSwitchCase="'text'"
            [id]="field.name"
            type="text"
            [formControlName]="field.name"
            [placeholder]="field.placeholder || ''"
          >
          
          <input 
            *ngSwitchCase="'email'"
            [id]="field.name"
            type="email"
            [formControlName]="field.name"
          >
          
          <input 
            *ngSwitchCase="'password'"
            [id]="field.name"
            type="password"
            [formControlName]="field.name"
          >
          
          <input 
            *ngSwitchCase="'number'"
            [id]="field.name"
            type="number"
            [formControlName]="field.name"
          >
          
          <!-- Select -->
          <select 
            *ngSwitchCase="'select'"
            [id]="field.name"
            [formControlName]="field.name"
          >
            <option value="">Select...</option>
            <option *ngFor="let opt of field.options" [value]="opt.value">
              {{ opt.label }}
            </option>
          </select>
          
          <!-- Checkbox -->
          <input 
            *ngSwitchCase="'checkbox'"
            [id]="field.name"
            type="checkbox"
            [formControlName]="field.name"
          >
          
          <!-- Textarea -->
          <textarea
            *ngSwitchCase="'textarea'"
            [id]="field.name"
            [formControlName]="field.name"
          ></textarea>
        </ng-container>
        
        <app-field-errors [control]="form.get(field.name)!"></app-field-errors>
      </div>
      
      <button type="submit" [disabled]="form.invalid">Submit</button>
    </form>
  `
})
export class DynamicFormComponent implements OnInit {
  @Input() fields: FormFieldConfig[] = [];
  @Output() formSubmit = new EventEmitter<any>();
  
  form!: FormGroup;
  
  constructor(private fb: FormBuilder) {}
  
  ngOnInit() {
    this.form = this.buildForm();
  }
  
  private buildForm(): FormGroup {
    const group: { [key: string]: FormControl } = {};
    
    this.fields.forEach(field => {
      group[field.name] = new FormControl(
        field.value || '',
        field.validators || [],
        field.asyncValidators || []
      );
    });
    
    return new FormGroup(group);
  }
  
  onSubmit() {
    if (this.form.valid) {
      this.formSubmit.emit(this.form.value);
    }
  }
}

// Usage
@Component({
  template: `
    <app-dynamic-form 
      [fields]="formConfig" 
      (formSubmit)="handleSubmit($event)"
    ></app-dynamic-form>
  `
})
export class RegistrationComponent {
  formConfig: FormFieldConfig[] = [
    {
      type: 'text',
      name: 'firstName',
      label: 'First Name',
      validators: [Validators.required]
    },
    {
      type: 'email',
      name: 'email',
      label: 'Email',
      validators: [Validators.required, Validators.email]
    },
    {
      type: 'select',
      name: 'country',
      label: 'Country',
      options: [
        { value: 'us', label: 'United States' },
        { value: 'uk', label: 'United Kingdom' },
        { value: 'ca', label: 'Canada' }
      ]
    }
  ];
  
  handleSubmit(data: any) {
    console.log('Form submitted:', data);
  }
}
```

---

## Form Arrays

### Basic FormArray Usage

```typescript
@Component({
  template: `
    <form [formGroup]="userForm" (ngSubmit)="onSubmit()">
      <input formControlName="name" placeholder="Name">
      
      <div formArrayName="phones">
        <h4>Phone Numbers</h4>
        
        <div *ngFor="let phone of phones.controls; let i = index" class="phone-row">
          <input [formControlName]="i" placeholder="Phone {{ i + 1 }}">
          <button type="button" (click)="removePhone(i)">Remove</button>
        </div>
        
        <button type="button" (click)="addPhone()">Add Phone</button>
      </div>
      
      <button type="submit">Submit</button>
    </form>
  `
})
export class UserFormComponent implements OnInit {
  userForm!: FormGroup;
  
  constructor(private fb: FormBuilder) {}
  
  ngOnInit() {
    this.userForm = this.fb.group({
      name: ['', Validators.required],
      phones: this.fb.array([
        this.fb.control('', Validators.required)  // Initial phone
      ])
    });
  }
  
  get phones(): FormArray {
    return this.userForm.get('phones') as FormArray;
  }
  
  addPhone() {
    this.phones.push(this.fb.control('', Validators.required));
  }
  
  removePhone(index: number) {
    if (this.phones.length > 1) {
      this.phones.removeAt(index);
    }
  }
  
  onSubmit() {
    console.log(this.userForm.value);
    // { name: 'John', phones: ['555-1234', '555-5678'] }
  }
}
```

### FormArray with FormGroups

```typescript
@Component({
  template: `
    <form [formGroup]="orderForm" (ngSubmit)="onSubmit()">
      <input formControlName="customerName" placeholder="Customer Name">
      
      <div formArrayName="items">
        <h4>Order Items</h4>
        
        <div *ngFor="let item of items.controls; let i = index" [formGroupName]="i" class="item-row">
          <input formControlName="productName" placeholder="Product">
          <input formControlName="quantity" type="number" placeholder="Qty">
          <input formControlName="price" type="number" placeholder="Price">
          
          <span class="subtotal">
            Subtotal: {{ calculateSubtotal(i) | currency }}
          </span>
          
          <button type="button" (click)="removeItem(i)">Remove</button>
        </div>
        
        <button type="button" (click)="addItem()">Add Item</button>
      </div>
      
      <div class="total">
        <strong>Total: {{ calculateTotal() | currency }}</strong>
      </div>
      
      <button type="submit" [disabled]="orderForm.invalid">Place Order</button>
    </form>
  `
})
export class OrderFormComponent implements OnInit {
  orderForm!: FormGroup;
  
  constructor(private fb: FormBuilder) {}
  
  ngOnInit() {
    this.orderForm = this.fb.group({
      customerName: ['', Validators.required],
      items: this.fb.array([this.createItem()])
    });
  }
  
  get items(): FormArray {
    return this.orderForm.get('items') as FormArray;
  }
  
  createItem(): FormGroup {
    return this.fb.group({
      productName: ['', Validators.required],
      quantity: [1, [Validators.required, Validators.min(1)]],
      price: [0, [Validators.required, Validators.min(0)]]
    });
  }
  
  addItem() {
    this.items.push(this.createItem());
  }
  
  removeItem(index: number) {
    if (this.items.length > 1) {
      this.items.removeAt(index);
    }
  }
  
  calculateSubtotal(index: number): number {
    const item = this.items.at(index);
    const qty = item.get('quantity')?.value || 0;
    const price = item.get('price')?.value || 0;
    return qty * price;
  }
  
  calculateTotal(): number {
    return this.items.controls.reduce((total, _, index) => {
      return total + this.calculateSubtotal(index);
    }, 0);
  }
  
  onSubmit() {
    console.log(this.orderForm.value);
  }
}
```

---

## Typed Forms (Angular 14+)

### Strictly Typed Forms

```typescript
import { FormControl, FormGroup, NonNullableFormBuilder } from '@angular/forms';

// Define interface
interface UserForm {
  name: FormControl<string>;
  email: FormControl<string>;
  age: FormControl<number | null>;
  subscribe: FormControl<boolean>;
}

@Component({...})
export class TypedFormComponent {
  // Method 1: Explicit typing
  userForm = new FormGroup<UserForm>({
    name: new FormControl('', { nonNullable: true }),
    email: new FormControl('', { nonNullable: true }),
    age: new FormControl<number | null>(null),
    subscribe: new FormControl(false, { nonNullable: true })
  });
  
  // Method 2: NonNullableFormBuilder
  constructor(private nnfb: NonNullableFormBuilder) {}
  
  userForm2 = this.nnfb.group({
    name: ['', Validators.required],      // Type: FormControl<string>
    email: ['', Validators.email],        // Type: FormControl<string>
    age: [null as number | null],         // Type: FormControl<number | null>
    subscribe: [false]                     // Type: FormControl<boolean>
  });
  
  // Value types are inferred
  submit() {
    const value = this.userForm.value;
    // Type: { name?: string; email?: string; age?: number | null; subscribe?: boolean }
    
    const rawValue = this.userForm.getRawValue();
    // Type: { name: string; email: string; age: number | null; subscribe: boolean }
  }
  
  // Type-safe access
  updateName() {
    const nameControl = this.userForm.controls.name;
    // Type: FormControl<string>
    
    nameControl.setValue('John');    // ✅ OK
    // nameControl.setValue(123);    // ❌ Error: number not assignable to string
  }
}
```

### Typed FormArrays

```typescript
interface OrderItem {
  productName: FormControl<string>;
  quantity: FormControl<number>;
  price: FormControl<number>;
}

interface OrderForm {
  customerName: FormControl<string>;
  items: FormArray<FormGroup<OrderItem>>;
}

@Component({...})
export class TypedOrderComponent {
  orderForm = new FormGroup<OrderForm>({
    customerName: new FormControl('', { nonNullable: true }),
    items: new FormArray<FormGroup<OrderItem>>([])
  });
  
  addItem() {
    const itemGroup = new FormGroup<OrderItem>({
      productName: new FormControl('', { nonNullable: true }),
      quantity: new FormControl(1, { nonNullable: true }),
      price: new FormControl(0, { nonNullable: true })
    });
    
    this.orderForm.controls.items.push(itemGroup);
  }
  
  // Type inference works
  calculateTotal(): number {
    return this.orderForm.controls.items.controls.reduce((total, item) => {
      const qty = item.controls.quantity.value;  // Type: number
      const price = item.controls.price.value;    // Type: number
      return total + qty * price;
    }, 0);
  }
}
```

---

## Best Practices

### 1. Form Organization

```typescript
// Separate form creation into methods
@Component({...})
export class ComplexFormComponent implements OnInit {
  form!: FormGroup;
  
  ngOnInit() {
    this.form = this.createForm();
  }
  
  private createForm(): FormGroup {
    return this.fb.group({
      personal: this.createPersonalGroup(),
      address: this.createAddressGroup(),
      preferences: this.createPreferencesGroup()
    });
  }
  
  private createPersonalGroup(): FormGroup {
    return this.fb.group({
      firstName: ['', Validators.required],
      lastName: ['', Validators.required],
      email: ['', [Validators.required, Validators.email]]
    });
  }
  
  private createAddressGroup(): FormGroup {
    return this.fb.group({
      street: [''],
      city: [''],
      zipCode: ['', Validators.pattern(/^\d{5}$/)]
    });
  }
  
  private createPreferencesGroup(): FormGroup {
    return this.fb.group({
      newsletter: [false],
      notifications: [true]
    });
  }
}
```

### 2. Reusable Form Components

```typescript
// address-form.component.ts
@Component({
  selector: 'app-address-form',
  template: `
    <div [formGroup]="addressGroup">
      <input formControlName="street" placeholder="Street">
      <input formControlName="city" placeholder="City">
      <input formControlName="zipCode" placeholder="Zip Code">
    </div>
  `
})
export class AddressFormComponent {
  @Input() addressGroup!: FormGroup;
}

// Parent component
@Component({
  template: `
    <form [formGroup]="form">
      <input formControlName="name">
      
      <h4>Billing Address</h4>
      <app-address-form [addressGroup]="form.get('billingAddress')!"></app-address-form>
      
      <h4>Shipping Address</h4>
      <app-address-form [addressGroup]="form.get('shippingAddress')!"></app-address-form>
    </form>
  `
})
export class CheckoutComponent {
  form = this.fb.group({
    name: [''],
    billingAddress: this.createAddressGroup(),
    shippingAddress: this.createAddressGroup()
  });
}
```

### 3. ControlValueAccessor (Custom Form Controls)

```typescript
@Component({
  selector: 'app-star-rating',
  template: `
    <span 
      *ngFor="let star of stars; let i = index"
      (click)="setRating(i + 1)"
      [class.filled]="i < value"
    >★</span>
  `,
  providers: [{
    provide: NG_VALUE_ACCESSOR,
    useExisting: forwardRef(() => StarRatingComponent),
    multi: true
  }]
})
export class StarRatingComponent implements ControlValueAccessor {
  stars = [1, 2, 3, 4, 5];
  value = 0;
  disabled = false;
  
  private onChange = (value: number) => {};
  private onTouched = () => {};
  
  writeValue(value: number): void {
    this.value = value || 0;
  }
  
  registerOnChange(fn: (value: number) => void): void {
    this.onChange = fn;
  }
  
  registerOnTouched(fn: () => void): void {
    this.onTouched = fn;
  }
  
  setDisabledState(disabled: boolean): void {
    this.disabled = disabled;
  }
  
  setRating(rating: number): void {
    if (!this.disabled) {
      this.value = rating;
      this.onChange(rating);
      this.onTouched();
    }
  }
}

// Usage
<app-star-rating formControlName="rating"></app-star-rating>
```

---

## Interview Questions & Answers

### Q1: What's the difference between setValue and patchValue?

**Answer:**

| Method | Behavior |
|--------|----------|
| **setValue** | Requires ALL form controls to be provided. Throws error if any missing. |
| **patchValue** | Allows partial updates. Only updates provided controls. |

```typescript
const form = this.fb.group({
  name: [''],
  email: [''],
  age: [null]
});

// setValue - must include all
form.setValue({
  name: 'John',
  email: 'john@example.com',
  age: 30
});
// form.setValue({ name: 'John' }); // ERROR!

// patchValue - can be partial
form.patchValue({ name: 'Jane' });  // Works fine
```

---

### Q2: How do async validators work?

**Answer:**

Async validators return an `Observable` or `Promise` that resolves to `ValidationErrors | null`.

```typescript
export function uniqueEmailValidator(service: UserService): AsyncValidatorFn {
  return (control: AbstractControl): Observable<ValidationErrors | null> => {
    return timer(300).pipe(  // Debounce
      switchMap(() => service.checkEmail(control.value)),
      map(exists => exists ? { emailTaken: true } : null),
      catchError(() => of(null))
    );
  };
}

// Usage
email: ['', 
  [Validators.required],        // Sync validators (2nd arg)
  [uniqueEmailValidator(svc)]   // Async validators (3rd arg)
]
```

**Key points:**
- Run AFTER sync validators pass
- Form status becomes `PENDING` while running
- Should include debounce to avoid excessive API calls
- Should handle errors gracefully

---

### Q3: What is ControlValueAccessor and when would you use it?

**Answer:**

`ControlValueAccessor` is an interface that bridges custom form controls with Angular forms (both template-driven and reactive).

**When to use:**
- Creating custom input components
- Wrapping third-party UI components
- Building complex inputs (date pickers, color pickers, ratings)

**Required methods:**
```typescript
interface ControlValueAccessor {
  writeValue(value: any): void;           // Form → Component
  registerOnChange(fn: any): void;        // Component → Form
  registerOnTouched(fn: any): void;       // Touch handling
  setDisabledState?(disabled: boolean): void;  // Disable state
}
```

---

### Q4: How do you handle form submission with validation?

**Answer:**

```typescript
onSubmit() {
  // 1. Check validity
  if (this.form.invalid) {
    // Mark all as touched to show errors
    this.form.markAllAsTouched();
    return;
  }
  
  // 2. Check if async validation pending
  if (this.form.pending) {
    // Wait for async validators
    this.form.statusChanges.pipe(
      filter(status => status !== 'PENDING'),
      take(1)
    ).subscribe(() => this.submitForm());
    return;
  }
  
  this.submitForm();
}

private submitForm() {
  const data = this.form.getRawValue();  // Includes disabled fields
  this.service.save(data).subscribe({
    next: () => {
      this.form.reset();
      this.router.navigate(['/success']);
    },
    error: (err) => {
      // Handle server-side validation errors
      if (err.errors) {
        Object.keys(err.errors).forEach(key => {
          this.form.get(key)?.setErrors({ serverError: err.errors[key] });
        });
      }
    }
  });
}
```

---

### Q5: Template-Driven vs Reactive Forms - When to use which?

**Answer:**

| Aspect | Template-Driven | Reactive |
|--------|-----------------|----------|
| **Form model** | Template | Component class |
| **Complexity** | Simple forms | Complex forms |
| **Testing** | Harder (async) | Easy (sync) |
| **Dynamic fields** | Difficult | Easy |
| **Validation** | Directives | Functions |
| **Scalability** | Poor | Good |

**Template-Driven:**
```html
<input [(ngModel)]="user.name" required #name="ngModel">
```

**Reactive:**
```typescript
name = new FormControl('', Validators.required);
```

**Recommendation:**
- Prototypes, simple forms → Template-Driven
- Enterprise apps, complex validation, testing → Reactive

---

## Summary Checklist

✅ **Template-Driven Forms**
- FormsModule, ngModel, ngForm
- Validation directives
- Template reference variables

✅ **Reactive Forms**
- FormControl, FormGroup, FormArray
- FormBuilder patterns
- setValue vs patchValue

✅ **Validation**
- Built-in validators
- Custom sync/async validators
- Cross-field validation

✅ **Advanced**
- Typed Forms (Angular 14+)
- ControlValueAccessor
- Dynamic forms

✅ **Best Practices**
- Separate form creation logic
- Reusable form components
- Proper error handling

---

**Key Takeaway:** Reactive forms are the preferred choice for enterprise applications due to their testability, type safety, and flexibility. Master FormArray for dynamic fields and ControlValueAccessor for custom inputs.
