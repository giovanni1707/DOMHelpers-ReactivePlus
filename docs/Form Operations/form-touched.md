# `form.touched` - Form Touched Fields State Property

**Quick Start (30 seconds)**
```javascript
const form = Forms.createForm({ name: '', email: '', phone: '' });

console.log(form.touched); // {} (no fields touched)

form.setValue('name', 'John');
console.log(form.touched); // { name: true }

form.setValue('email', 'john@example.com');
console.log(form.touched); // { name: true, email: true }

form.setTouched('phone', true);
console.log(form.touched); // { name: true, email: true, phone: true }

form.reset();
console.log(form.touched); // {} (reset clears touched)
```

---

## **What is `form.touched`?**

`form.touched` is a **reactive state property** that tracks which form fields have been interacted with (touched) by the user. It's a plain object where each key is a field name and each value is typically `true` (touched) or omitted/`false` (untouched).

**Key characteristics:**
- **Reactive**: Changes trigger effects and re-renders
- **Touch Tracking**: Records which fields users have interacted with
- **Plain Object**: Standard JavaScript object with field flags
- **Mutable**: Can be set via methods or direct mutation
- **Initially Empty**: Starts as `{}` until fields are touched

---

## **Syntax**

```javascript
// Read touched state
const allTouched = form.touched;
const isEmailTouched = form.touched.email;

// Write touched state (direct)
form.touched.email = true;
form.touched.name = false;

// Preferred: Use methods
form.setTouched('email', true);
form.setTouchedFields({ email: true, name: true });
form.touchAll(); // Mark all fields as touched
```

### **Structure**
```javascript
form.touched = {
  fieldName1: true,
  fieldName2: true,
  ...
}
```

---

## **Why does `form.touched` exist?**

### ❌ **The Problem**
```javascript
// Without form.touched, you'd manually track interaction
let emailTouched = false;
let nameTouched = false;

function handleBlur(field) {
  if (field === 'email') emailTouched = true;
  if (field === 'name') nameTouched = true;
}

// Tedious to manage
function shouldShowError(field) {
  if (field === 'email') return emailTouched && hasError;
  if (field === 'name') return nameTouched && hasError;
}

// No centralized tracking
```

### ✅ **The Solution**
```javascript
// Centralized reactive touched state
const form = Forms.createForm({ email: '', name: '' });

// All touched state in one place
console.log(form.touched);

// Easy to check if field was touched
if (form.touched.email) {
  showValidation();
}

// Reactive touched tracking
effect(() => {
  if (form.touched.email && form.errors.email) {
    showError('email', form.errors.email);
  }
});
```

---

## **Mental Model**

Think of `form.touched` as a **footprint tracker** that records where users have stepped:

```
┌─────────────────────────────────────────────┐
│         FORM TOUCHED OBJECT                 │
├─────────────────────────────────────────────┤
│                                             │
│  form.touched = {                           │
│    name: true,         ← User edited name   │
│    email: true,        ← User edited email  │
│    // phone: not set  ← User hasn't touched │
│  }                                          │
│                                             │
│  ↓ Reactive - triggers effects              │
│  ↓ true = touched, absent = untouched       │
│  ↓ Updated by setValue() or manually        │
│                                             │
└─────────────────────────────────────────────┘
```

**The user interaction flow:**
```
User Focuses → User Types → Field Blurs → Touched
     ↓             ↓            ↓            ↓
   Input        onChange      onBlur    form.touched.field = true
```

---

## **How does it work?**

When you create a form, the `touched` property is initialized and made reactive:

```javascript
const formObj = {
  values: { ...initialValues },
  errors: {},
  touched: {},  // Empty initially
  isSubmitting: false
};

const form = createState(formObj); // Made reactive
```

When a field is modified via `setValue()`, it's marked as touched:
```javascript
form.setValue('email', 'test@example.com');
// Internally: this.touched.email = true
```

---

## **Examples**

### **Example 1: Show Errors Only for Touched Fields**
```javascript
const form = Forms.createForm({ email: '', password: '' }, {
  validators: {
    email: Validators.email(),
    password: Validators.minLength(8)
  }
});

effect(() => {
  const emailErrorEl = querySelector('#email-error');
  const passwordErrorEl = querySelector('#password-error');

  // Only show error if field was touched
  emailErrorEl.textContent = form.touched.email && form.errors.email
    ? form.errors.email
    : '';

  passwordErrorEl.textContent = form.touched.password && form.errors.password
    ? form.errors.password
    : '';
});
```

### **Example 2: Field Blur Handler**
```javascript
const form = Forms.createForm({ name: '', email: '' });

querySelector('#name').addEventListener('blur', () => {
  form.setTouched('name', true);
  form.validateField('name');
});

querySelector('#email').addEventListener('blur', () => {
  form.setTouched('email', true);
  form.validateField('email');
});

// Or use the built-in handler
const handleBlur = form.handleBlur();
querySelector('#name').addEventListener('blur', handleBlur);
```

### **Example 3: Mark All Fields as Touched on Submit**
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

function handleSubmit(e) {
  e.preventDefault();

  // Mark all fields as touched to show all errors
  form.touchAll();

  // Validate all fields
  form.validate();

  if (form.isValid) {
    form.submit();
  }
  // Errors will now be visible for all fields
}
```

### **Example 4: Touched Field Counter**
```javascript
const form = Forms.createForm({
  field1: '',
  field2: '',
  field3: '',
  field4: ''
});

effect(() => {
  const counter = querySelector('#touched-counter');
  const touchedCount = Object.keys(form.touched).length;
  const totalFields = Object.keys(form.values).length;

  counter.textContent = `${touchedCount} / ${totalFields} fields touched`;
});
```

### **Example 5: Conditional Validation Display**
```javascript
const form = Forms.createForm({ username: '', email: '' }, {
  validators: {
    username: Validators.required('Username is required'),
    email: Validators.email('Invalid email')
  }
});

effect(() => {
  Object.keys(form.values).forEach(field => {
    const input = querySelector(`input[name="${field}"]`);
    const errorEl = querySelector(`#${field}-error`);

    // Only show validation state if touched
    if (form.touched[field]) {
      if (form.errors[field]) {
        input.classList.add('error');
        input.classList.remove('valid');
        errorEl.textContent = form.errors[field];
      } else {
        input.classList.add('valid');
        input.classList.remove('error');
        errorEl.textContent = '';
      }
    } else {
      input.classList.remove('error', 'valid');
      errorEl.textContent = '';
    }
  });
});
```

### **Example 6: Reset Touched State After Save**
```javascript
const form = Forms.createForm({ title: '', content: '' });

async function handleSave() {
  if (form.isDirty) {
    await fetch('/api/save', {
      method: 'POST',
      body: JSON.stringify(form.values)
    });

    // Clear touched state after successful save
    form.setTouchedFields({});
    // Now form.isDirty will be false
  }
}
```

### **Example 7: Partial Form Validation**
```javascript
const form = Forms.createForm({
  step1: '',
  step2: '',
  step3: ''
}, {
  validators: {
    step1: Validators.required(),
    step2: Validators.required(),
    step3: Validators.required()
  }
});

// Only validate fields the user has touched
function validateTouchedFields() {
  Object.keys(form.touched).forEach(field => {
    form.validateField(field);
  });
}

// Call on blur, not on every keystroke
querySelectorAll('input').forEach(input => {
  input.addEventListener('blur', () => {
    form.setTouched(input.name, true);
    validateTouchedFields();
  });
});
```

### **Example 8: Highlight Touched Fields**
```javascript
const form = Forms.createForm({
  name: '',
  email: '',
  phone: ''
});

effect(() => {
  // Add visual indicator to touched fields
  Object.keys(form.touched).forEach(field => {
    const input = querySelector(`input[name="${field}"]`);
    if (input) {
      input.classList.add('touched');
    }
  });
});
```

### **Example 9: Warning Before Navigation**
```javascript
const form = Forms.createForm({ content: '' });

// Warn only if user has made changes
window.addEventListener('beforeunload', (e) => {
  if (Object.keys(form.touched).length > 0) {
    e.preventDefault();
    e.returnValue = 'You have unsaved changes!';
    return 'You have unsaved changes!';
  }
});

// Or use form.isDirty (which checks touched.length > 0)
window.addEventListener('beforeunload', (e) => {
  if (form.isDirty) {
    e.preventDefault();
    e.returnValue = 'You have unsaved changes!';
  }
});
```

### **Example 10: Auto-save Touched Fields**
```javascript
const form = Forms.createForm({
  field1: '',
  field2: '',
  field3: ''
});

let saveTimeout;

effect(() => {
  const touchedFields = Object.keys(form.touched);

  if (touchedFields.length > 0) {
    clearTimeout(saveTimeout);

    saveTimeout = setTimeout(() => {
      const dataToSave = {};

      touchedFields.forEach(field => {
        dataToSave[field] = form.values[field];
      });

      console.log('Auto-saving touched fields:', dataToSave);
      saveToServer(dataToSave);

      // Clear touched after save
      form.setTouchedFields({});
    }, 2000);
  }
});
```

---

## **Common Patterns**

### **Pattern 1: Check if Field is Touched**
```javascript
if (form.touched.email) {
  // Field was touched
}
```

### **Pattern 2: Show Error Only if Touched**
```javascript
const shouldShowError = form.touched.email && form.errors.email;
```

### **Pattern 3: Mark Field as Touched**
```javascript
form.touched.email = true;
// Or
form.setTouched('email', true);
```

### **Pattern 4: Touch All Fields**
```javascript
form.touchAll();
// Or manually
Object.keys(form.values).forEach(field => {
  form.touched[field] = true;
});
```

### **Pattern 5: Clear All Touched**
```javascript
form.setTouchedFields({});
// Or manually
form.touched = {};
```

---

## **Comparison with Related Properties**

| Property | Type | What it represents |
|----------|------|-------------------|
| `form.touched` | Object | `{ fieldName: true, ... }` |
| `form.touchedFields` | Array | `['fieldName', ...]` |
| `form.isDirty` | Boolean | `touchedFields.length > 0` |
| `form.values` | Object | Current field values |

```javascript
// Example relationships
form.setValue('email', 'test');
form.setValue('name', 'John');

form.touched;          // { email: true, name: true }
form.touchedFields;    // ['email', 'name']
form.isDirty;          // true
form.values;           // { email: 'test', name: 'John', ... }
```

---

## **When to Show Validation Errors**

| Strategy | When to show | Use case |
|----------|--------------|----------|
| **On touch** | `form.touched[field] && form.errors[field]` | Most user-friendly |
| **Always** | `form.errors[field]` | Aggressive feedback |
| **On submit** | After `form.touchAll()` | Delayed feedback |
| **On dirty + touch** | `form.touched[field] && form.isDirty` | Hybrid approach |

```javascript
// Most common: Show error only if touched
effect(() => {
  if (form.touched.email && form.errors.email) {
    showError(form.errors.email);
  }
});
```

---

## **Key Takeaways**

1. **Reactive Object**: Changes to `form.touched` trigger effects
2. **Touch Tracking**: Records which fields users have interacted with
3. **Direct Access**: Can read state directly: `form.touched.fieldName`
4. **Set by setValue()**: Automatically set when using `setValue()` method
5. **Manual Control**: Can set via `setTouched()` or direct mutation
6. **Initially Empty**: Starts as `{}`, populated as user interacts
7. **Reset Clears**: `form.reset()` clears touched state
8. **Common Use**: Conditional error display, dirty state, auto-save logic

---

## **Summary**

`form.touched` is a reactive state property that tracks which form fields have been interacted with (touched) by the user as a plain JavaScript object. Each key is a field name with a value of `true` (touched) or omitted (untouched). Changes to this object automatically trigger reactive effects, making it perfect for showing validation errors only for fields users have interacted with, tracking form dirty state, implementing auto-save logic, and providing better UX by avoiding overwhelming users with errors before they've had a chance to interact with fields. Use it to create more user-friendly forms that progressively show feedback.
