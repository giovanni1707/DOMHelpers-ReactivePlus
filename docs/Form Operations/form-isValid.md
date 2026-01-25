# `form.isValid` - Form Validity Computed Property

**Quick Start (30 seconds)**
```javascript
const form = Forms.createForm({ email: '', password: '' }, {
  validators: {
    email: Validators.email(),
    password: Validators.minLength(8)
  }
});

// Access the computed property
console.log(form.isValid); // true (no errors yet)

form.setValue('email', 'invalid');
form.validateField('email');
console.log(form.isValid); // false (validation failed)

form.setValue('email', 'user@example.com');
form.validateField('email');
console.log(form.isValid); // true (all valid)
```

---

## **What is `form.isValid`?**

`form.isValid` is a **reactive computed property** that automatically indicates whether the form is currently valid. It returns `true` when there are no validation errors or all error values are falsy, and `false` otherwise.

**Key characteristics:**
- **Computed**: Automatically recalculates when errors change
- **Reactive**: Triggers effects when accessed in reactive contexts
- **Read-only**: A getter property, not a method
- **Boolean**: Always returns `true` or `false`
- **Real-time**: Updates immediately when validation state changes

---

## **Syntax**

```javascript
form.isValid
```

### **Returns**
- **Type**: `Boolean`
- **Value**:
  - `true` - No errors exist, or all error values are falsy
  - `false` - At least one error value is truthy

---

## **Why does `form.isValid` exist?**

### ❌ **The Problem**
```javascript
// Manual validity checking is verbose and error-prone
const form = Forms.createForm({ email: '', name: '' });

// You'd need to manually check errors every time
function isFormValid() {
  const errorKeys = Object.keys(form.errors);
  if (errorKeys.length === 0) return true;
  return errorKeys.every(key => !form.errors[key]);
}

// Have to call this repeatedly
if (isFormValid()) {
  submitForm();
}

// Easy to forget to check all error keys
if (!form.errors.email && !form.errors.name) { // Fragile!
  submitForm();
}
```

### ✅ **The Solution**
```javascript
// Computed property handles the complexity
const form = Forms.createForm({ email: '', name: '' });

// Clean, simple, always up-to-date
if (form.isValid) {
  submitForm();
}

// Automatically updates when errors change
effect(() => {
  submitButton.disabled = !form.isValid;
  // Re-runs whenever validation state changes
});
```

---

## **Mental Model**

Think of `form.isValid` as a **traffic light** that automatically changes color based on your form's validation state:

```
┌─────────────────────────────────────┐
│          FORM STATE                 │
├─────────────────────────────────────┤
│ errors: {}                          │ ──→ 🟢 isValid = true
│                                     │
├─────────────────────────────────────┤
│ errors: {                           │
│   email: null,                      │ ──→ 🟢 isValid = true
│   password: null                    │     (all errors are falsy)
│ }                                   │
├─────────────────────────────────────┤
│ errors: {                           │
│   email: "Invalid email",           │ ──→ 🔴 isValid = false
│   password: null                    │     (at least one error)
│ }                                   │
└─────────────────────────────────────┘
```

**The formula:**
```
isValid = (no error keys) OR (all error values are falsy)
```

---

## **How does it work?**

The `isValid` property is implemented as a computed property:

```javascript
form.$computed('isValid', function() {
  const errorKeys = Object.keys(this.errors);
  return errorKeys.length === 0 || errorKeys.every(k => !this.errors[k]);
});
```

**The algorithm:**
1. Get all keys from the `errors` object
2. If there are no keys → return `true` (no errors)
3. Otherwise, check if every error value is falsy → return that result
4. Automatically recalculates whenever `this.errors` changes

---

## **Examples**

### **Example 1: Conditional Submit Button**
```javascript
const form = Forms.createForm({ username: '', email: '' }, {
  validators: {
    username: Validators.required(),
    email: Validators.email()
  }
});

const submitBtn = document.querySelector('#submit');

// Disable button when form is invalid
effect(() => {
  submitBtn.disabled = !form.isValid;
  submitBtn.className = form.isValid ? 'btn-primary' : 'btn-disabled';
});

// Initially disabled (fields are empty)
// Becomes enabled when all validations pass
```

### **Example 2: Form Validation Indicator**
```javascript
const form = Forms.createForm({
  email: '',
  password: '',
  confirmPassword: ''
}, {
  validators: {
    email: Validators.email(),
    password: Validators.minLength(8),
    confirmPassword: Validators.match('password')
  }
});

effect(() => {
  const indicator = document.querySelector('#form-status');

  if (form.isValid) {
    indicator.textContent = '✓ Form is valid';
    indicator.className = 'success';
  } else {
    indicator.textContent = '✗ Please fix errors';
    indicator.className = 'error';
  }
});
```

### **Example 3: Multi-Step Form**
```javascript
const step1Form = Forms.createForm({ name: '', email: '' }, {
  validators: {
    name: Validators.required(),
    email: Validators.email()
  }
});

const nextButton = document.querySelector('#next-step');

// Only allow proceeding when current step is valid
nextButton.addEventListener('click', () => {
  step1Form.validate(); // Trigger validation

  if (step1Form.isValid) {
    goToNextStep();
  } else {
    alert('Please complete all required fields');
  }
});
```

### **Example 4: Real-time Form Summary**
```javascript
const form = Forms.createForm({
  name: '',
  email: '',
  age: ''
}, {
  validators: {
    name: Validators.required(),
    email: Validators.email(),
    age: Validators.min(18)
  }
});

effect(() => {
  const stats = {
    valid: form.isValid,
    dirty: form.isDirty,
    errorCount: form.errorFields.length,
    touchedCount: form.touchedFields.length
  };

  console.log('Form stats:', stats);
  updateDashboard(stats);
});
```

### **Example 5: Conditional API Call**
```javascript
const loginForm = Forms.createForm({ username: '', password: '' }, {
  validators: {
    username: Validators.required('Username is required'),
    password: Validators.minLength(6, 'Password must be 6+ characters')
  },
  async onSubmit(values) {
    const response = await fetch('/api/login', {
      method: 'POST',
      body: JSON.stringify(values)
    });
    return response.json();
  }
});

// Only submit if valid
async function handleSubmit(e) {
  e.preventDefault();

  loginForm.validate(); // Run all validators

  if (loginForm.isValid) {
    await loginForm.submit(); // Safe to submit
  }
}
```

### **Example 6: Form with Optional Fields**
```javascript
const form = Forms.createForm({
  email: '',          // Required
  phone: '',          // Optional
  newsletter: false   // Optional
}, {
  validators: {
    email: Validators.email('Valid email required'),
    // No validator for optional fields
  }
});

// isValid only checks validated fields
form.setValue('email', 'user@example.com');
form.validateField('email');

console.log(form.isValid); // true
// phone and newsletter don't affect validity
```

### **Example 7: Validation State Badge**
```javascript
const form = Forms.createForm({
  title: '',
  content: ''
}, {
  validators: {
    title: Validators.required(),
    content: Validators.minLength(10)
  }
});

function FormValidationBadge() {
  effect(() => {
    const badge = document.querySelector('#validation-badge');

    if (!form.isDirty) {
      badge.innerHTML = '○ Not started';
      badge.className = 'badge-neutral';
    } else if (form.isValid) {
      badge.innerHTML = '✓ Valid';
      badge.className = 'badge-success';
    } else {
      badge.innerHTML = `✗ ${form.errorFields.length} errors`;
      badge.className = 'badge-error';
    }
  });
}
```

### **Example 8: Form State Machine**
```javascript
const form = Forms.createForm({ name: '', email: '' }, {
  validators: {
    name: Validators.required(),
    email: Validators.email()
  }
});

function getFormState() {
  if (!form.isDirty) return 'pristine';
  if (form.isSubmitting) return 'submitting';
  if (!form.isValid) return 'invalid';
  if (form.isDirty && form.isValid) return 'ready';
  return 'unknown';
}

effect(() => {
  const state = getFormState();
  console.log('Form state:', state);

  // Update UI based on state
  updateUIForState(state);
});
```

### **Example 9: Save Draft vs Submit**
```javascript
const form = Forms.createForm({ title: '', content: '' }, {
  validators: {
    title: Validators.required(),
    content: Validators.minLength(100)
  }
});

// Save draft - no validation required
function saveDraft() {
  if (form.isDirty) {
    localStorage.setItem('draft', JSON.stringify(form.values));
  }
}

// Submit - requires valid form
function submitPost() {
  form.validate();

  if (form.isValid) {
    fetch('/api/posts', {
      method: 'POST',
      body: JSON.stringify(form.values)
    });
  } else {
    alert('Please fix validation errors before submitting');
  }
}
```

### **Example 10: Progressive Enhancement**
```javascript
const form = Forms.createForm({
  search: '',
  filters: { category: '', minPrice: 0 }
}, {
  validators: {
    search: Validators.minLength(2, 'Search must be 2+ characters')
  }
});

// Auto-search when form is valid
effect(() => {
  if (form.isValid && form.isDirty) {
    performSearch(form.values);
  }
});

// Manual search ignores validation
function forceSearch() {
  performSearch(form.values); // Search even if invalid
}
```

---

## **Common Patterns**

### **Pattern 1: Submit Button State**
```javascript
effect(() => {
  submitButton.disabled = !form.isValid || form.isSubmitting;
});
```

### **Pattern 2: Validation Feedback**
```javascript
effect(() => {
  if (form.isDirty && !form.isValid) {
    showValidationErrors(form.errorFields);
  }
});
```

### **Pattern 3: Guard Clause**
```javascript
async function handleSubmit() {
  form.validate();
  if (!form.isValid) return; // Early exit

  await form.submit();
}
```

### **Pattern 4: Valid + Dirty Check**
```javascript
effect(() => {
  saveButton.hidden = !(form.isValid && form.isDirty);
  // Only show save when there are valid changes
});
```

---

## **Comparison Table**

| Scenario | `isValid` | `hasErrors` | Description |
|----------|-----------|-------------|-------------|
| No errors object keys | `true` | `false` | Clean form |
| All errors are `null` | `true` | `false` | Validated, all passed |
| Mix of `null` and errors | `false` | `true` | Some fields invalid |
| All errors have messages | `false` | `true` | Multiple failures |

```javascript
// Scenario examples
form.errors = {};                    // isValid: true,  hasErrors: false
form.errors = { email: null };       // isValid: true,  hasErrors: false
form.errors = { email: 'Invalid' };  // isValid: false, hasErrors: true
```

---

## **Key Takeaways**

1. **Reactive Computed Property**: `isValid` automatically updates when errors change
2. **Boolean Result**: Always returns `true` or `false`, never undefined
3. **Falsy-Aware**: Treats `null`, `undefined`, `''`, `false` as valid (no error)
4. **Not a Method**: Access as `form.isValid`, not `form.isValid()`
5. **Validation Required**: Doesn't run validators, just checks current error state
6. **Common Use**: Disable/enable submit buttons, guard clauses, conditional logic
7. **Works with Effects**: Triggers re-runs when validation state changes
8. **Complements `hasErrors`**: `isValid` is essentially `!hasErrors` (with edge cases)

---

## **Summary**

`form.isValid` is a reactive computed property that returns `true` when your form has no validation errors (or all error values are falsy), and `false` otherwise. It automatically updates when the form's error state changes, making it perfect for controlling submit buttons, showing validation indicators, and creating conditional logic based on form validity. Use it anywhere you need to check if a form is ready to submit or to provide real-time validation feedback to users.
