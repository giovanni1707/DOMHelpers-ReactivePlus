# Form State Properties - Complete Reference

**Quick Start (30 seconds)**
```javascript
const form = ReactiveUtils.form({
  email: '',
  password: '',
  rememberMe: false
}, {
  validators: {
    email: Forms.v.email(),
    password: Forms.v.minLength(8)
  }
});

// Access form values
console.log(form.values); // { email: '', password: '', rememberMe: false }

// Check validation errors
console.log(form.errors); // {}

// Check touched fields
console.log(form.touched); // {}

// Check submission state
console.log(form.isSubmitting); // false

// Update values
form.setValue('email', 'user@example.com');
console.log(form.values.email); // 'user@example.com'
console.log(form.touched.email); // true
```

---

## **What are Form State Properties?**

Form state properties are **reactive object properties** that store the core state of a form, automatically tracking changes and triggering effects.

**The 4 core state properties:**
1. **`values`** - Object containing all form field values
2. **`errors`** - Object containing validation error messages
3. **`touched`** - Object tracking which fields have been modified
4. **`isSubmitting`** - Boolean indicating form submission state

**Key characteristics:**
- **Reactive**: Changes trigger effects automatically
- **Mutable**: Can be modified directly or via methods
- **Tracked**: All changes are dependency-tracked
- **Persistent**: State persists across interactions
- **Type Safe**: Predictable structure and types

---

## **Property Reference**

### **1. `form.values`**

**Type**: `Object`
**Structure**: `{ [fieldName: string]: any }`
**Default**: Copy of initial values
**Reactive**: Yes

Object containing all form field values. Each key is a field name, each value is the field's current value.

```javascript
const form = ReactiveUtils.form({
  name: 'John',
  email: 'john@example.com',
  age: 25
});

console.log(form.values);
// {
//   name: 'John',
//   email: 'john@example.com',
//   age: 25
// }

// Modify values
form.values.name = 'Jane';
form.setValue('email', 'jane@example.com');

console.log(form.values.name); // 'Jane'
```

**Modification methods:**
- Direct assignment: `form.values.field = value`
- `setValue(field, value)` - Preferred, also marks touched
- `setValues(obj)` - Batch update multiple fields

**Use cases:**
- Read current field values
- Submit form data
- Sync with external state
- Reset to defaults
- Serialize form data

---

### **2. `form.errors`**

**Type**: `Object`
**Structure**: `{ [fieldName: string]: string | null }`
**Default**: `{}`
**Reactive**: Yes

Object containing validation error messages. Each key is a field name, each value is an error message string (or null/undefined if valid).

```javascript
const form = ReactiveUtils.form({
  email: '',
  password: ''
}, {
  validators: {
    email: Forms.v.email(),
    password: Forms.v.minLength(8)
  }
});

// Initially no errors
console.log(form.errors); // {}

// Validate invalid email
form.setValue('email', 'invalid');
form.validateField('email');
console.log(form.errors);
// { email: 'Must be a valid email' }

// Fix the error
form.setValue('email', 'test@example.com');
form.validateField('email');
console.log(form.errors);
// { email: null } or {}
```

**Modification methods:**
- `setError(field, message)` - Set error message
- `setErrors(obj)` - Batch set multiple errors
- `clearError(field)` - Remove error
- `clearErrors()` - Remove all errors
- `validateField(field)` - Validate and set error
- `validateAll()` - Validate all fields

**Error values:**
- `string` - Error message
- `null` - No error (explicitly cleared)
- `undefined` - No error (field not validated)

**Use cases:**
- Display field-level errors
- Show error summary
- Enable/disable submit
- Track validation state
- Custom error messages

---

### **3. `form.touched`**

**Type**: `Object`
**Structure**: `{ [fieldName: string]: boolean }`
**Default**: `{}`
**Reactive**: Yes

Object tracking which fields have been touched (modified by user). Each key is a field name, each value is `true` (or undefined if not touched).

```javascript
const form = ReactiveUtils.form({
  name: '',
  email: '',
  phone: ''
});

// Initially empty
console.log(form.touched); // {}

// Touch a field
form.setValue('name', 'John');
console.log(form.touched); // { name: true }

// Touch more fields
form.setValue('email', 'john@example.com');
console.log(form.touched); // { name: true, email: true }

// Check if field is touched
console.log(form.touched.name); // true
console.log(form.touched.phone); // undefined
```

**Modification methods:**
- `setValue(field, value)` - Marks field as touched
- `setTouched(field, boolean)` - Manually set touched state
- `setTouchedFields(obj)` - Batch set touched state
- `reset()` - Clears all touched state

**Touched states:**
- `true` - Field has been modified
- `undefined` - Field not touched

**Use cases:**
- Show errors only for touched fields
- Track user interaction
- Warn on unsaved changes
- Enable validation on blur
- Conditional error display

---

### **4. `form.isSubmitting`**

**Type**: `Boolean`
**Default**: `false`
**Reactive**: Yes

Boolean indicating whether the form is currently being submitted.

```javascript
const form = ReactiveUtils.form({
  email: '',
  message: ''
}, {
  onSubmit: async (values) => {
    console.log('Submitting:', values);
    await new Promise(resolve => setTimeout(resolve, 2000));
    console.log('Submitted!');
  }
});

console.log(form.isSubmitting); // false

// Start submission
form.handleSubmit();
console.log(form.isSubmitting); // true

// After submission completes
setTimeout(() => {
  console.log(form.isSubmitting); // false
}, 2500);
```

**State changes:**
- Set to `true` when `handleSubmit()` is called
- Set to `false` when submission completes (success or error)
- Set to `false` when `reset()` is called

**Use cases:**
- Disable submit button during submission
- Show loading spinner
- Prevent double submission
- Display submission status
- Lock form during async operations

---

## **Examples**

### **Example 1: Read and Display Values**
```javascript
const form = ReactiveUtils.form({
  firstName: 'John',
  lastName: 'Doe',
  email: 'john@example.com'
});

ReactiveUtils.effect(() => {
  const fullName = `${form.values.firstName} ${form.values.lastName}`;
  const display = document.getElementById('preview');

  display.innerHTML = `
    <p>Name: ${fullName}</p>
    <p>Email: ${form.values.email}</p>
  `;
});

// Changes automatically update display
form.setValue('firstName', 'Jane');
```

### **Example 2: Error Display**
```javascript
const form = ReactiveUtils.form({
  email: '',
  password: ''
}, {
  validators: {
    email: Forms.v.email(),
    password: Forms.v.minLength(8)
  }
});

ReactiveUtils.effect(() => {
  ['email', 'password'].forEach(field => {
    const errorDiv = document.getElementById(`${field}-error`);
    const error = form.errors[field];

    if (error && form.touched[field]) {
      errorDiv.textContent = error;
      errorDiv.style.display = 'block';
    } else {
      errorDiv.style.display = 'none';
    }
  });
});
```

### **Example 3: Submit Button State**
```javascript
const form = ReactiveUtils.form({
  email: '',
  message: ''
}, {
  validators: {
    email: Forms.v.email(),
    message: Forms.v.minLength(10)
  },
  onSubmit: async (values) => {
    await fetch('/api/contact', {
      method: 'POST',
      body: JSON.stringify(values)
    });
  }
});

ReactiveUtils.effect(() => {
  const submitBtn = document.getElementById('submit');

  submitBtn.disabled = form.isSubmitting || !form.isValid;

  if (form.isSubmitting) {
    submitBtn.textContent = 'Submitting...';
  } else if (!form.isValid) {
    submitBtn.textContent = 'Fix Errors';
  } else {
    submitBtn.textContent = 'Submit';
  }
});
```

### **Example 4: Touched Field Indicators**
```javascript
const form = ReactiveUtils.form({
  username: '',
  email: '',
  password: ''
});

ReactiveUtils.effect(() => {
  ['username', 'email', 'password'].forEach(field => {
    const input = document.getElementById(field);
    const indicator = document.getElementById(`${field}-indicator`);

    if (form.touched[field]) {
      indicator.textContent = '✓';
      indicator.className = 'touched';
    } else {
      indicator.textContent = '○';
      indicator.className = 'untouched';
    }
  });
});
```

### **Example 5: Form Data Serialization**
```javascript
const form = ReactiveUtils.form({
  name: 'John Doe',
  email: 'john@example.com',
  preferences: {
    newsletter: true,
    notifications: false
  }
});

function serializeForm() {
  return JSON.stringify(form.values, null, 2);
}

function saveToLocalStorage() {
  localStorage.setItem('formData', serializeForm());
}

function loadFromLocalStorage() {
  const data = localStorage.getItem('formData');
  if (data) {
    const values = JSON.parse(data);
    form.setValues(values);
  }
}

// Auto-save on change
ReactiveUtils.effect(() => {
  if (form.isDirty) {
    saveToLocalStorage();
  }
});
```

### **Example 6: Error Summary**
```javascript
const form = ReactiveUtils.form({
  field1: '', field2: '', field3: ''
}, {
  validators: {
    field1: Forms.v.required(),
    field2: Forms.v.email(),
    field3: Forms.v.minLength(5)
  }
});

ReactiveUtils.effect(() => {
  const errorList = document.getElementById('error-list');
  const errorEntries = Object.entries(form.errors)
    .filter(([field, error]) => error);

  if (errorEntries.length === 0) {
    errorList.innerHTML = '<p class="success">No errors</p>';
    return;
  }

  const html = errorEntries
    .map(([field, error]) => `<li><strong>${field}:</strong> ${error}</li>`)
    .join('');

  errorList.innerHTML = `
    <h4>${errorEntries.length} Error(s)</h4>
    <ul>${html}</ul>
  `;
});
```

### **Example 7: Loading Overlay**
```javascript
const form = ReactiveUtils.form({
  email: '',
  password: ''
}, {
  onSubmit: async (values) => {
    await fetch('/api/login', {
      method: 'POST',
      body: JSON.stringify(values)
    });
  }
});

ReactiveUtils.effect(() => {
  const overlay = document.getElementById('loading-overlay');
  const formElement = document.getElementById('login-form');

  if (form.isSubmitting) {
    overlay.style.display = 'flex';
    formElement.style.opacity = '0.5';
    formElement.style.pointerEvents = 'none';
  } else {
    overlay.style.display = 'none';
    formElement.style.opacity = '1';
    formElement.style.pointerEvents = 'auto';
  }
});
```

### **Example 8: Conditional Field Display**
```javascript
const form = ReactiveUtils.form({
  accountType: 'personal',
  businessName: '',
  taxId: ''
});

ReactiveUtils.effect(() => {
  const isBusiness = form.values.accountType === 'business';
  const businessFields = document.getElementById('business-fields');

  businessFields.style.display = isBusiness ? 'block' : 'none';

  // Clear business fields if switching to personal
  if (!isBusiness) {
    form.values.businessName = '';
    form.values.taxId = '';
    delete form.touched.businessName;
    delete form.touched.taxId;
  }
});
```

### **Example 9: Validation on Blur**
```javascript
const form = ReactiveUtils.form({
  email: '',
  phone: ''
}, {
  validators: {
    email: Forms.v.email(),
    phone: Forms.v.pattern(/^\d{3}-\d{3}-\d{4}$/)
  }
});

// Only show errors for touched fields
ReactiveUtils.effect(() => {
  Object.keys(form.values).forEach(field => {
    const errorDiv = document.getElementById(`${field}-error`);
    const isTouched = form.touched[field];
    const error = form.errors[field];

    // Only show error if field was touched
    if (isTouched && error) {
      errorDiv.textContent = error;
      errorDiv.style.display = 'block';
    } else {
      errorDiv.style.display = 'none';
    }
  });
});

// Set up blur handlers
['email', 'phone'].forEach(field => {
  document.getElementById(field).addEventListener('blur', () => {
    form.validateField(field);
  });
});
```

### **Example 10: Change Tracking**
```javascript
const form = ReactiveUtils.form({
  title: 'Original Title',
  description: 'Original description'
});

const originalValues = { ...form.values };

ReactiveUtils.effect(() => {
  const changes = [];

  Object.keys(form.values).forEach(field => {
    if (form.values[field] !== originalValues[field]) {
      changes.push({
        field,
        old: originalValues[field],
        new: form.values[field]
      });
    }
  });

  const changeList = document.getElementById('changes');

  if (changes.length === 0) {
    changeList.innerHTML = '<p>No changes</p>';
    return;
  }

  const html = changes
    .map(c => `
      <li>
        <strong>${c.field}:</strong>
        "${c.old}" → "${c.new}"
      </li>
    `)
    .join('');

  changeList.innerHTML = `<ul>${html}</ul>`;
});
```

### **Example 11: Form State Debugger**
```javascript
const form = ReactiveUtils.form({
  email: '',
  password: ''
}, {
  validators: {
    email: Forms.v.email(),
    password: Forms.v.minLength(8)
  }
});

ReactiveUtils.effect(() => {
  console.clear();
  console.log('📋 Form State:');
  console.log('─'.repeat(60));

  console.log('\n📝 Values:');
  Object.entries(form.values).forEach(([field, value]) => {
    console.log(`  ${field}: "${value}"`);
  });

  console.log('\n❌ Errors:');
  const errors = Object.entries(form.errors).filter(([, e]) => e);
  if (errors.length > 0) {
    errors.forEach(([field, error]) => {
      console.log(`  ${field}: ${error}`);
    });
  } else {
    console.log('  (none)');
  }

  console.log('\n👆 Touched:');
  const touched = Object.keys(form.touched);
  if (touched.length > 0) {
    console.log(`  ${touched.join(', ')}`);
  } else {
    console.log('  (none)');
  }

  console.log('\n🚀 Submitting:', form.isSubmitting ? 'Yes' : 'No');
  console.log('─'.repeat(60));
});
```

### **Example 12: Dynamic Validation**
```javascript
const form = ReactiveUtils.form({
  password: '',
  confirmPassword: ''
}, {
  validators: {
    password: Forms.v.minLength(8),
    confirmPassword: (value) => {
      // Access form.values for dynamic validation
      return value === form.values.password
        ? null
        : 'Passwords must match';
    }
  }
});

// Re-validate confirmPassword when password changes
ReactiveUtils.effect(() => {
  const password = form.values.password;

  // If confirmPassword is touched, re-validate it
  if (form.touched.confirmPassword) {
    form.validateField('confirmPassword');
  }
});
```

### **Example 13: Multi-Step Progress**
```javascript
const form = ReactiveUtils.form({
  // Step 1
  firstName: '',
  lastName: '',
  email: '',
  // Step 2
  address: '',
  city: '',
  zipCode: '',
  // Step 3
  cardNumber: '',
  expiryDate: ''
});

const steps = {
  1: ['firstName', 'lastName', 'email'],
  2: ['address', 'city', 'zipCode'],
  3: ['cardNumber', 'expiryDate']
};

ReactiveUtils.effect(() => {
  Object.entries(steps).forEach(([stepNum, fields]) => {
    const touchedCount = fields.filter(f => form.touched[f]).length;
    const totalFields = fields.length;
    const progress = (touchedCount / totalFields) * 100;

    const progressBar = document.getElementById(`step-${stepNum}-progress`);
    progressBar.style.width = `${progress}%`;
    progressBar.textContent = `${touchedCount}/${totalFields}`;
  });
});
```

### **Example 14: Prevent Double Submit**
```javascript
const form = ReactiveUtils.form({
  email: '',
  message: ''
}, {
  onSubmit: async (values) => {
    // Simulate API call
    await new Promise(resolve => setTimeout(resolve, 2000));
    console.log('Form submitted:', values);
  }
});

document.getElementById('form').addEventListener('submit', async (e) => {
  e.preventDefault();

  // Prevent double submit
  if (form.isSubmitting) {
    console.log('Already submitting...');
    return;
  }

  await form.handleSubmit();
});

// Visual feedback
ReactiveUtils.effect(() => {
  const submitBtn = document.getElementById('submit');

  if (form.isSubmitting) {
    submitBtn.disabled = true;
    submitBtn.innerHTML = '<span class="spinner"></span> Submitting...';
  } else {
    submitBtn.disabled = false;
    submitBtn.textContent = 'Submit';
  }
});
```

### **Example 15: Unsaved Changes Modal**
```javascript
const form = ReactiveUtils.form({
  title: '',
  content: ''
});

let hasShownWarning = false;

ReactiveUtils.effect(() => {
  const hasTouchedFields = Object.keys(form.touched).length > 0;

  if (hasTouchedFields && !hasShownWarning) {
    console.log('Form has unsaved changes');
  }
});

function navigateAway() {
  const hasTouchedFields = Object.keys(form.touched).length > 0;

  if (hasTouchedFields && !form.isSubmitting) {
    const confirmed = confirm('You have unsaved changes. Are you sure?');
    if (!confirmed) {
      return false;
    }
  }

  return true;
}

// Attach to navigation
document.querySelectorAll('a').forEach(link => {
  link.addEventListener('click', (e) => {
    if (!navigateAway()) {
      e.preventDefault();
    }
  });
});
```

### **Example 16: Form Reset with Confirmation**
```javascript
const form = ReactiveUtils.form({
  name: '',
  email: '',
  message: ''
});

function resetForm() {
  const touchedCount = Object.keys(form.touched).length;

  if (touchedCount > 0) {
    const confirmed = confirm(
      `This will reset ${touchedCount} field(s). Continue?`
    );

    if (!confirmed) {
      return;
    }
  }

  form.reset();
  console.log('Form reset');
  console.log('Values:', form.values); // Back to initial values
  console.log('Touched:', form.touched); // Empty
  console.log('Errors:', form.errors); // Empty
}

document.getElementById('reset-btn').addEventListener('click', resetForm);
```

### **Example 17: Field Value Transformation**
```javascript
const form = ReactiveUtils.form({
  email: '',
  phone: '',
  zipCode: ''
});

// Auto-transform values
ReactiveUtils.effect(() => {
  // Lowercase email
  if (form.values.email !== form.values.email.toLowerCase()) {
    form.values.email = form.values.email.toLowerCase();
  }

  // Format phone: (123) 456-7890
  const phone = form.values.phone.replace(/\D/g, '');
  if (phone.length === 10 && !form.values.phone.includes('(')) {
    form.values.phone = `(${phone.slice(0, 3)}) ${phone.slice(3, 6)}-${phone.slice(6)}`;
  }

  // Uppercase zip code
  if (form.values.zipCode !== form.values.zipCode.toUpperCase()) {
    form.values.zipCode = form.values.zipCode.toUpperCase();
  }
});
```

### **Example 18: Error Recovery**
```javascript
const form = ReactiveUtils.form({
  email: '',
  username: ''
}, {
  validators: {
    email: Forms.v.email(),
    username: Forms.v.minLength(3)
  },
  onSubmit: async (values) => {
    const response = await fetch('/api/register', {
      method: 'POST',
      body: JSON.stringify(values)
    });

    if (!response.ok) {
      const data = await response.json();

      // Set server-side errors
      if (data.errors) {
        form.setErrors(data.errors);
      }

      throw new Error('Registration failed');
    }
  }
});

ReactiveUtils.effect(() => {
  // Clear server errors when user starts typing
  Object.keys(form.values).forEach(field => {
    if (form.touched[field] && form.errors[field]) {
      // User is fixing the error
      console.log(`User is fixing ${field} error`);
    }
  });
});
```

### **Example 19: Form State Persistence**
```javascript
const STORAGE_KEY = 'form-draft';

const form = ReactiveUtils.form({
  title: '',
  content: '',
  tags: ''
});

// Load from localStorage on init
const savedData = localStorage.getItem(STORAGE_KEY);
if (savedData) {
  const { values, touched } = JSON.parse(savedData);
  form.setValues(values);

  // Restore touched state
  Object.keys(touched).forEach(field => {
    form.touched[field] = true;
  });
}

// Save to localStorage on change
let saveTimeout;
ReactiveUtils.effect(() => {
  if (Object.keys(form.touched).length > 0) {
    clearTimeout(saveTimeout);

    saveTimeout = setTimeout(() => {
      const state = {
        values: form.values,
        touched: form.touched,
        timestamp: Date.now()
      };

      localStorage.setItem(STORAGE_KEY, JSON.stringify(state));
      console.log('Draft saved');
    }, 1000);
  }
});

// Clear on successful submit
form.onSubmitSuccess = () => {
  localStorage.removeItem(STORAGE_KEY);
};
```

### **Example 20: Complex Form State Dashboard**
```javascript
const form = ReactiveUtils.form({
  personalInfo: {
    firstName: '',
    lastName: '',
    email: ''
  },
  preferences: {
    newsletter: false,
    notifications: true
  }
}, {
  validators: {
    'personalInfo.email': Forms.v.email()
  }
});

ReactiveUtils.effect(() => {
  const dashboard = {
    // Values stats
    totalFields: Object.keys(flattenObject(form.values)).length,
    filledFields: Object.values(flattenObject(form.values))
      .filter(v => v !== '' && v !== false && v !== null).length,

    // Error stats
    totalErrors: Object.keys(form.errors).length,
    activeErrors: Object.values(form.errors).filter(e => e).length,

    // Touched stats
    touchedCount: Object.keys(form.touched).length,

    // Submission
    isSubmitting: form.isSubmitting,
    canSubmit: form.isValid && form.isDirty && !form.isSubmitting
  };

  console.log('\n📊 Form Dashboard:');
  console.log('─'.repeat(60));
  console.log(`Fields: ${dashboard.filledFields}/${dashboard.totalFields} filled`);
  console.log(`Errors: ${dashboard.activeErrors} active`);
  console.log(`Touched: ${dashboard.touchedCount} fields`);
  console.log(`Submitting: ${dashboard.isSubmitting ? 'Yes' : 'No'}`);
  console.log(`Can Submit: ${dashboard.canSubmit ? 'Yes' : 'No'}`);
  console.log('─'.repeat(60));
});

function flattenObject(obj, prefix = '') {
  return Object.keys(obj).reduce((acc, key) => {
    const path = prefix ? `${prefix}.${key}` : key;
    if (typeof obj[key] === 'object' && obj[key] !== null && !Array.isArray(obj[key])) {
      Object.assign(acc, flattenObject(obj[key], path));
    } else {
      acc[path] = obj[key];
    }
    return acc;
  }, {});
}
```

---

## **Common Patterns**

### **Pattern 1: Read Field Value**
```javascript
const value = form.values.fieldName;
```

### **Pattern 2: Check Field Error**
```javascript
const error = form.errors.fieldName;
if (error) {
  console.log('Error:', error);
}
```

### **Pattern 3: Check Touched State**
```javascript
const isTouched = form.touched.fieldName === true;
```

### **Pattern 4: Show Error Only If Touched**
```javascript
if (form.touched.fieldName && form.errors.fieldName) {
  showError(form.errors.fieldName);
}
```

### **Pattern 5: Disable Submit During Submission**
```javascript
submitButton.disabled = form.isSubmitting;
```

---

## **Property Structures**

### **`values` Object**
```javascript
{
  fieldName1: 'value1',
  fieldName2: 'value2',
  fieldName3: true,
  nested: {
    field: 'value'
  }
}
```

### **`errors` Object**
```javascript
{
  fieldName1: 'Error message',
  fieldName2: null,           // Explicitly no error
  fieldName3: 'Another error'
  // fieldName4 not present = not validated
}
```

### **`touched` Object**
```javascript
{
  fieldName1: true,
  fieldName2: true
  // Other fields not present = not touched
}
```

### **`isSubmitting` Boolean**
```javascript
false  // Not submitting
true   // Currently submitting
```

---

## **Property Comparison Table**

| Property | Type | Default | Tracks | Use Case |
|----------|------|---------|--------|----------|
| `values` | Object | Initial values | Field values | Read/write form data |
| `errors` | Object | `{}` | Validation errors | Display error messages |
| `touched` | Object | `{}` | Modified fields | Conditional error display |
| `isSubmitting` | Boolean | `false` | Submit state | Disable controls during submit |

---

## **State Lifecycle**

```javascript
// 1. Initial state
form.values      // { email: '', password: '' }
form.errors      // {}
form.touched     // {}
form.isSubmitting // false

// 2. User enters email
form.setValue('email', 'test@example.com')
form.values      // { email: 'test@example.com', password: '' }
form.touched     // { email: true }

// 3. Validation runs
form.validateField('email')
form.errors      // { email: null } or {}

// 4. Submit starts
form.handleSubmit()
form.isSubmitting // true

// 5. Submit completes
form.isSubmitting // false

// 6. Reset
form.reset()
form.values      // { email: '', password: '' }
form.errors      // {}
form.touched     // {}
form.isSubmitting // false
```

---

## **Direct Access vs Methods**

### **Direct Access** (Reactive)
```javascript
form.values.email = 'test@example.com';    // Sets value
form.errors.email = 'Invalid email';        // Sets error
form.touched.email = true;                  // Marks touched
form.isSubmitting = true;                   // Sets submitting
```

### **Method Access** (Recommended)
```javascript
form.setValue('email', 'test@example.com'); // Sets value + touched
form.setError('email', 'Invalid email');    // Sets error
form.setTouched('email', true);             // Marks touched
form.handleSubmit();                        // Handles submission
```

**Why methods?**
- Cleaner API
- Batch operations
- Additional logic (auto-validation, etc.)
- Type safety
- Consistency

---

## **Reactive Behavior**

All four properties are reactive and trigger effects:

```javascript
const form = ReactiveUtils.form({
  email: ''
}, {
  validators: { email: Forms.v.email() }
});

// Track all properties
ReactiveUtils.effect(() => {
  console.log('Values:', form.values);
  console.log('Errors:', form.errors);
  console.log('Touched:', form.touched);
  console.log('Submitting:', form.isSubmitting);
});

// Any change triggers the effect
form.setValue('email', 'test@example.com'); // Effect runs
form.validateField('email');                 // Effect runs
form.handleSubmit();                         // Effect runs (twice)
```

---

## **Key Takeaways**

1. **`values`** - Object containing all form field values
2. **`errors`** - Object mapping fields to error messages
3. **`touched`** - Object tracking modified fields
4. **`isSubmitting`** - Boolean for submission state
5. **Reactive** - All properties trigger effects on change
6. **Mutable** - Can be modified directly or via methods
7. **Methods Preferred** - Use setValue, setError, etc.
8. **Predictable Structure** - Consistent object shapes
9. **Tracked** - Dependency tracking for all changes
10. **Lifecycle** - Clear progression from initial to submitted

---

## **Summary**

Form state properties represent the core reactive state of a form through four essential properties: `values` (object containing all field values), `errors` (object mapping fields to validation error messages), `touched` (object tracking which fields have been modified), and `isSubmitting` (boolean indicating submission state). All four properties are reactive and automatically trigger effects when modified, enabling real-time UI updates without manual tracking. The `values` object stores the actual form data and can be accessed directly or via methods like `setValue()`. The `errors` object contains validation messages where truthy values indicate errors and null/undefined indicates validity. The `touched` object tracks user interaction with fields, crucial for showing errors only after user engagement. The `isSubmitting` boolean prevents double submissions and enables loading states during async operations. While direct property access is supported, using form methods (`setValue`, `setError`, `setTouched`, `handleSubmit`) is recommended for cleaner code, batch operations, and additional built-in logic like auto-validation. These properties work together to provide complete form state management with minimal boilerplate.
