# `form.errorFields` - Error Fields Array Computed Property

**Quick Start (30 seconds)**
```javascript
const form = Forms.createForm({ email: '', name: '' }, {
  validators: {
    email: Validators.email(),
    name: Validators.required()
  }
});

console.log(form.errorFields); // [] (no errors yet)

form.setValue('email', 'invalid');
form.validateField('email');
console.log(form.errorFields); // ['email']

form.setValue('name', '');
form.validateField('name');
console.log(form.errorFields); // ['email', 'name']

form.setValue('email', 'user@example.com');
form.validateField('email');
console.log(form.errorFields); // ['name'] (only name has error now)
```

---

## **What is `form.errorFields`?**

`form.errorFields` is a **reactive computed property** that returns an array of field names that currently have validation errors (truthy error values). It provides a convenient way to get a list of all fields that failed validation.

**Key characteristics:**
- **Computed**: Automatically recalculates when errors change
- **Reactive**: Triggers effects when accessed in reactive contexts
- **Read-only**: A getter property, not a method
- **Array**: Returns an array of strings (field names)
- **Filtered**: Only includes fields with truthy error values

---

## **Syntax**

```javascript
form.errorFields
```

### **Returns**
- **Type**: `Array<string>`
- **Value**: Array of field names that have truthy error values
- **Empty Array**: `[]` when no fields have errors

---

## **Why does `form.errorFields` exist?**

### ❌ **The Problem**
```javascript
// Manual extraction of error field names is verbose
const form = Forms.createForm({ email: '', name: '' });

// You'd need to manually filter error keys
function getErrorFields() {
  return Object.keys(form.errors).filter(k => form.errors[k]);
}

// Repetitive and error-prone
const errorCount = Object.keys(form.errors).filter(k => form.errors[k]).length;
const errorList = Object.keys(form.errors).filter(k => form.errors[k]).join(', ');

// Easy to forget the filter step
const allKeys = Object.keys(form.errors); // WRONG - includes null errors!
```

### ✅ **The Solution**
```javascript
// Computed property gives you filtered array directly
const form = Forms.createForm({ email: '', name: '' });

// Clean, automatic, always accurate
const errorCount = form.errorFields.length;
const errorList = form.errorFields.join(', ');

// Reactive updates
effect(() => {
  console.log('Fields with errors:', form.errorFields);
});
```

---

## **Mental Model**

Think of `form.errorFields` as a **red flag list** that shows which fields need attention:

```
┌──────────────────────────────────────────────┐
│          VALIDATION TIMELINE                 │
├──────────────────────────────────────────────┤
│ 1. Form created, no validation yet           │
│    errors: {}                                │
│    errorFields: []                           │
│                                              │
├──────────────────────────────────────────────┤
│ 2. Email validated - failed                  │
│    errors: { email: 'Invalid email' }        │
│    errorFields: ['email']                    │
│                                              │
├──────────────────────────────────────────────┤
│ 3. Name validated - failed too               │
│    errors: {                                 │
│      email: 'Invalid email',                 │
│      name: 'Required'                        │
│    }                                         │
│    errorFields: ['email', 'name']            │
│                                              │
├──────────────────────────────────────────────┤
│ 4. Email fixed, name still has error         │
│    errors: {                                 │
│      email: null,                            │
│      name: 'Required'                        │
│    }                                         │
│    errorFields: ['name']                     │
│    (email has null error, so not included)   │
│                                              │
├──────────────────────────────────────────────┤
│ 5. All errors fixed                          │
│    errors: { email: null, name: null }       │
│    errorFields: []                           │
└──────────────────────────────────────────────┘
```

**The formula:**
```
errorFields = Object.keys(errors).filter(k => errors[k])
```

---

## **How does it work?**

The `errorFields` property is implemented as a computed property:

```javascript
form.$computed('errorFields', function() {
  return Object.keys(this.errors).filter(k => this.errors[k]);
});
```

**The algorithm:**
1. Get all keys from the `this.errors` object
2. Filter to only include keys where the error value is truthy
3. Return the filtered array
4. Automatically recalculates whenever `this.errors` changes

---

## **Examples**

### **Example 1: Error Count Badge**
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
  const badge = document.querySelector('#error-badge');
  const count = form.errorFields.length;

  if (count > 0) {
    badge.textContent = count;
    badge.style.display = 'inline-block';
    badge.title = `${count} validation error(s)`;
  } else {
    badge.style.display = 'none';
  }
});
```

### **Example 2: Error Summary List**
```javascript
const form = Forms.createForm({
  name: '',
  email: '',
  phone: ''
}, {
  validators: {
    name: Validators.required('Name is required'),
    email: Validators.email('Invalid email address'),
    phone: Validators.pattern(/^\d{10}$/, '10 digits required')
  }
});

effect(() => {
  const errorList = document.querySelector('#error-summary');

  if (form.errorFields.length === 0) {
    errorList.innerHTML = '<p class="success">All fields valid ✓</p>';
  } else {
    const items = form.errorFields
      .map(field => `<li><strong>${field}:</strong> ${form.errors[field]}</li>`)
      .join('');

    errorList.innerHTML = `
      <p class="error">${form.errorFields.length} error(s) found:</p>
      <ul>${items}</ul>
    `;
  }
});
```

### **Example 3: Focus First Error Field**
```javascript
const form = Forms.createForm({
  username: '',
  email: '',
  password: ''
}, {
  validators: {
    username: Validators.required(),
    email: Validators.email(),
    password: Validators.minLength(8)
  }
});

function handleSubmit(e) {
  e.preventDefault();

  form.validate(); // Run all validators

  if (form.errorFields.length > 0) {
    // Focus the first field with an error
    const firstErrorField = form.errorFields[0];
    const input = document.querySelector(`input[name="${firstErrorField}"]`);

    if (input) {
      input.focus();
      input.scrollIntoView({ behavior: 'smooth', block: 'center' });
    }

    return;
  }

  // Form is valid, proceed with submission
  form.submit();
}
```

### **Example 4: Highlight Error Fields**
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
  // Remove all error highlights
  document.querySelectorAll('input').forEach(input => {
    input.classList.remove('error');
  });

  // Add error class to fields with errors
  form.errorFields.forEach(field => {
    const input = document.querySelector(`input[name="${field}"]`);
    if (input) {
      input.classList.add('error');
    }
  });
});
```

### **Example 5: Multi-Step Form Progress**
```javascript
const form = Forms.createForm({
  step1_name: '',
  step1_email: '',
  step2_address: '',
  step2_city: '',
  step3_payment: ''
}, {
  validators: {
    step1_name: Validators.required(),
    step1_email: Validators.email(),
    step2_address: Validators.required(),
    step2_city: Validators.required(),
    step3_payment: Validators.required()
  }
});

function canProceedFromStep(stepNumber) {
  // Validate current step fields
  const stepPrefix = `step${stepNumber}_`;
  const stepFields = Object.keys(form.values).filter(f => f.startsWith(stepPrefix));

  stepFields.forEach(field => form.validateField(field));

  // Check if any step fields have errors
  const stepErrors = form.errorFields.filter(f => f.startsWith(stepPrefix));

  if (stepErrors.length > 0) {
    alert(`Please fix ${stepErrors.length} error(s) before continuing`);
    return false;
  }

  return true;
}
```

### **Example 6: Real-time Validation Status**
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
  const status = document.querySelector('#validation-status');

  if (!form.isDirty) {
    status.textContent = '○ No fields touched';
    status.className = 'neutral';
  } else if (form.errorFields.length === 0) {
    status.textContent = '✓ All valid';
    status.className = 'success';
  } else {
    status.textContent = `✗ ${form.errorFields.length} error(s): ${form.errorFields.join(', ')}`;
    status.className = 'error';
  }
});
```

### **Example 7: Conditional Submit Button**
```javascript
const form = Forms.createForm({
  title: '',
  content: ''
}, {
  validators: {
    title: Validators.required('Title required'),
    content: Validators.minLength(10, 'Content too short')
  }
});

effect(() => {
  const submitBtn = document.querySelector('#submit');
  const errorCount = form.errorFields.length;

  if (errorCount === 0 && form.isDirty) {
    submitBtn.disabled = false;
    submitBtn.textContent = 'Submit';
  } else if (errorCount > 0) {
    submitBtn.disabled = true;
    submitBtn.textContent = `Fix ${errorCount} error(s)`;
  } else {
    submitBtn.disabled = true;
    submitBtn.textContent = 'No changes';
  }
});
```

### **Example 8: Error Tooltip**
```javascript
const form = Forms.createForm({
  username: '',
  email: ''
}, {
  validators: {
    username: Validators.required('Username is required'),
    email: Validators.email('Invalid email format')
  }
});

effect(() => {
  form.errorFields.forEach(field => {
    const input = document.querySelector(`input[name="${field}"]`);
    const error = form.errors[field];

    if (input && error) {
      input.setAttribute('title', error);
      input.setAttribute('aria-invalid', 'true');
      input.setAttribute('aria-describedby', `${field}-error`);
    }
  });

  // Clear attributes for valid fields
  Object.keys(form.values).forEach(field => {
    if (!form.errorFields.includes(field)) {
      const input = document.querySelector(`input[name="${field}"]`);
      if (input) {
        input.removeAttribute('title');
        input.setAttribute('aria-invalid', 'false');
      }
    }
  });
});
```

### **Example 9: Validation Progress Bar**
```javascript
const form = Forms.createForm({
  field1: '',
  field2: '',
  field3: '',
  field4: ''
}, {
  validators: {
    field1: Validators.required(),
    field2: Validators.required(),
    field3: Validators.required(),
    field4: Validators.required()
  }
});

effect(() => {
  const totalFields = Object.keys(form.values).length;
  const validFields = totalFields - form.errorFields.length;
  const percentage = (validFields / totalFields) * 100;

  const progressBar = document.querySelector('#progress-bar');
  progressBar.style.width = `${percentage}%`;
  progressBar.textContent = `${validFields}/${totalFields} valid`;

  if (form.errorFields.length === 0) {
    progressBar.className = 'complete';
  } else {
    progressBar.className = 'in-progress';
  }
});
```

### **Example 10: Analytics Tracking**
```javascript
const form = Forms.createForm({
  name: '',
  email: '',
  phone: '',
  message: ''
}, {
  validators: {
    name: Validators.required(),
    email: Validators.email(),
    phone: Validators.pattern(/^\d{10}$/),
    message: Validators.minLength(20)
  }
});

effect(() => {
  const analytics = {
    errorCount: form.errorFields.length,
    errorFields: form.errorFields,
    mostCommonError: getMostCommonError(form.errorFields),
    validationRate: ((4 - form.errorFields.length) / 4) * 100
  };

  console.log('Validation analytics:', analytics);
  sendAnalytics('form_validation', analytics);
});
```

---

## **Common Patterns**

### **Pattern 1: Count Errors**
```javascript
const errorCount = form.errorFields.length;
```

### **Pattern 2: Check if Specific Field Has Error**
```javascript
const emailHasError = form.errorFields.includes('email');
```

### **Pattern 3: Get Error Messages**
```javascript
const errorMessages = form.errorFields.map(field => form.errors[field]);
```

### **Pattern 4: Iterate Error Fields**
```javascript
form.errorFields.forEach(field => {
  console.log(`${field}: ${form.errors[field]}`);
});
```

### **Pattern 5: Focus First Error**
```javascript
if (form.errorFields.length > 0) {
  const firstError = form.errorFields[0];
  document.querySelector(`input[name="${firstError}"]`).focus();
}
```

---

## **Comparison Table**

| Property | Type | What it includes |
|----------|------|-----------------|
| `form.errors` | Object | All error keys (including `null` values) |
| `form.errorFields` | Array | Only fields with truthy error values |
| `form.hasErrors` | Boolean | `errorFields.length > 0` |
| `form.isValid` | Boolean | `errorFields.length === 0` (approximately) |

```javascript
// Example showing the difference
form.errors = {
  email: 'Invalid email',
  name: null,
  phone: ''
};

Object.keys(form.errors);  // ['email', 'name', 'phone']
form.errorFields;          // ['email'] (only truthy errors)
form.hasErrors;            // true
form.isValid;              // false
```

---

## **Key Takeaways**

1. **Reactive Computed Property**: `errorFields` automatically updates when errors change
2. **Array of Strings**: Returns field names with truthy errors only
3. **Filtered**: Excludes fields with `null`, `undefined`, `''`, or `false` errors
4. **Empty When Valid**: Returns `[]` when no fields have truthy errors
5. **Order Preserved**: Array order matches object key order
6. **Common Use**: Error lists, focus management, validation summaries, progress tracking
7. **Pairs with `errors`**: Use together to show which fields failed and why
8. **Different from `Object.keys(errors)`**: Filters out falsy error values

---

## **Summary**

`form.errorFields` is a reactive computed property that returns an array of field names that currently have validation errors (truthy error values). It automatically updates when the form's error state changes, providing a convenient way to identify which fields failed validation, show error summaries, highlight problematic fields, and track validation progress. Use it to create better user experiences by clearly showing which fields need attention and guiding users to fix validation issues efficiently.
