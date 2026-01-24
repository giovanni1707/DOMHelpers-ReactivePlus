# `form.isDirty` - Form Dirty State Computed Property

**Quick Start (30 seconds)**
```javascript
const form = Forms.createForm({ name: '', email: '' });

console.log(form.isDirty); // false (untouched)

form.setValue('name', 'John');
console.log(form.isDirty); // true (field was touched)

form.reset();
console.log(form.isDirty); // false (reset clears touched)
```

---

## **What is `form.isDirty`?**

`form.isDirty` is a **reactive computed property** that indicates whether any form field has been touched (modified) by the user. It returns `true` if at least one field has been touched, and `false` if the form is pristine (untouched).

**Key characteristics:**
- **Computed**: Automatically recalculates when touched state changes
- **Reactive**: Triggers effects when accessed in reactive contexts
- **Read-only**: A getter property, not a method
- **Boolean**: Always returns `true` or `false`
- **Touch-based**: Based on the `touched` object, not value changes

---

## **Syntax**

```javascript
form.isDirty
```

### **Returns**
- **Type**: `Boolean`
- **Value**:
  - `true` - At least one field has been touched
  - `false` - No fields have been touched (pristine form)

---

## **Why does `form.isDirty` exist?**

### ❌ **The Problem**
```javascript
// Manual dirty checking is repetitive
const form = Forms.createForm({ name: '', email: '' });

// You'd need to manually check touched state
function isFormDirty() {
  return Object.keys(form.touched).length > 0;
}

// Have to call this every time
if (isFormDirty()) {
  showUnsavedWarning();
}

// Or track it yourself
let formTouched = false;
form.setValue('name', 'John');
formTouched = true; // Manual tracking
```

### ✅ **The Solution**
```javascript
// Computed property tracks it automatically
const form = Forms.createForm({ name: '', email: '' });

// Clean, automatic, always accurate
if (form.isDirty) {
  showUnsavedWarning();
}

// Reactive updates
effect(() => {
  window.onbeforeunload = form.isDirty
    ? () => 'You have unsaved changes!'
    : null;
});
```

---

## **Mental Model**

Think of `form.isDirty` as a **flag that flips** the moment any field is touched:

```
┌─────────────────────────────────────┐
│      FORM LIFECYCLE                 │
├─────────────────────────────────────┤
│ 1. Created                          │ ──→ isDirty = false
│    touched: {}                      │     (pristine)
│                                     │
├─────────────────────────────────────┤
│ 2. User edits field                 │ ──→ isDirty = true
│    touched: { name: true }          │     (dirty)
│                                     │
├─────────────────────────────────────┤
│ 3. User edits more fields           │ ──→ isDirty = true
│    touched: { name: true,           │     (still dirty)
│               email: true }         │
│                                     │
├─────────────────────────────────────┤
│ 4. Form reset                       │ ──→ isDirty = false
│    touched: {}                      │     (back to pristine)
└─────────────────────────────────────┘
```

**The formula:**
```
isDirty = Object.keys(touched).length > 0
```

---

## **How does it work?**

The `isDirty` property is implemented as a computed property:

```javascript
form.$computed('isDirty', function() {
  return Object.keys(this.touched).length > 0;
});
```

**The algorithm:**
1. Get all keys from the `touched` object
2. Check if the length is greater than 0
3. Return `true` if any fields are touched, `false` otherwise
4. Automatically recalculates whenever `this.touched` changes

---

## **Examples**

### **Example 1: Unsaved Changes Warning**
```javascript
const form = Forms.createForm({ title: '', content: '' });

// Warn before leaving page
effect(() => {
  window.onbeforeunload = form.isDirty
    ? (e) => {
        e.preventDefault();
        e.returnValue = 'You have unsaved changes!';
        return 'You have unsaved changes!';
      }
    : null;
});

// Form is clean → no warning
// User types → isDirty becomes true → warning appears
// User saves → reset() → warning removed
```

### **Example 2: Save Button Visibility**
```javascript
const form = Forms.createForm({ name: '', bio: '' });

const saveButton = document.querySelector('#save');

effect(() => {
  saveButton.style.display = form.isDirty ? 'block' : 'none';
});

// Button hidden initially
// Appears when user makes changes
// Hides after successful save (when form.reset() is called)
```

### **Example 3: Reset Confirmation**
```javascript
const form = Forms.createForm({ email: '', password: '' });

function handleReset() {
  if (form.isDirty) {
    if (confirm('Discard your changes?')) {
      form.reset();
    }
  } else {
    form.reset(); // No changes, just reset
  }
}
```

### **Example 4: Form State Indicator**
```javascript
const form = Forms.createForm({ username: '', email: '' });

effect(() => {
  const indicator = document.querySelector('#form-state');

  if (!form.isDirty) {
    indicator.textContent = '○ Pristine';
    indicator.className = 'state-pristine';
  } else if (form.isDirty && form.isValid) {
    indicator.textContent = '✓ Ready to save';
    indicator.className = 'state-ready';
  } else {
    indicator.textContent = '✗ Has changes with errors';
    indicator.className = 'state-invalid';
  }
});
```

### **Example 5: Auto-save Only When Dirty**
```javascript
const form = Forms.createForm({ notes: '' });

let autoSaveTimer;

effect(() => {
  if (form.isDirty) {
    clearTimeout(autoSaveTimer);
    autoSaveTimer = setTimeout(() => {
      saveToLocalStorage(form.values);
      form.setTouchedFields({}); // Mark as saved
    }, 2000);
  }
});
```

### **Example 6: Submit Button State**
```javascript
const form = Forms.createForm({ name: '', email: '' }, {
  validators: {
    name: Validators.required(),
    email: Validators.email()
  }
});

effect(() => {
  const submitBtn = document.querySelector('#submit');

  // Enable only if dirty AND valid
  submitBtn.disabled = !form.isDirty || !form.isValid;

  // Update text based on state
  if (!form.isDirty) {
    submitBtn.textContent = 'No changes';
  } else if (!form.isValid) {
    submitBtn.textContent = 'Fix errors';
  } else {
    submitBtn.textContent = 'Save changes';
  }
});
```

### **Example 7: Navigation Guard**
```javascript
const form = Forms.createForm({ title: '', content: '' });

// Single-page app navigation guard
router.beforeNavigate((to, from, next) => {
  if (form.isDirty) {
    const answer = confirm('Leave without saving?');
    if (answer) {
      form.reset(); // Clean up
      next();
    } else {
      next(false); // Cancel navigation
    }
  } else {
    next(); // Allow navigation
  }
});
```

### **Example 8: Different Buttons for Different States**
```javascript
const form = Forms.createForm({ post: '' });

effect(() => {
  const publishBtn = document.querySelector('#publish');
  const draftBtn = document.querySelector('#draft');
  const discardBtn = document.querySelector('#discard');

  if (!form.isDirty) {
    publishBtn.hidden = true;
    draftBtn.hidden = true;
    discardBtn.hidden = true;
  } else {
    publishBtn.hidden = false;
    draftBtn.hidden = false;
    discardBtn.hidden = false;
  }
});
```

### **Example 9: Dirty Fields Count**
```javascript
const form = Forms.createForm({ name: '', email: '', phone: '' });

effect(() => {
  const count = form.touchedFields.length;
  const status = document.querySelector('#dirty-status');

  if (!form.isDirty) {
    status.textContent = 'No changes made';
  } else {
    status.textContent = `${count} field(s) modified`;
  }
});
```

### **Example 10: Conditional Validation**
```javascript
const form = Forms.createForm({ search: '', filters: '' });

// Only validate when user has made changes
function handleSearch() {
  if (form.isDirty) {
    form.validate(); // Run validators

    if (form.isValid) {
      performSearch(form.values);
    }
  } else {
    // Use default search
    performSearch({ search: '', filters: '' });
  }
}
```

---

## **Common Patterns**

### **Pattern 1: Unsaved Changes Warning**
```javascript
effect(() => {
  window.onbeforeunload = form.isDirty ? () => 'Unsaved changes!' : null;
});
```

### **Pattern 2: Conditional Save Button**
```javascript
effect(() => {
  saveButton.disabled = !form.isDirty;
});
```

### **Pattern 3: Dirty + Valid Combo**
```javascript
effect(() => {
  submitBtn.disabled = !(form.isDirty && form.isValid);
});
```

### **Pattern 4: Reset with Confirmation**
```javascript
function resetForm() {
  if (!form.isDirty || confirm('Discard changes?')) {
    form.reset();
  }
}
```

---

## **Comparison with Related Properties**

| Property | What it checks | Example when `true` |
|----------|----------------|---------------------|
| `isDirty` | Any field touched | User edited any field |
| `isValid` | No validation errors | All validators pass |
| `hasErrors` | Any errors exist | At least one validator failed |
| `touchedFields` | Array of touched field names | `['email', 'name']` |

```javascript
// Example states
form.touched = { email: true };
form.errors = {};

form.isDirty;         // true (field was touched)
form.isValid;         // true (no errors)
form.hasErrors;       // false (no errors)
form.touchedFields;   // ['email']
```

---

## **Key Takeaways**

1. **Reactive Computed Property**: `isDirty` automatically updates when touched state changes
2. **Boolean Result**: Always returns `true` or `false`
3. **Touch-Based**: Tracks if fields were interacted with, not if values changed
4. **Not Value Comparison**: Setting a field back to original value still leaves it dirty
5. **Reset Clears**: `form.reset()` sets `isDirty` back to `false`
6. **Common Use**: Unsaved changes warnings, conditional save buttons, navigation guards
7. **Works with Effects**: Triggers re-runs when touch state changes
8. **Pristine Detection**: `!form.isDirty` means the form is pristine (untouched)

---

## **Summary**

`form.isDirty` is a reactive computed property that returns `true` when any form field has been touched (modified), and `false` when the form is pristine (untouched). It automatically updates when the form's touched state changes, making it ideal for showing unsaved changes warnings, controlling save button visibility, implementing navigation guards, and distinguishing between new and modified forms. Use it to provide better UX by warning users before they lose their work.
