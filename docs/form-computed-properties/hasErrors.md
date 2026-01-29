# `form.hasErrors` - Form Error Presence Computed Property

**Quick Start (30 seconds)**
```javascript
const form = Forms.createForm({ email: '' }, {
  validators: {
    email: Validators.email()
  }
});

console.log(form.hasErrors); // false (no errors yet)

form.setValue('email', 'invalid');
form.validateField('email');
console.log(form.hasErrors); // true (validation failed)

form.setValue('email', 'user@example.com');
form.validateField('email');
console.log(form.hasErrors); // false (validation passed)
```

---

## **What is `form.hasErrors`?**

`form.hasErrors` is a **reactive computed property** that indicates whether the form currently has any validation errors. It returns `true` if at least one error exists (truthy error value), and `false` if there are no errors or all errors are falsy.

**Key characteristics:**
- **Computed**: Automatically recalculates when errors change
- **Reactive**: Triggers effects when accessed in reactive contexts
- **Read-only**: A getter property, not a method
- **Boolean**: Always returns `true` or `false`
- **Truthy-based**: Only counts truthy error values (not `null`, `undefined`, `''`, `false`)

---

## **Syntax**

```javascript
form.hasErrors
```

### **Returns**
- **Type**: `Boolean`
- **Value**:
  - `true` - At least one error value is truthy
  - `false` - No errors or all error values are falsy

---

## **Why does `form.hasErrors` exist?**

### ❌ **The Problem**
```javascript
// Manual error checking is verbose
const form = Forms.createForm({ email: '', password: '' });

// You'd need to manually check errors
function formHasErrors() {
  return Object.keys(form.errors).some(key => form.errors[key]);
}

// Repetitive checks everywhere
if (formHasErrors()) {
  showErrorSummary();
}

// Easy to make mistakes
if (form.errors.email || form.errors.password) { // Fragile!
  showErrorSummary();
}
```

### ✅ **The Solution**
```javascript
// Computed property handles it automatically
const form = Forms.createForm({ email: '', password: '' });

// Clean, simple, always accurate
if (form.hasErrors) {
  showErrorSummary();
}

// Reactive updates
effect(() => {
  errorBanner.style.display = form.hasErrors ? 'block' : 'none';
});
```

---

## **Mental Model**

Think of `form.hasErrors` as an **alarm that triggers** when any validation fails:

```
┌─────────────────────────────────────┐
│          ERROR STATE                │
├─────────────────────────────────────┤
│ errors: {}                          │ ──→ 🟢 hasErrors = false
│                                     │     (no errors)
│                                     │
├─────────────────────────────────────┤
│ errors: {                           │
│   email: null,                      │ ──→ 🟢 hasErrors = false
│   password: null                    │     (all errors are falsy)
│ }                                   │
├─────────────────────────────────────┤
│ errors: {                           │
│   email: "Invalid email",           │ ──→ 🔴 hasErrors = true
│   password: null                    │     (at least one truthy error)
│ }                                   │
├─────────────────────────────────────┤
│ errors: {                           │
│   email: "Invalid email",           │ ──→ 🔴 hasErrors = true
│   password: "Too short"             │     (multiple errors)
│ }                                   │
└─────────────────────────────────────┘
```

**The formula:**
```
hasErrors = some error value is truthy
```

---

## **How does it work?**

The `hasErrors` property is implemented as a computed property:

```javascript
form.$computed('hasErrors', function() {
  return Object.keys(this.errors).some(k => this.errors[k]);
});
```

**The algorithm:**
1. Get all keys from the `errors` object
2. Check if any error value is truthy using `.some()`
3. Return `true` if any error is truthy, `false` otherwise
4. Automatically recalculates whenever `this.errors` changes

---

## **Examples**

### **Example 1: Error Banner**
```javascript
const form = Forms.createForm({ email: '', name: '' }, {
  validators: {
    email: Validators.email(),
    name: Validators.required()
  }
});

const errorBanner = querySelector('#error-banner');

effect(() => {
  if (form.hasErrors) {
    errorBanner.style.display = 'block';
    errorBanner.textContent = `${form.errorFields.length} validation error(s)`;
  } else {
    errorBanner.style.display = 'none';
  }
});
```

### **Example 2: Form Submission Guard**
```javascript
const form = Forms.createForm({ username: '', password: '' }, {
  validators: {
    username: Validators.required(),
    password: Validators.minLength(8)
  }
});

async function handleSubmit(e) {
  e.preventDefault();

  form.validate(); // Run all validators

  if (form.hasErrors) {
    alert('Please fix the errors before submitting');
    return;
  }

  await form.submit();
}
```

### **Example 3: Error Summary List**
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
  const summaryEl = querySelector('#error-summary');

  if (form.hasErrors) {
    const errorList = form.errorFields
      .map(field => `<li>${field}: ${form.errors[field]}</li>`)
      .join('');

    summaryEl.innerHTML = `<ul>${errorList}</ul>`;
    summaryEl.style.display = 'block';
  } else {
    summaryEl.style.display = 'none';
  }
});
```

### **Example 4: Progress Indicator**
```javascript
const form = Forms.createForm({ step1: '', step2: '', step3: '' }, {
  validators: {
    step1: Validators.required(),
    step2: Validators.required(),
    step3: Validators.required()
  }
});

effect(() => {
  const progress = querySelector('#progress');

  if (!form.isDirty) {
    progress.textContent = 'Not started';
  } else if (form.hasErrors) {
    progress.textContent = `In progress (${form.errorFields.length} remaining)`;
  } else {
    progress.textContent = 'Complete ✓';
  }
});
```

### **Example 5: Conditional Styling**
```javascript
const form = Forms.createForm({ email: '', phone: '' }, {
  validators: {
    email: Validators.email(),
    phone: Validators.pattern(/^\d{10}$/)
  }
});

const formEl = querySelector('#myForm');

effect(() => {
  if (form.hasErrors) {
    formEl.classList.add('has-errors');
    formEl.classList.remove('all-valid');
  } else if (form.isDirty) {
    formEl.classList.add('all-valid');
    formEl.classList.remove('has-errors');
  } else {
    formEl.classList.remove('has-errors', 'all-valid');
  }
});
```

### **Example 6: Error Count Badge**
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
  const badge = querySelector('#error-badge');

  if (form.hasErrors) {
    const count = form.errorFields.length;
    badge.textContent = count;
    badge.style.display = 'inline-block';
  } else {
    badge.style.display = 'none';
  }
});
```

### **Example 7: Submit Button Text**
```javascript
const form = Forms.createForm({ email: '', password: '' }, {
  validators: {
    email: Validators.email(),
    password: Validators.minLength(8)
  }
});

effect(() => {
  const submitBtn = querySelector('#submit');

  if (form.isSubmitting) {
    submitBtn.textContent = 'Submitting...';
    submitBtn.disabled = true;
  } else if (form.hasErrors) {
    submitBtn.textContent = 'Fix errors to continue';
    submitBtn.disabled = true;
  } else if (form.isDirty) {
    submitBtn.textContent = 'Submit';
    submitBtn.disabled = false;
  } else {
    submitBtn.textContent = 'No changes';
    submitBtn.disabled = true;
  }
});
```

### **Example 8: Multi-Step Form Navigation**
```javascript
const form = Forms.createForm({
  personalInfo: '',
  address: '',
  payment: ''
}, {
  validators: {
    personalInfo: Validators.required(),
    address: Validators.required(),
    payment: Validators.required()
  }
});

function canProceedToNextStep(currentStep) {
  // Validate only current step fields
  form.validateField(currentStep);

  if (form.hasErrors) {
    alert(`Please fix errors in ${currentStep} before continuing`);
    return false;
  }

  return true;
}
```

### **Example 9: Error vs Success Message**
```javascript
const form = Forms.createForm({ username: '' }, {
  validators: {
    username: Validators.custom(async (value) => {
      const available = await checkUsernameAvailability(value);
      return available ? null : 'Username taken';
    })
  }
});

effect(() => {
  const message = querySelector('#validation-message');

  if (!form.isDirty) {
    message.textContent = '';
  } else if (form.hasErrors) {
    message.textContent = form.errors.username;
    message.className = 'error';
  } else {
    message.textContent = 'Username available!';
    message.className = 'success';
  }
});
```

### **Example 10: Form Analytics**
```javascript
const form = Forms.createForm({ email: '', name: '', phone: '' }, {
  validators: {
    email: Validators.email(),
    name: Validators.required(),
    phone: Validators.pattern(/^\d{10}$/)
  }
});

effect(() => {
  const analytics = {
    hasErrors: form.hasErrors,
    errorCount: form.errorFields.length,
    isValid: form.isValid,
    isDirty: form.isDirty,
    touchedCount: form.touchedFields.length
  };

  console.log('Form analytics:', analytics);
  sendAnalytics('form_state', analytics);
});
```

---

## **Common Patterns**

### **Pattern 1: Error Banner Toggle**
```javascript
effect(() => {
  errorBanner.style.display = form.hasErrors ? 'block' : 'none';
});
```

### **Pattern 2: Submit Validation**
```javascript
function handleSubmit() {
  form.validate();
  if (form.hasErrors) return; // Guard clause

  submitForm();
}
```

### **Pattern 3: Error List Rendering**
```javascript
effect(() => {
  if (form.hasErrors) {
    showErrorList(form.errorFields.map(f => form.errors[f]));
  }
});
```

### **Pattern 4: Combined State Check**
```javascript
effect(() => {
  submitBtn.disabled = form.hasErrors || !form.isDirty || form.isSubmitting;
});
```

---

## **Comparison with `isValid`**

| Property | When `true` | When `false` |
|----------|-------------|--------------|
| `hasErrors` | At least one error exists | No errors or all errors falsy |
| `isValid` | No errors or all errors falsy | At least one error exists |

```javascript
// They are nearly opposite (with edge cases for falsy errors)
form.errors = {};                    // hasErrors: false, isValid: true
form.errors = { email: null };       // hasErrors: false, isValid: true
form.errors = { email: 'Invalid' };  // hasErrors: true,  isValid: false

// Relationship
form.hasErrors === !form.isValid  // Usually true (but not always!)
```

**Edge case:**
```javascript
// When all errors are falsy but error keys exist
form.errors = { email: null, name: '' };
form.hasErrors;  // false (no truthy errors)
form.isValid;    // true (all errors are falsy)
```

---

## **Key Takeaways**

1. **Reactive Computed Property**: `hasErrors` automatically updates when errors change
2. **Boolean Result**: Always returns `true` or `false`
3. **Truthy Check**: Only counts truthy error values, ignores falsy ones
4. **Nearly Opposite of `isValid`**: `hasErrors ≈ !isValid` (with edge cases)
5. **Common Use**: Error banners, validation summaries, submit guards
6. **Works with Effects**: Triggers re-runs when error state changes
7. **Pairs with `errorFields`**: Use together to show which fields have errors
8. **Validation Required**: Doesn't run validators, just checks current error state

---

## **Summary**

`form.hasErrors` is a reactive computed property that returns `true` when the form has any validation errors (truthy error values), and `false` otherwise. It automatically updates when the form's error state changes, making it perfect for showing error banners, creating validation summaries, guarding form submission, and providing real-time feedback about form validity. Use it to improve UX by clearly indicating when user action is needed to fix validation issues.
