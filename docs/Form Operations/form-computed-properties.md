# Form Computed Properties - Complete Reference

**Quick Start (30 seconds)**
```javascript
const form = ReactiveUtils.form({
  values: { email: '', password: '', age: '' },
  validators: {
    email: Forms.v.email(),
    password: Forms.v.minLength(8),
    age: Forms.v.min(18)
  }
});

// Check form validity
console.log(form.isValid); // true (no errors yet)

// Set values and check dirty state
form.setValue('email', 'test');
console.log(form.isDirty); // true (field touched)

// Check for errors
form.validateField('email');
console.log(form.hasErrors); // true (invalid email)
console.log(form.isValid); // false (has errors)

// Get touched and error fields
console.log(form.touchedFields); // ['email']
console.log(form.errorFields); // ['email']
```

---

## **What are Form Computed Properties?**

Form computed properties are **reactive, auto-updating getters** that provide derived information about form state without manual tracking.

**The 5 core computed properties:**
1. **`isValid`** - True if form has no validation errors
2. **`isDirty`** - True if any field has been touched
3. **`hasErrors`** - True if any validation errors exist
4. **`touchedFields`** - Array of field names that have been touched
5. **`errorFields`** - Array of field names with validation errors

**Key characteristics:**
- **Reactive**: Automatically update when form state changes
- **Cached**: Computed once, cached until dependencies change
- **Read-Only**: Cannot be set directly
- **Always Current**: Always reflect latest form state
- **Type Safe**: Predictable return types

---

## **Property Reference**

### **1. `isValid`**

**Type**: Computed boolean
**Returns**: `boolean`
**Reactive**: Yes

Returns `true` if the form has no validation errors, `false` otherwise.

```javascript
const form = ReactiveUtils.form({
  values: { name: '', email: '' },
  validators: {
    name: Forms.v.required(),
    email: Forms.v.email()
  }
});

console.log(form.isValid); // true (no errors initially)

form.setValue('email', 'invalid');
form.validateField('email');
console.log(form.isValid); // false (has error)

form.setValue('email', 'test@example.com');
form.validateField('email');
console.log(form.isValid); // true (error cleared)
```

**Implementation:**
```javascript
form.$computed('isValid', function() {
  const errorKeys = Object.keys(this.errors);
  return errorKeys.length === 0 || errorKeys.every(k => !this.errors[k]);
});
```

**Logic:**
- Returns `true` if no error keys exist
- Returns `true` if all error values are falsy (null, undefined, '')
- Returns `false` if any error has a truthy value

**Use cases:**
- Enable/disable submit button
- Show/hide success message
- Form submission guard
- Validation indicator
- Progress tracking

---

### **2. `isDirty`**

**Type**: Computed boolean
**Returns**: `boolean`
**Reactive**: Yes

Returns `true` if any field has been touched (modified by user), `false` otherwise.

```javascript
const form = ReactiveUtils.form({
  values: { username: '', email: '' }
});

console.log(form.isDirty); // false (pristine)

form.setValue('username', 'john');
console.log(form.isDirty); // true (touched)
console.log(form.touched); // { username: true }
```

**Implementation:**
```javascript
form.$computed('isDirty', function() {
  return Object.keys(this.touched).length > 0;
});
```

**Logic:**
- Returns `true` if any field in `touched` object
- Returns `false` if `touched` object is empty

**Use cases:**
- Warn before navigation
- Show unsaved changes indicator
- Enable save button
- Track user interaction
- Reset functionality

---

### **3. `hasErrors`**

**Type**: Computed boolean
**Returns**: `boolean`
**Reactive**: Yes

Returns `true` if any validation errors exist with truthy values, `false` otherwise.

```javascript
const form = ReactiveUtils.form({
  values: { email: '' },
  validators: {
    email: Forms.v.email()
  }
});

console.log(form.hasErrors); // false

form.setValue('email', 'invalid');
form.validateField('email');
console.log(form.hasErrors); // true
console.log(form.errors); // { email: 'Must be a valid email' }
```

**Implementation:**
```javascript
form.$computed('hasErrors', function() {
  return Object.keys(this.errors).some(k => this.errors[k]);
});
```

**Logic:**
- Returns `true` if any error value is truthy
- Returns `false` if all errors are falsy or no errors exist
- Inverse of `isValid` in most cases

**Use cases:**
- Show error summary
- Display error count
- Conditional styling
- Validation status indicator
- Form blocking

---

### **4. `touchedFields`**

**Type**: Computed array
**Returns**: `string[]`
**Reactive**: Yes

Returns an array of field names that have been touched (modified).

```javascript
const form = ReactiveUtils.form({
  values: { name: '', email: '', age: '' }
});

console.log(form.touchedFields); // []

form.setValue('name', 'John');
console.log(form.touchedFields); // ['name']

form.setValue('email', 'john@example.com');
console.log(form.touchedFields); // ['name', 'email']
```

**Implementation:**
```javascript
form.$computed('touchedFields', function() {
  return Object.keys(this.touched);
});
```

**Logic:**
- Returns array of keys from `touched` object
- Empty array if no fields touched
- Order may vary (object key order)

**Use cases:**
- Display modified fields
- Partial validation
- Change tracking
- Field-level indicators
- Debug information

---

### **5. `errorFields`**

**Type**: Computed array
**Returns**: `string[]`
**Reactive**: Yes

Returns an array of field names that have validation errors.

```javascript
const form = ReactiveUtils.form({
  values: { email: '', password: '', age: '' },
  validators: {
    email: Forms.v.email(),
    password: Forms.v.minLength(8),
    age: Forms.v.min(18)
  }
});

form.validateAll();
console.log(form.errorFields); // ['email', 'password', 'age'] (all invalid)

form.setValue('email', 'test@example.com');
form.validateField('email');
console.log(form.errorFields); // ['password', 'age']
```

**Implementation:**
```javascript
form.$computed('errorFields', function() {
  return Object.keys(this.errors).filter(k => this.errors[k]);
});
```

**Logic:**
- Returns array of error keys with truthy values
- Filters out falsy errors (null, undefined, '')
- Empty array if no errors

**Use cases:**
- Error summary
- Focus first error field
- Error count display
- Field highlighting
- Validation progress

---

## **Examples**

### **Example 1: Submit Button State**
```javascript
const form = ReactiveUtils.form({
  values: { email: '', password: '' },
  validators: {
    email: Forms.v.email(),
    password: Forms.v.minLength(8)
  }
});

ReactiveUtils.effect(() => {
  const submitBtn = document.getElementById('submit');
  submitBtn.disabled = !form.isValid || !form.isDirty;

  if (form.isValid && form.isDirty) {
    submitBtn.textContent = 'Submit';
  } else if (!form.isDirty) {
    submitBtn.textContent = 'No Changes';
  } else {
    submitBtn.textContent = 'Fix Errors';
  }
});
```

### **Example 2: Unsaved Changes Warning**
```javascript
const form = ReactiveUtils.form({
  values: { title: '', content: '' }
});

window.addEventListener('beforeunload', (e) => {
  if (form.isDirty && !form.isSubmitting) {
    e.preventDefault();
    e.returnValue = 'You have unsaved changes. Are you sure you want to leave?';
    return e.returnValue;
  }
});

ReactiveUtils.effect(() => {
  const indicator = document.getElementById('unsaved');
  indicator.style.display = form.isDirty ? 'block' : 'none';
  indicator.textContent = `${form.touchedFields.length} field(s) modified`;
});
```

### **Example 3: Error Summary Display**
```javascript
const form = ReactiveUtils.form({
  values: { name: '', email: '', phone: '' },
  validators: {
    name: Forms.v.required(),
    email: Forms.v.email(),
    phone: Forms.v.pattern(/^\d{3}-\d{3}-\d{4}$/)
  }
});

ReactiveUtils.effect(() => {
  const summary = document.getElementById('error-summary');

  if (!form.hasErrors) {
    summary.innerHTML = '<p class="success">✓ All fields valid</p>';
    return;
  }

  const errorCount = form.errorFields.length;
  const errorList = form.errorFields
    .map(field => `<li>${field}: ${form.errors[field]}</li>`)
    .join('');

  summary.innerHTML = `
    <div class="error">
      <h4>${errorCount} Error${errorCount > 1 ? 's' : ''}</h4>
      <ul>${errorList}</ul>
    </div>
  `;
});
```

### **Example 4: Progress Indicator**
```javascript
const form = ReactiveUtils.form({
  values: { step1: '', step2: '', step3: '', step4: '' },
  validators: {
    step1: Forms.v.required(),
    step2: Forms.v.required(),
    step3: Forms.v.required(),
    step4: Forms.v.required()
  }
});

ReactiveUtils.effect(() => {
  const totalFields = 4;
  const validFields = totalFields - form.errorFields.length;
  const touchedCount = form.touchedFields.length;
  const progress = (validFields / totalFields) * 100;

  console.log(`Progress: ${progress.toFixed(0)}%`);
  console.log(`Touched: ${touchedCount}/${totalFields}`);
  console.log(`Valid: ${validFields}/${totalFields}`);

  document.getElementById('progress-bar').style.width = `${progress}%`;
});
```

### **Example 5: Conditional Validation**
```javascript
const form = ReactiveUtils.form({
  values: {
    country: 'US',
    state: '',
    zipCode: '',
    postalCode: ''
  },
  validators: {
    country: Forms.v.required(),
    state: Forms.v.required(),
    zipCode: (value) => {
      if (form.values.country === 'US') {
        return Forms.v.pattern(/^\d{5}$/)(value);
      }
      return null; // Not required for other countries
    }
  }
});

ReactiveUtils.effect(() => {
  // Only show US-specific fields if country is US
  const isUS = form.values.country === 'US';
  const usFields = querySelectorAll('.us-only');

  usFields.forEach(field => {
    field.style.display = isUS ? 'block' : 'none';
  });

  // Validate after country change
  if (form.touchedFields.includes('country')) {
    form.validateAll();
  }
});
```

### **Example 6: Field-Level Error Display**
```javascript
const form = ReactiveUtils.form({
  values: { email: '', username: '', password: '' },
  validators: {
    email: Forms.v.email(),
    username: Forms.v.minLength(3),
    password: Forms.v.minLength(8)
  }
});

ReactiveUtils.effect(() => {
  ['email', 'username', 'password'].forEach(fieldName => {
    const input = document.getElementById(fieldName);
    const errorDiv = document.getElementById(`${fieldName}-error`);

    const hasError = form.errorFields.includes(fieldName);
    const isTouched = form.touchedFields.includes(fieldName);

    if (hasError && isTouched) {
      input.classList.add('error');
      errorDiv.textContent = form.errors[fieldName];
      errorDiv.style.display = 'block';
    } else {
      input.classList.remove('error');
      errorDiv.style.display = 'none';
    }
  });
});
```

### **Example 7: Auto-Save Indicator**
```javascript
const form = ReactiveUtils.form({
  values: { title: '', content: '' }
});

let saveTimeout;

ReactiveUtils.effect(() => {
  const status = document.getElementById('save-status');

  if (!form.isDirty) {
    status.textContent = '✓ All changes saved';
    status.className = 'saved';
    return;
  }

  if (form.hasErrors) {
    status.textContent = '⚠ Cannot save - fix errors first';
    status.className = 'error';
    return;
  }

  // Auto-save after 2 seconds of inactivity
  clearTimeout(saveTimeout);
  status.textContent = '✎ Unsaved changes...';
  status.className = 'pending';

  saveTimeout = setTimeout(() => {
    form.handleSubmit();
    status.textContent = '💾 Saving...';
    status.className = 'saving';
  }, 2000);
});
```

### **Example 8: Multi-Step Form Validation**
```javascript
const form = ReactiveUtils.form({
  values: {
    // Step 1
    firstName: '',
    lastName: '',
    // Step 2
    email: '',
    phone: '',
    // Step 3
    address: '',
    city: ''
  },
  validators: {
    firstName: Forms.v.required(),
    lastName: Forms.v.required(),
    email: Forms.v.email(),
    phone: Forms.v.required(),
    address: Forms.v.required(),
    city: Forms.v.required()
  }
});

const steps = [
  ['firstName', 'lastName'],
  ['email', 'phone'],
  ['address', 'city']
];

function canProceedToStep(stepIndex) {
  const stepFields = steps[stepIndex - 1];
  const stepErrors = form.errorFields.filter(f => stepFields.includes(f));
  const stepTouched = stepFields.every(f => form.touchedFields.includes(f));

  return stepTouched && stepErrors.length === 0;
}

ReactiveUtils.effect(() => {
  steps.forEach((stepFields, index) => {
    const button = document.getElementById(`next-step-${index + 1}`);
    if (button) {
      button.disabled = !canProceedToStep(index + 1);
    }
  });
});
```

### **Example 9: Validation Badge**
```javascript
const form = ReactiveUtils.form({
  values: { password: '', confirmPassword: '' },
  validators: {
    password: Forms.v.combine([
      Forms.v.minLength(8),
      Forms.v.pattern(/[A-Z]/, 'Must contain uppercase'),
      Forms.v.pattern(/[a-z]/, 'Must contain lowercase'),
      Forms.v.pattern(/[0-9]/, 'Must contain number')
    ]),
    confirmPassword: Forms.v.match('password')
  }
});

ReactiveUtils.effect(() => {
  const badge = document.getElementById('validation-badge');

  if (form.isValid && form.isDirty) {
    badge.textContent = '✓ Valid';
    badge.className = 'badge success';
  } else if (form.hasErrors) {
    badge.textContent = `✗ ${form.errorFields.length} Error${form.errorFields.length > 1 ? 's' : ''}`;
    badge.className = 'badge error';
  } else {
    badge.textContent = '○ Not Started';
    badge.className = 'badge neutral';
  }
});
```

### **Example 10: Focus First Error**
```javascript
const form = ReactiveUtils.form({
  values: { name: '', email: '', age: '' },
  validators: {
    name: Forms.v.required(),
    email: Forms.v.email(),
    age: Forms.v.min(18)
  }
});

function focusFirstError() {
  if (form.errorFields.length === 0) {
    return;
  }

  const firstErrorField = form.errorFields[0];
  const input = document.getElementById(firstErrorField);

  if (input) {
    input.focus();
    input.scrollIntoView({ behavior: 'smooth', block: 'center' });
  }
}

// On submit attempt
document.getElementById('form').addEventListener('submit', (e) => {
  e.preventDefault();
  form.validateAll();

  if (form.hasErrors) {
    focusFirstError();
  } else {
    form.handleSubmit();
  }
});
```

### **Example 11: Real-Time Validation Status**
```javascript
const form = ReactiveUtils.form({
  values: { username: '', email: '', password: '' },
  validators: {
    username: Forms.v.minLength(3),
    email: Forms.v.email(),
    password: Forms.v.minLength(8)
  }
});

ReactiveUtils.effect(() => {
  const status = {
    valid: form.isValid,
    dirty: form.isDirty,
    hasErrors: form.hasErrors,
    touchedCount: form.touchedFields.length,
    errorCount: form.errorFields.length,
    errorFields: form.errorFields,
    touchedFields: form.touchedFields
  };

  console.clear();
  console.log('📋 Form Status:');
  console.log('─'.repeat(40));
  console.log(`Valid: ${status.valid ? '✓' : '✗'}`);
  console.log(`Dirty: ${status.dirty ? '✓' : '✗'}`);
  console.log(`Has Errors: ${status.hasErrors ? 'Yes' : 'No'}`);
  console.log(`Touched: ${status.touchedCount} field(s)`);
  console.log(`Errors: ${status.errorCount} field(s)`);

  if (status.errorFields.length > 0) {
    console.log(`\nError Fields: ${status.errorFields.join(', ')}`);
  }

  if (status.touchedFields.length > 0) {
    console.log(`Touched Fields: ${status.touchedFields.join(', ')}`);
  }
});
```

### **Example 12: Conditional Submit Handler**
```javascript
const form = ReactiveUtils.form({
  values: { email: '', message: '' },
  validators: {
    email: Forms.v.email(),
    message: Forms.v.minLength(10)
  },
  onSubmit: async (values) => {
    console.log('Submitting:', values);
    await fetch('/api/contact', {
      method: 'POST',
      body: JSON.stringify(values)
    });
  }
});

ReactiveUtils.effect(() => {
  const submitBtn = document.getElementById('submit');
  const statusDiv = document.getElementById('status');

  // Can't submit if not valid or not dirty
  submitBtn.disabled = !form.isValid || !form.isDirty;

  // Update status message
  if (!form.isDirty) {
    statusDiv.textContent = 'Fill out the form to submit';
  } else if (form.hasErrors) {
    statusDiv.textContent = `Fix ${form.errorFields.length} error(s) to submit`;
  } else {
    statusDiv.textContent = 'Ready to submit!';
  }
});
```

### **Example 13: Partial Form Reset**
```javascript
const form = ReactiveUtils.form({
  values: { section1: '', section2: '', section3: '' },
  validators: {
    section1: Forms.v.required(),
    section2: Forms.v.required(),
    section3: Forms.v.required()
  }
});

function resetTouchedFields() {
  console.log(`Resetting ${form.touchedFields.length} field(s):`);
  console.log(form.touchedFields);

  form.touchedFields.forEach(field => {
    form.values[field] = '';
    delete form.touched[field];
    delete form.errors[field];
  });

  console.log('Reset complete');
  console.log(`Dirty: ${form.isDirty}`); // false
  console.log(`Has Errors: ${form.hasErrors}`); // false
}

// Use case: "Clear changes" button
document.getElementById('clear-changes').addEventListener('click', () => {
  if (form.isDirty) {
    resetTouchedFields();
  }
});
```

### **Example 14: Error Statistics**
```javascript
const form = ReactiveUtils.form({
  values: {
    field1: '', field2: '', field3: '',
    field4: '', field5: '', field6: ''
  },
  validators: {
    field1: Forms.v.required(),
    field2: Forms.v.required(),
    field3: Forms.v.required(),
    field4: Forms.v.required(),
    field5: Forms.v.required(),
    field6: Forms.v.required()
  }
});

ReactiveUtils.effect(() => {
  const stats = {
    totalFields: 6,
    touchedCount: form.touchedFields.length,
    errorCount: form.errorFields.length,
    validCount: form.touchedFields.length - form.errorFields.length,
    untouchedCount: 6 - form.touchedFields.length,
    completionRate: (form.touchedFields.length / 6) * 100,
    validationRate: form.touchedFields.length > 0
      ? ((form.touchedFields.length - form.errorFields.length) / form.touchedFields.length) * 100
      : 0
  };

  console.log('\n📊 Form Statistics:');
  console.log(`Completion: ${stats.completionRate.toFixed(0)}%`);
  console.log(`Validation: ${stats.validationRate.toFixed(0)}%`);
  console.log(`✓ Valid: ${stats.validCount}`);
  console.log(`✗ Errors: ${stats.errorCount}`);
  console.log(`○ Untouched: ${stats.untouchedCount}`);
});
```

### **Example 15: Dynamic Field Requirements**
```javascript
const form = ReactiveUtils.form({
  values: {
    accountType: 'personal',
    businessName: '',
    taxId: '',
    firstName: '',
    lastName: ''
  },
  validators: {
    accountType: Forms.v.required(),
    firstName: Forms.v.required(),
    lastName: Forms.v.required()
  }
});

ReactiveUtils.effect(() => {
  const isBusiness = form.values.accountType === 'business';

  // Update validators dynamically
  if (isBusiness) {
    form.setValidators({
      businessName: Forms.v.required(),
      taxId: Forms.v.required()
    });
  } else {
    delete form.validators.businessName;
    delete form.validators.taxId;
    form.clearError('businessName');
    form.clearError('taxId');
  }

  // Show/hide fields
  document.getElementById('business-fields').style.display =
    isBusiness ? 'block' : 'none';

  // Re-validate if needed
  if (form.touchedFields.includes('accountType')) {
    form.validateAll();
  }
});
```

### **Example 16: Validation Checklist**
```javascript
const form = ReactiveUtils.form({
  values: {
    email: '',
    password: '',
    terms: false,
    newsletter: false
  },
  validators: {
    email: Forms.v.email(),
    password: Forms.v.minLength(8),
    terms: Forms.v.custom(v => v ? null : 'Must accept terms')
  }
});

ReactiveUtils.effect(() => {
  const checklist = [
    { label: 'Valid email', valid: !form.errorFields.includes('email') && form.touchedFields.includes('email') },
    { label: 'Password (8+ chars)', valid: !form.errorFields.includes('password') && form.touchedFields.includes('password') },
    { label: 'Terms accepted', valid: !form.errorFields.includes('terms') && form.touchedFields.includes('terms') },
    { label: 'All fields complete', valid: form.touchedFields.length >= 3 },
    { label: 'No errors', valid: !form.hasErrors }
  ];

  const checklistHTML = checklist
    .map(item => `
      <li class="${item.valid ? 'complete' : 'incomplete'}">
        ${item.valid ? '✓' : '○'} ${item.label}
      </li>
    `)
    .join('');

  document.getElementById('checklist').innerHTML = `<ul>${checklistHTML}</ul>`;
});
```

### **Example 17: Touched vs Error Comparison**
```javascript
const form = ReactiveUtils.form({
  values: { a: '', b: '', c: '', d: '' },
  validators: {
    a: Forms.v.required(),
    b: Forms.v.required(),
    c: Forms.v.required(),
    d: Forms.v.required()
  }
});

// Touch some fields
form.setValue('a', 'valid');
form.setValue('b', ''); // Invalid
form.setValue('c', 'valid');

// Validate
form.validateAll();

ReactiveUtils.effect(() => {
  console.log('\n📋 Field Status:');
  console.log('─'.repeat(40));
  console.log('Field | Touched | Has Error');
  console.log('─'.repeat(40));

  ['a', 'b', 'c', 'd'].forEach(field => {
    const touched = form.touchedFields.includes(field) ? '✓' : '✗';
    const error = form.errorFields.includes(field) ? '✓' : '✗';
    console.log(`  ${field}   |    ${touched}    |     ${error}`);
  });

  console.log('─'.repeat(40));
  console.log(`Totals | ${form.touchedFields.length}/4 | ${form.errorFields.length}/4`);
});
```

### **Example 18: Smart Save Button**
```javascript
const form = ReactiveUtils.form({
  values: { title: '', description: '' },
  validators: {
    title: Forms.v.required(),
    description: Forms.v.minLength(20)
  }
});

let originalValues = { ...form.values };

ReactiveUtils.effect(() => {
  const saveBtn = document.getElementById('save');
  const resetBtn = document.getElementById('reset');

  // Check if values actually changed from original
  const hasChanges = JSON.stringify(form.values) !== JSON.stringify(originalValues);

  // Enable save only if valid, dirty, and has real changes
  saveBtn.disabled = !form.isValid || !form.isDirty || !hasChanges;

  // Enable reset only if dirty
  resetBtn.disabled = !form.isDirty;

  if (!form.isDirty) {
    saveBtn.textContent = 'Saved';
  } else if (form.hasErrors) {
    saveBtn.textContent = `Fix ${form.errorFields.length} Error(s)`;
  } else if (hasChanges) {
    saveBtn.textContent = 'Save Changes';
  } else {
    saveBtn.textContent = 'No Changes';
  }
});

// After successful save
function onSaveSuccess() {
  originalValues = { ...form.values };
  form.reset(); // This clears touched state
}
```

### **Example 19: Field Completion Meter**
```javascript
const form = ReactiveUtils.form({
  values: {
    step1: '', step2: '', step3: '',
    step4: '', step5: '', step6: ''
  },
  validators: {
    step1: Forms.v.required(),
    step2: Forms.v.required(),
    step3: Forms.v.required(),
    step4: Forms.v.required(),
    step5: Forms.v.required(),
    step6: Forms.v.required()
  }
});

ReactiveUtils.effect(() => {
  const totalFields = 6;
  const touchedCount = form.touchedFields.length;
  const validCount = form.touchedFields.filter(f => !form.errorFields.includes(f)).length;

  const completionPercent = (touchedCount / totalFields) * 100;
  const validPercent = touchedCount > 0 ? (validCount / touchedCount) * 100 : 0;

  document.getElementById('completion-meter').innerHTML = `
    <div class="meter">
      <div class="meter-label">
        Completion: ${touchedCount}/${totalFields} (${completionPercent.toFixed(0)}%)
      </div>
      <div class="meter-bar">
        <div class="meter-fill" style="width: ${completionPercent}%"></div>
      </div>
    </div>
    <div class="meter">
      <div class="meter-label">
        Valid: ${validCount}/${touchedCount} (${validPercent.toFixed(0)}%)
      </div>
      <div class="meter-bar">
        <div class="meter-fill valid" style="width: ${validPercent}%"></div>
      </div>
    </div>
  `;

  if (form.isValid && touchedCount === totalFields) {
    document.getElementById('complete-message').textContent = '🎉 Form Complete!';
  }
});
```

### **Example 20: Validation Timeline**
```javascript
const form = ReactiveUtils.form({
  values: { field1: '', field2: '', field3: '' },
  validators: {
    field1: Forms.v.required(),
    field2: Forms.v.email(),
    field3: Forms.v.minLength(5)
  }
});

const timeline = [];

ReactiveUtils.effect(() => {
  timeline.push({
    timestamp: Date.now(),
    isDirty: form.isDirty,
    isValid: form.isValid,
    hasErrors: form.hasErrors,
    touchedCount: form.touchedFields.length,
    errorCount: form.errorFields.length
  });

  // Keep only last 10 entries
  if (timeline.length > 10) {
    timeline.shift();
  }

  console.log('\n📈 Validation Timeline:');
  console.log('─'.repeat(60));
  timeline.forEach((entry, i) => {
    const time = new Date(entry.timestamp).toLocaleTimeString();
    const status = entry.isValid ? '✓' : '✗';
    console.log(`${i + 1}. ${time} ${status} | Dirty: ${entry.isDirty} | Errors: ${entry.errorCount} | Touched: ${entry.touchedCount}`);
  });
});
```

---

## **Common Patterns**

### **Pattern 1: Submit Guard**
```javascript
if (form.isValid && form.isDirty) {
  form.handleSubmit();
}
```

### **Pattern 2: Unsaved Changes Check**
```javascript
if (form.isDirty) {
  confirm('You have unsaved changes. Continue?');
}
```

### **Pattern 3: Error Count Display**
```javascript
const errorCount = form.errorFields.length;
console.log(`${errorCount} error(s)`);
```

### **Pattern 4: Field Validation Status**
```javascript
const isFieldValid = !form.errorFields.includes(fieldName);
```

### **Pattern 5: Progress Calculation**
```javascript
const progress = (form.touchedFields.length / totalFields) * 100;
```

---

## **Property Comparison Table**

| Property | Type | Returns | Use Case |
|----------|------|---------|----------|
| `isValid` | boolean | true if no errors | Enable submit, show success |
| `isDirty` | boolean | true if touched | Warn on navigate, show save |
| `hasErrors` | boolean | true if errors exist | Show error summary, block submit |
| `touchedFields` | array | Field names touched | Track changes, partial validation |
| `errorFields` | array | Field names with errors | Focus errors, error count |

---

## **Computed Property Relationships**

```javascript
// isValid is inverse of hasErrors (usually)
form.isValid === !form.hasErrors; // True in most cases

// errorFields is subset of touchedFields (usually)
form.errorFields.every(f => form.touchedFields.includes(f)); // May be true

// Relationship examples
form.isValid && form.isDirty;     // Valid and modified
!form.isValid && form.isDirty;    // Invalid and modified
form.hasErrors && form.isDirty;   // Has errors and modified
form.errorFields.length > 0;      // Same as hasErrors
```

---

## **State Matrix**

| isValid | isDirty | hasErrors | Meaning |
|---------|---------|-----------|---------|
| ✓ | ✗ | ✗ | Pristine, no errors |
| ✓ | ✓ | ✗ | Modified, all valid |
| ✗ | ✗ | ✓ | Pristine with errors* |
| ✗ | ✓ | ✓ | Modified with errors |

*Usually doesn't occur unless errors set programmatically

---

## **Reactive Behavior**

All computed properties automatically update when dependencies change:

```javascript
const form = ReactiveUtils.form({
  values: { email: '' },
  validators: { email: Forms.v.email() }
});

ReactiveUtils.effect(() => {
  console.log('Valid:', form.isValid);      // Reacts to errors
  console.log('Dirty:', form.isDirty);      // Reacts to touched
  console.log('Errors:', form.hasErrors);   // Reacts to errors
  console.log('Touched:', form.touchedFields); // Reacts to touched
  console.log('Error fields:', form.errorFields); // Reacts to errors
});

form.setValue('email', 'test'); // All 5 effects may trigger
```

---

## **Key Takeaways**

1. **`isValid`** - True when no validation errors exist
2. **`isDirty`** - True when any field has been touched
3. **`hasErrors`** - True when any errors have truthy values
4. **`touchedFields`** - Array of modified field names
5. **`errorFields`** - Array of field names with errors
6. **Reactive** - All automatically update when dependencies change
7. **Cached** - Computed once, cached until dependencies change
8. **Read-Only** - Cannot be set directly
9. **Type Safe** - Predictable boolean or array returns
10. **Performant** - Minimal recomputation due to caching

---

## **Summary**

Form computed properties provide reactive, auto-updating information about form state through five core properties: `isValid` (no validation errors), `isDirty` (has touched fields), `hasErrors` (has validation errors), `touchedFields` (array of modified field names), and `errorFields` (array of fields with errors). These properties are computed getters that automatically recalculate when their dependencies change, enabling reactive UI updates for submit buttons, error displays, progress indicators, and validation status. The `isValid` and `hasErrors` properties are usually inverse of each other, while `touchedFields` and `errorFields` provide arrays for iteration and counting. All properties are read-only, type-safe, and cached for performance. Use them to build responsive forms with real-time validation feedback, unsaved change warnings, conditional rendering, multi-step wizards, and comprehensive error handling - all without manual state tracking or imperative updates.
