# `form.touchedFields` - Touched Fields Array Computed Property

**Quick Start (30 seconds)**
```javascript
const form = Forms.createForm({ name: '', email: '', phone: '' });

console.log(form.touchedFields); // [] (no fields touched)

form.setValue('name', 'John');
console.log(form.touchedFields); // ['name']

form.setValue('email', 'john@example.com');
console.log(form.touchedFields); // ['name', 'email']

form.reset();
console.log(form.touchedFields); // [] (reset clears touched)
```

---

## **What is `form.touchedFields`?**

`form.touchedFields` is a **reactive computed property** that returns an array of field names that have been touched (modified) by the user. It provides a convenient way to get a list of all fields that have been interacted with.

**Key characteristics:**
- **Computed**: Automatically recalculates when touched state changes
- **Reactive**: Triggers effects when accessed in reactive contexts
- **Read-only**: A getter property, not a method
- **Array**: Returns an array of strings (field names)
- **Derived from `touched`**: Extracts keys from the `form.touched` object

---

## **Syntax**

```javascript
form.touchedFields
```

### **Returns**
- **Type**: `Array<string>`
- **Value**: Array of field names that have been touched
- **Empty Array**: `[]` when no fields have been touched

---

## **Why does `form.touchedFields` exist?**

### ❌ **The Problem**
```javascript
// Manual extraction of touched field names is repetitive
const form = Forms.createForm({ name: '', email: '', phone: '' });

// You'd need to manually extract keys
function getTouchedFields() {
  return Object.keys(form.touched);
}

// Repetitive and error-prone
const touchedCount = Object.keys(form.touched).length;
const touchedList = Object.keys(form.touched).join(', ');

// Have to remember the pattern everywhere
```

### ✅ **The Solution**
```javascript
// Computed property gives you the array directly
const form = Forms.createForm({ name: '', email: '', phone: '' });

// Clean, automatic, always up-to-date
const touchedCount = form.touchedFields.length;
const touchedList = form.touchedFields.join(', ');

// Reactive updates
effect(() => {
  console.log('Touched fields:', form.touchedFields);
});
```

---

## **Mental Model**

Think of `form.touchedFields` as a **tracking list** that grows as users interact with fields:

```
┌─────────────────────────────────────────────┐
│          INTERACTION TIMELINE               │
├─────────────────────────────────────────────┤
│ 1. Form created                             │
│    touched: {}                              │
│    touchedFields: []                        │
│                                             │
├─────────────────────────────────────────────┤
│ 2. User edits name                          │
│    touched: { name: true }                  │
│    touchedFields: ['name']                  │
│                                             │
├─────────────────────────────────────────────┤
│ 3. User edits email                         │
│    touched: { name: true, email: true }     │
│    touchedFields: ['name', 'email']         │
│                                             │
├─────────────────────────────────────────────┤
│ 4. User edits phone                         │
│    touched: { name: true, email: true,      │
│               phone: true }                 │
│    touchedFields: ['name', 'email', 'phone']│
│                                             │
├─────────────────────────────────────────────┤
│ 5. Form reset                               │
│    touched: {}                              │
│    touchedFields: []                        │
└─────────────────────────────────────────────┘
```

**The formula:**
```
touchedFields = Object.keys(touched)
```

---

## **How does it work?**

The `touchedFields` property is implemented as a computed property:

```javascript
form.$computed('touchedFields', function() {
  return Object.keys(this.touched);
});
```

**The algorithm:**
1. Get all keys from the `this.touched` object
2. Return them as an array
3. Automatically recalculates whenever `this.touched` changes

---

## **Examples**

### **Example 1: Progress Counter**
```javascript
const form = Forms.createForm({
  step1: '',
  step2: '',
  step3: '',
  step4: ''
});

const totalFields = 4;

effect(() => {
  const completed = form.touchedFields.length;
  const progress = document.querySelector('#progress');

  progress.textContent = `${completed} / ${totalFields} fields completed`;
  progress.style.width = `${(completed / totalFields) * 100}%`;
});
```

### **Example 2: Show Touched Field List**
```javascript
const form = Forms.createForm({ name: '', email: '', phone: '' });

effect(() => {
  const list = document.querySelector('#touched-list');

  if (form.touchedFields.length === 0) {
    list.innerHTML = '<li>No fields touched yet</li>';
  } else {
    const items = form.touchedFields
      .map(field => `<li>${field}</li>`)
      .join('');
    list.innerHTML = items;
  }
});
```

### **Example 3: Conditional Save Button**
```javascript
const form = Forms.createForm({
  title: '',
  description: '',
  tags: ''
});

const minFieldsRequired = 2;

effect(() => {
  const saveBtn = document.querySelector('#save');

  saveBtn.disabled = form.touchedFields.length < minFieldsRequired;
  saveBtn.title = `At least ${minFieldsRequired} fields must be edited`;
});
```

### **Example 4: Touched Fields Summary**
```javascript
const form = Forms.createForm({
  name: '',
  email: '',
  phone: '',
  address: ''
});

effect(() => {
  const summary = document.querySelector('#summary');
  const count = form.touchedFields.length;

  if (count === 0) {
    summary.textContent = 'No changes made';
  } else if (count === 1) {
    summary.textContent = `1 field modified: ${form.touchedFields[0]}`;
  } else {
    summary.textContent = `${count} fields modified: ${form.touchedFields.join(', ')}`;
  }
});
```

### **Example 5: Partial Form Validation**
```javascript
const form = Forms.createForm({
  email: '',
  name: '',
  phone: ''
}, {
  validators: {
    email: Validators.email(),
    name: Validators.required(),
    phone: Validators.pattern(/^\d{10}$/)
  }
});

// Only validate fields that have been touched
function validateTouchedFields() {
  form.touchedFields.forEach(field => {
    form.validateField(field);
  });
}

// User can submit partial data
function handlePartialSubmit() {
  validateTouchedFields();

  const touchedData = {};
  form.touchedFields.forEach(field => {
    touchedData[field] = form.values[field];
  });

  console.log('Submitting touched fields:', touchedData);
}
```

### **Example 6: Auto-save Touched Fields Only**
```javascript
const form = Forms.createForm({
  field1: '',
  field2: '',
  field3: ''
});

let saveTimeout;

effect(() => {
  if (form.touchedFields.length > 0) {
    clearTimeout(saveTimeout);

    saveTimeout = setTimeout(() => {
      const dataToSave = {};
      form.touchedFields.forEach(field => {
        dataToSave[field] = form.values[field];
      });

      console.log('Auto-saving:', dataToSave);
      saveToServer(dataToSave);

      // Clear touched after save
      form.setTouchedFields({});
    }, 2000);
  }
});
```

### **Example 7: Reset Specific Fields**
```javascript
const form = Forms.createForm({
  name: '',
  email: '',
  bio: ''
});

function resetTouchedFields() {
  const touched = [...form.touchedFields]; // Copy array

  touched.forEach(field => {
    form.resetField(field);
  });

  console.log(`Reset ${touched.length} touched field(s)`);
}
```

### **Example 8: Highlight Touched Fields**
```javascript
const form = Forms.createForm({
  firstName: '',
  lastName: '',
  email: ''
});

effect(() => {
  // Remove all highlights
  document.querySelectorAll('input').forEach(input => {
    input.classList.remove('touched');
  });

  // Add highlights to touched fields
  form.touchedFields.forEach(field => {
    const input = document.querySelector(`input[name="${field}"]`);
    if (input) {
      input.classList.add('touched');
    }
  });
});
```

### **Example 9: Unsaved Changes Count**
```javascript
const form = Forms.createForm({
  setting1: false,
  setting2: '',
  setting3: ''
});

effect(() => {
  const badge = document.querySelector('#unsaved-badge');
  const count = form.touchedFields.length;

  if (count > 0) {
    badge.textContent = count;
    badge.style.display = 'inline-block';
    badge.title = `${count} unsaved change(s): ${form.touchedFields.join(', ')}`;
  } else {
    badge.style.display = 'none';
  }
});
```

### **Example 10: Analytics Tracking**
```javascript
const form = Forms.createForm({
  productName: '',
  category: '',
  price: '',
  description: ''
});

effect(() => {
  const analytics = {
    touchedFieldCount: form.touchedFields.length,
    touchedFields: form.touchedFields,
    completionRate: (form.touchedFields.length / 4) * 100,
    mostRecentField: form.touchedFields[form.touchedFields.length - 1]
  };

  console.log('Form analytics:', analytics);
  sendToAnalytics('form_interaction', analytics);
});
```

---

## **Common Patterns**

### **Pattern 1: Count Touched Fields**
```javascript
const count = form.touchedFields.length;
```

### **Pattern 2: Check if Specific Field Touched**
```javascript
const isEmailTouched = form.touchedFields.includes('email');
```

### **Pattern 3: Iterate Touched Fields**
```javascript
form.touchedFields.forEach(field => {
  console.log(`${field}: ${form.values[field]}`);
});
```

### **Pattern 4: Get Touched Values Only**
```javascript
const touchedValues = {};
form.touchedFields.forEach(field => {
  touchedValues[field] = form.values[field];
});
```

### **Pattern 5: Validate Only Touched**
```javascript
form.touchedFields.forEach(field => {
  form.validateField(field);
});
```

---

## **Comparison Table**

| Property/Method | Type | What it represents |
|----------------|------|-------------------|
| `form.touched` | Object | `{ fieldName: true, ... }` |
| `form.touchedFields` | Array | `['fieldName', ...]` |
| `form.isDirty` | Boolean | `touchedFields.length > 0` |
| `form.errorFields` | Array | `['fieldName', ...]` (fields with errors) |

```javascript
// Example relationships
form.setValue('email', 'test');
form.setValue('name', 'John');

form.touched;           // { email: true, name: true }
form.touchedFields;     // ['email', 'name']
form.isDirty;           // true
form.errorFields;       // Depends on validation
```

---

## **Key Takeaways**

1. **Reactive Computed Property**: `touchedFields` automatically updates when touched state changes
2. **Array of Strings**: Returns field names, not field values
3. **Derived from `touched`**: Equivalent to `Object.keys(form.touched)`
4. **Empty When Pristine**: Returns `[]` when no fields have been touched
5. **Grows as User Interacts**: New fields are added when touched
6. **Reset Clears**: `form.reset()` empties the array
7. **Order Preserved**: Array order matches the order fields were touched
8. **Common Use**: Progress tracking, partial validation, auto-save, analytics

---

## **Summary**

`form.touchedFields` is a reactive computed property that returns an array of field names that have been touched (modified) by the user. It automatically updates when the form's touched state changes, providing a convenient way to track user interaction, implement progressive validation, show completion progress, and handle partial form submissions. Use it to create better user experiences by understanding which fields users have engaged with and focusing validation or feedback on those fields.
