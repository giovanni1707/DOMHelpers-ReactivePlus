# `form.errors` - Form Errors State Property

**Quick Start (30 seconds)**
```javascript
const form = Forms.createForm({ email: '', password: '' }, {
  validators: {
    email: Validators.email(),
    password: Validators.minLength(8)
  }
});

console.log(form.errors); // {} (no errors initially)

form.setValue('email', 'invalid');
form.validateField('email');
console.log(form.errors); // { email: 'Invalid email format' }

form.setValue('email', 'user@example.com');
form.validateField('email');
console.log(form.errors); // { email: null } (validated, but passed)

form.setError('password', 'Custom error message');
console.log(form.errors); // { email: null, password: 'Custom error message' }
```

---

## **What is `form.errors`?**

`form.errors` is a **reactive state property** that contains all validation error messages for your form fields. It's a plain object where each key is a field name and each value is either an error message string (when invalid) or `null` (when valid).

**Key characteristics:**
- **Reactive**: Changes trigger effects and re-renders
- **Error Messages**: Values are strings (errors) or `null` (no error)
- **Plain Object**: Standard JavaScript object with field errors
- **Mutable**: Can be set via validation or manually
- **Initially Empty**: Starts as `{}` until validation runs

---

## **Syntax**

```javascript
// Read errors
const allErrors = form.errors;
const emailError = form.errors.email;

// Write errors (manual)
form.errors.email = 'Invalid email';
form.errors.password = null; // Clear error

// Preferred: Use methods
form.setError('email', 'Invalid email');
form.clearError('email');
form.validateField('email'); // Auto-sets error based on validator
```

### **Structure**
```javascript
form.errors = {
  fieldName1: 'Error message' | null,
  fieldName2: 'Error message' | null,
  ...
}
```

---

## **Why does `form.errors` exist?**

### ❌ **The Problem**
```javascript
// Without form.errors, you'd manually track errors
let emailError = null;
let passwordError = null;

function validateEmail(value) {
  if (!value.includes('@')) {
    emailError = 'Invalid email';
  } else {
    emailError = null;
  }
}

// Tedious to manage
if (emailError || passwordError) {
  console.log('Form has errors');
}

// No centralized error state
```

### ✅ **The Solution**
```javascript
// Centralized reactive error state
const form = Forms.createForm({ email: '', password: '' }, {
  validators: {
    email: Validators.email(),
    password: Validators.minLength(8)
  }
});

// All errors in one place
console.log(form.errors);

// Easy to check validity
if (form.hasErrors) {
  console.log('Form has errors');
}

// Reactive error display
effect(() => {
  if (form.errors.email) {
    showError('email', form.errors.email);
  }
});
```

---

## **Mental Model**

Think of `form.errors` as a **problem report** that highlights what's wrong with your form:

```
┌─────────────────────────────────────────────┐
│           FORM ERRORS OBJECT                │
├─────────────────────────────────────────────┤
│                                             │
│  form.errors = {                            │
│    email: null,                 ← Valid ✓   │
│    password: 'Too short',       ← Error ✗   │
│    confirmPassword: null,       ← Valid ✓   │
│    age: 'Must be 18+'          ← Error ✗   │
│  }                                          │
│                                             │
│  ↓ Reactive - triggers effects              │
│  ↓ Null = valid, String = error             │
│  ↓ Updated by validators or manually        │
│                                             │
└─────────────────────────────────────────────┘
```

**The validation flow:**
```
User Input → Validator Runs → Error Set
    ↓             ↓              ↓
 "test"      email()      "Invalid email"
                              ↓
                        form.errors.email
```

---

## **How does it work?**

When you create a form, the `errors` property is initialized and made reactive:

```javascript
const formObj = {
  values: { ...initialValues },
  errors: {},  // Empty initially
  touched: {},
  isSubmitting: false
};

const form = createState(formObj); // Made reactive
```

When validation runs, errors are set:
```javascript
// Validator returns null (valid) or error message (invalid)
const error = validator(form.values.email);
form.errors.email = error; // Sets 'Invalid email' or null
```

---

## **Examples**

### **Example 1: Display Field Errors**
```javascript
const form = Forms.createForm({ email: '', password: '' }, {
  validators: {
    email: Validators.email('Please enter a valid email'),
    password: Validators.minLength(8, 'Password must be 8+ characters')
  }
});

effect(() => {
  const emailErrorEl = document.querySelector('#email-error');
  const passwordErrorEl = document.querySelector('#password-error');

  emailErrorEl.textContent = form.errors.email || '';
  passwordErrorEl.textContent = form.errors.password || '';

  // Show/hide error messages
  emailErrorEl.style.display = form.errors.email ? 'block' : 'none';
  passwordErrorEl.style.display = form.errors.password ? 'block' : 'none';
});
```

### **Example 2: Error Summary**
```javascript
const form = Forms.createForm({
  name: '',
  email: '',
  phone: ''
}, {
  validators: {
    name: Validators.required(),
    email: Validators.email(),
    phone: Validators.pattern(/^\d{10}$/)
  }
});

effect(() => {
  const summaryEl = document.querySelector('#error-summary');

  // Get all non-null errors
  const errors = Object.entries(form.errors)
    .filter(([field, error]) => error)
    .map(([field, error]) => ({ field, error }));

  if (errors.length === 0) {
    summaryEl.innerHTML = '<p class="success">No errors</p>';
  } else {
    const errorList = errors
      .map(({ field, error }) => `<li><strong>${field}:</strong> ${error}</li>`)
      .join('');

    summaryEl.innerHTML = `
      <p class="error">${errors.length} error(s) found:</p>
      <ul>${errorList}</ul>
    `;
  }
});
```

### **Example 3: Manual Error Setting**
```javascript
const form = Forms.createForm({ username: '' });

async function checkUsername(username) {
  const response = await fetch(`/api/check-username/${username}`);
  const data = await response.json();

  if (!data.available) {
    form.setError('username', 'Username already taken');
  } else {
    form.clearError('username');
  }
}

// Trigger on blur
form.bindToInputs('#my-form', {
  onBlur: (field, value) => {
    if (field === 'username') {
      checkUsername(value);
    }
  }
});
```

### **Example 4: Server-side Errors**
```javascript
const form = Forms.createForm({ email: '', password: '' });

async function handleSubmit() {
  try {
    await fetch('/api/login', {
      method: 'POST',
      body: JSON.stringify(form.values)
    });
  } catch (error) {
    // Server returned validation errors
    if (error.response.status === 422) {
      const serverErrors = await error.response.json();

      // Set errors from server
      form.setErrors(serverErrors);
      // { email: 'Email not found', password: 'Incorrect password' }
    }
  }
}
```

### **Example 5: Conditional Error Messages**
```javascript
const form = Forms.createForm({ password: '', confirmPassword: '' }, {
  validators: {
    password: Validators.minLength(8),
    confirmPassword: Validators.match('password', 'Passwords must match')
  }
});

effect(() => {
  const errorEl = document.querySelector('#password-error');

  if (form.errors.password) {
    errorEl.textContent = form.errors.password;
    errorEl.className = 'error';
  } else if (form.errors.confirmPassword) {
    errorEl.textContent = form.errors.confirmPassword;
    errorEl.className = 'error';
  } else if (form.values.password && form.values.confirmPassword) {
    errorEl.textContent = '✓ Passwords match';
    errorEl.className = 'success';
  } else {
    errorEl.textContent = '';
  }
});
```

### **Example 6: Field Highlighting**
```javascript
const form = Forms.createForm({
  name: '',
  email: '',
  phone: ''
}, {
  validators: {
    name: Validators.required(),
    email: Validators.email(),
    phone: Validators.pattern(/^\d{10}$/)
  }
});

effect(() => {
  // Clear all highlights
  document.querySelectorAll('input').forEach(input => {
    input.classList.remove('error', 'valid');
  });

  // Highlight based on error state
  Object.keys(form.errors).forEach(field => {
    const input = document.querySelector(`input[name="${field}"]`);

    if (input && form.touched[field]) {
      if (form.errors[field]) {
        input.classList.add('error');
      } else {
        input.classList.add('valid');
      }
    }
  });
});
```

### **Example 7: Error Count Badge**
```javascript
const form = Forms.createForm({
  field1: '',
  field2: '',
  field3: ''
}, {
  validators: {
    field1: Validators.required(),
    field2: Validators.required(),
    field3: Validators.required()
  }
});

effect(() => {
  const badge = document.querySelector('#error-badge');

  const errorCount = Object.values(form.errors)
    .filter(error => error) // Count non-null errors
    .length;

  if (errorCount > 0) {
    badge.textContent = errorCount;
    badge.style.display = 'inline-block';
  } else {
    badge.style.display = 'none';
  }
});
```

### **Example 8: Clear All Errors**
```javascript
const form = Forms.createForm({ email: '', password: '' });

function handleReset() {
  form.reset(); // Clears values, errors, and touched
}

function clearErrorsOnly() {
  form.clearErrors(); // Clears all errors but keeps values
}

// Or manually
function manualClear() {
  Object.keys(form.errors).forEach(field => {
    form.errors[field] = null;
  });
}
```

### **Example 9: Async Validation Errors**
```javascript
const form = Forms.createForm({ username: '' }, {
  validators: {
    username: Validators.custom(async (value) => {
      if (value.length < 3) {
        return 'Username must be 3+ characters';
      }

      const response = await fetch(`/api/check/${value}`);
      const data = await response.json();

      return data.available ? null : 'Username taken';
    })
  }
});

effect(() => {
  const errorEl = document.querySelector('#username-error');

  if (form.errors.username) {
    errorEl.textContent = form.errors.username;
    errorEl.style.display = 'block';
  } else {
    errorEl.style.display = 'none';
  }
});
```

### **Example 10: Error Tooltips**
```javascript
const form = Forms.createForm({
  email: '',
  password: ''
}, {
  validators: {
    email: Validators.email(),
    password: Validators.minLength(8)
  }
});

effect(() => {
  Object.entries(form.errors).forEach(([field, error]) => {
    const input = document.querySelector(`input[name="${field}"]`);

    if (input) {
      if (error && form.touched[field]) {
        input.setAttribute('title', error);
        input.setAttribute('aria-invalid', 'true');
        input.setAttribute('aria-describedby', `${field}-error`);
      } else {
        input.removeAttribute('title');
        input.setAttribute('aria-invalid', 'false');
      }
    }
  });
});
```

---

## **Common Patterns**

### **Pattern 1: Check if Field Has Error**
```javascript
if (form.errors.email) {
  showError(form.errors.email);
}
```

### **Pattern 2: Get All Error Messages**
```javascript
const errorMessages = Object.values(form.errors).filter(e => e);
```

### **Pattern 3: Count Errors**
```javascript
const errorCount = Object.values(form.errors).filter(e => e).length;
```

### **Pattern 4: Iterate Errors**
```javascript
Object.entries(form.errors).forEach(([field, error]) => {
  if (error) {
    console.log(`${field}: ${error}`);
  }
});
```

### **Pattern 5: Clear Specific Error**
```javascript
form.errors.email = null;
// Or
form.clearError('email');
```

---

## **Comparison with Related Properties**

| Property | Type | What it contains |
|----------|------|-----------------|
| `form.errors` | Object | All field errors (null or string) |
| `form.errorFields` | Array | Only field names with truthy errors |
| `form.hasErrors` | Boolean | `true` if any truthy errors exist |
| `form.isValid` | Boolean | `true` if no truthy errors exist |

```javascript
// Example relationships
form.errors = {
  email: 'Invalid email',
  password: null,
  phone: ''
};

form.errors;           // { email: 'Invalid...', password: null, phone: '' }
form.errorFields;      // ['email'] (only truthy errors)
form.hasErrors;        // true
form.isValid;          // false
```

---

## **Error Value Meanings**

| Value | Meaning | Example |
|-------|---------|---------|
| `undefined` | Field not yet validated | `form.errors.email === undefined` |
| `null` | Field validated, passed | `form.errors.email === null` |
| `''` (empty string) | Field validated, passed (rare) | `form.errors.email === ''` |
| String (non-empty) | Field validated, failed | `form.errors.email === 'Invalid'` |

```javascript
const form = Forms.createForm({ email: '' });

console.log(form.errors.email); // undefined (not validated)

form.validateField('email');
console.log(form.errors.email); // null or 'error message'
```

---

## **Key Takeaways**

1. **Reactive Object**: Changes to `form.errors` trigger effects
2. **Error Messages**: Values are strings (errors) or `null` (valid)
3. **Direct Access**: Can read errors directly: `form.errors.fieldName`
4. **Prefer Methods**: Use `setError()`, `clearError()` for proper behavior
5. **Set by Validators**: Automatically updated when `validateField()` runs
6. **Manual Control**: Can set custom errors from server or async validation
7. **Initially Empty**: Starts as `{}`, populated as validation runs
8. **Common Use**: Error displays, validation feedback, form state management

---

## **Summary**

`form.errors` is a reactive state property containing all validation error messages for your form fields as a plain JavaScript object. Each key is a field name with a value of either `null` (valid) or an error message string (invalid). Changes to this object automatically trigger reactive effects, making it perfect for displaying validation errors, highlighting problematic fields, and managing form submission logic. While you can access and modify errors directly, using form methods like `setError()`, `clearError()`, and `validateField()` is recommended for proper error management.
