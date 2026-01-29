# `form.values` - Form Values State Property

**Quick Start (30 seconds)**
```javascript
const form = Forms.createForm({ name: '', email: '', age: 0 });

console.log(form.values); // { name: '', email: '', age: 0 }

form.setValue('name', 'John');
console.log(form.values); // { name: 'John', email: '', age: 0 }

form.setValues({ email: 'john@example.com', age: 30 });
console.log(form.values); // { name: 'John', email: 'john@example.com', age: 30 }

const formData = form.toObject();
console.log(formData); // Same as form.values
```

---

## **What is `form.values`?**

`form.values` is a **reactive state property** that contains all the current field values in your form. It's a plain object where each key is a field name and each value is the current field value.

**Key characteristics:**
- **Reactive**: Changes trigger effects and re-renders
- **Direct Access**: Read and write values directly
- **Plain Object**: Standard JavaScript object with field values
- **Mutable**: Can be modified directly or via methods
- **Initialized**: Contains initial values when form is created

---

## **Syntax**

```javascript
// Read values
const currentValues = form.values;
const emailValue = form.values.email;

// Write values (direct mutation)
form.values.name = 'John';
form.values.email = 'john@example.com';

// Preferred: Use methods (handles touched state & validation)
form.setValue('name', 'John');
form.setValues({ email: 'john@example.com' });
```

### **Structure**
```javascript
form.values = {
  fieldName1: value1,
  fieldName2: value2,
  ...
}
```

---

## **Why does `form.values` exist?**

### ❌ **The Problem**
```javascript
// Without form.values, you'd need separate state for each field
let nameValue = '';
let emailValue = '';
let ageValue = 0;

// Tedious to manage and pass around
function submit() {
  const data = {
    name: nameValue,
    email: emailValue,
    age: ageValue
  };
  sendToServer(data);
}

// No centralized state
```

### ✅ **The Solution**
```javascript
// Centralized reactive state
const form = Forms.createForm({ name: '', email: '', age: 0 });

// All values in one place
console.log(form.values);

// Easy to submit
function submit() {
  sendToServer(form.values);
}

// Reactive updates
effect(() => {
  console.log('Current values:', form.values);
});
```

---

## **Mental Model**

Think of `form.values` as a **living document** that represents the current state of your form:

```
┌─────────────────────────────────────────────┐
│           FORM VALUES OBJECT                │
├─────────────────────────────────────────────┤
│                                             │
│  form.values = {                            │
│    name: 'John Doe',           ← Field 1    │
│    email: 'john@example.com',  ← Field 2    │
│    age: 30,                    ← Field 3    │
│    newsletter: true            ← Field 4    │
│  }                                          │
│                                             │
│  ↓ Reactive - triggers effects              │
│  ↓ Mutable - can be changed                 │
│  ↓ Serializable - ready for API calls       │
│                                             │
└─────────────────────────────────────────────┘
```

**The lifecycle:**
```
Initial → User Input → Validation → Submission
  ↓          ↓            ↓           ↓
values   values       values      values
updated  updated      checked     sent to API
```

---

## **How does it work?**

When you create a form, the `values` property is initialized and made reactive:

```javascript
const formObj = {
  values: { ...initialValues },  // Copy of initial values
  errors: {},
  touched: {},
  isSubmitting: false
};

const form = createState(formObj); // Made reactive
```

Any changes to `form.values` trigger reactive updates:
```javascript
effect(() => {
  console.log(form.values.email); // Tracks email specifically
});

form.values.email = 'new@example.com'; // Triggers the effect
```

---

## **Examples**

### **Example 1: Reading Current Values**
```javascript
const form = Forms.createForm({
  username: '',
  email: '',
  password: ''
});

form.setValue('username', 'johndoe');
form.setValue('email', 'john@example.com');

// Read all values
console.log(form.values);
// { username: 'johndoe', email: 'john@example.com', password: '' }

// Read specific value
console.log(form.values.username); // 'johndoe'
```

### **Example 2: Reactive Display**
```javascript
const form = Forms.createForm({ name: '', bio: '' });

// Auto-update display when values change
effect(() => {
  const preview = querySelector('#preview');
  preview.innerHTML = `
    <h3>${form.values.name || 'No name'}</h3>
    <p>${form.values.bio || 'No bio'}</p>
  `;
});

// When form changes, preview updates automatically
form.setValue('name', 'Jane');
form.setValue('bio', 'Software developer');
```

### **Example 3: Form Submission**
```javascript
const form = Forms.createForm({
  email: '',
  password: ''
}, {
  validators: {
    email: Validators.email(),
    password: Validators.minLength(8)
  },
  async onSubmit(values) {
    // values === form.values
    const response = await fetch('/api/login', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(values)
    });
    return response.json();
  }
});

// Submit sends form.values
await form.submit();
```

### **Example 4: Character Counter**
```javascript
const form = Forms.createForm({ bio: '' });

const maxChars = 200;

effect(() => {
  const counter = querySelector('#char-counter');
  const remaining = maxChars - form.values.bio.length;

  counter.textContent = `${remaining} characters remaining`;
  counter.className = remaining < 20 ? 'warning' : '';
});
```

### **Example 5: Conditional Fields**
```javascript
const form = Forms.createForm({
  accountType: 'personal',
  companyName: '',
  taxId: ''
});

effect(() => {
  const companyFields = querySelector('#company-fields');

  // Show/hide based on account type
  if (form.values.accountType === 'business') {
    companyFields.style.display = 'block';
  } else {
    companyFields.style.display = 'none';
    // Clear business fields when hidden
    form.values.companyName = '';
    form.values.taxId = '';
  }
});
```

### **Example 6: Auto-save to localStorage**
```javascript
const form = Forms.createForm({
  title: '',
  content: '',
  draft: true
});

let saveTimeout;

effect(() => {
  // Save to localStorage whenever values change
  clearTimeout(saveTimeout);

  saveTimeout = setTimeout(() => {
    localStorage.setItem('draft', JSON.stringify(form.values));
    console.log('Auto-saved:', form.values);
  }, 1000);
});

// Load on startup
const savedDraft = localStorage.getItem('draft');
if (savedDraft) {
  form.setValues(JSON.parse(savedDraft));
}
```

### **Example 7: Computed Summary**
```javascript
const form = Forms.createForm({
  quantity: 0,
  price: 0,
  discount: 0
});

effect(() => {
  const subtotal = form.values.quantity * form.values.price;
  const discountAmount = subtotal * (form.values.discount / 100);
  const total = subtotal - discountAmount;

  const summary = querySelector('#order-summary');
  summary.innerHTML = `
    <div>Subtotal: $${subtotal.toFixed(2)}</div>
    <div>Discount: -$${discountAmount.toFixed(2)}</div>
    <div>Total: $${total.toFixed(2)}</div>
  `;
});
```

### **Example 8: Form Comparison**
```javascript
const form = Forms.createForm({
  name: 'John Doe',
  email: 'john@example.com'
});

// Store original values
const originalValues = { ...form.values };

function hasChanges() {
  return JSON.stringify(form.values) !== JSON.stringify(originalValues);
}

effect(() => {
  const status = querySelector('#save-status');

  if (hasChanges()) {
    status.textContent = 'Unsaved changes';
    status.className = 'warning';
  } else {
    status.textContent = 'All changes saved';
    status.className = 'success';
  }
});
```

### **Example 9: Search Form with Debounce**
```javascript
const form = Forms.createForm({
  query: '',
  category: '',
  minPrice: 0,
  maxPrice: 1000
});

let searchTimeout;

effect(() => {
  // Trigger search when any value changes
  clearTimeout(searchTimeout);

  searchTimeout = setTimeout(() => {
    if (form.values.query.length >= 2) {
      performSearch(form.values);
    }
  }, 300);
});
```

### **Example 10: Multi-Page Form State**
```javascript
const form = Forms.createForm({
  // Page 1
  firstName: '',
  lastName: '',
  email: '',

  // Page 2
  address: '',
  city: '',
  zipCode: '',

  // Page 3
  cardNumber: '',
  expiry: '',
  cvv: ''
});

let currentPage = 1;

function getPageFields(page) {
  const pages = {
    1: ['firstName', 'lastName', 'email'],
    2: ['address', 'city', 'zipCode'],
    3: ['cardNumber', 'expiry', 'cvv']
  };
  return pages[page];
}

function getPageValues(page) {
  const fields = getPageFields(page);
  const pageValues = {};

  fields.forEach(field => {
    pageValues[field] = form.values[field];
  });

  return pageValues;
}

// When moving to next page
function nextPage() {
  console.log(`Page ${currentPage} values:`, getPageValues(currentPage));
  currentPage++;
}
```

---

## **Common Patterns**

### **Pattern 1: Read All Values**
```javascript
const allValues = form.values;
const copy = { ...form.values };
```

### **Pattern 2: Read Specific Value**
```javascript
const email = form.values.email;
```

### **Pattern 3: Direct Write (Not Recommended)**
```javascript
form.values.name = 'John'; // Works, but doesn't mark as touched
```

### **Pattern 4: Preferred Write (Use Methods)**
```javascript
form.setValue('name', 'John'); // Marks as touched, triggers validation
```

### **Pattern 5: Submit Values**
```javascript
async function submit() {
  await fetch('/api/submit', {
    method: 'POST',
    body: JSON.stringify(form.values)
  });
}
```

### **Pattern 6: Clone Values**
```javascript
const snapshot = JSON.parse(JSON.stringify(form.values));
```

### **Pattern 7: Merge Values**
```javascript
form.setValues({ ...form.values, ...newValues });
```

---

## **Comparison with Methods**

| Approach | Updates `touched`? | Triggers Validation? | Use Case |
|----------|-------------------|---------------------|----------|
| `form.values.field = val` | No | No | Quick internal updates |
| `form.setValue(field, val)` | Yes | Yes | User input handling |
| `form.setValues(obj)` | Yes (for all) | Yes | Bulk updates |
| `form.reset()` | Clears touched | Clears errors | Reset to initial |

```javascript
// Direct mutation
form.values.email = 'test@example.com';
console.log(form.touched.email); // undefined (not touched)

// Using method
form.setValue('email', 'test@example.com');
console.log(form.touched.email); // true (marked as touched)
```

---

## **Key Takeaways**

1. **Reactive Object**: Changes to `form.values` trigger effects
2. **Centralized State**: All form field values in one object
3. **Direct Access**: Can read values directly: `form.values.fieldName`
4. **Prefer Methods**: Use `setValue()` instead of direct mutation for proper behavior
5. **Serializable**: Ready to send to APIs via `JSON.stringify()`
6. **Initial Values**: Set when form is created, can be reset to them
7. **Shallow Object**: Top-level fields only (nested objects need special handling)
8. **Common Use**: Form submission, reactive displays, auto-save, comparisons

---

## **Summary**

`form.values` is a reactive state property containing all current field values in your form as a plain JavaScript object. Changes to this object automatically trigger reactive effects, making it perfect for real-time displays, auto-save functionality, and form submission. While you can access and modify values directly, using form methods like `setValue()` and `setValues()` is recommended as they properly handle touched state and validation. Use `form.values` to access the complete form state for submission, comparison, or serialization.
