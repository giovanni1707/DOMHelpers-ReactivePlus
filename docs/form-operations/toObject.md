# `form.toObject()` - Convert to Plain Object

## Quick Start (30 seconds)

```javascript
// Create form
const userForm = form({
  name: 'Alice',
  email: 'alice@example.com'
});

// Set some state
userForm.setValue('name', 'Bob');
userForm.setError('email', 'Invalid format');
userForm.setTouched('name');

// Convert to plain object
const snapshot = userForm.toObject();

console.log(snapshot);
// {
//   values: { name: 'Bob', email: 'alice@example.com' },
//   errors: { email: 'Invalid format' },
//   touched: { name: true },
//   isValid: false,
//   isDirty: true,
//   isSubmitting: false,
//   submitCount: 0
// }

// Use for debugging
console.log('Form state:', JSON.stringify(userForm.toObject(), null, 2));

// Use for persistence
localStorage.setItem('formState', JSON.stringify(userForm.toObject()));

// Compare states
const before = userForm.toObject();
userForm.setValue('email', 'bob@example.com');
const after = userForm.toObject();

console.log('Values changed:', 
  JSON.stringify(before.values) !== JSON.stringify(after.values)
);
```

**That's it.** Convert form to a plain JavaScript object with all state. Useful for debugging, persistence, and serialization.

---

## What is `form.toObject()`?

`form.toObject()` **creates a plain JavaScript object containing a snapshot of the entire form state**, including values, errors, touched fields, and computed properties.

**Syntax:**
```javascript
const plainObject = form.toObject();
```

**Parameters:**
- None

**Returns:**
- Plain object with:
  - `values`: Copy of form values
  - `errors`: Copy of form errors
  - `touched`: Copy of touched fields
  - `isValid`: Current validity state
  - `isDirty`: Current dirty state
  - `isSubmitting`: Current submitting state
  - `submitCount`: Number of submissions

**Key Points:**
- Returns **plain object** (not reactive)
- Creates **shallow copies** of values, errors, touched
- Includes **computed properties** (isValid, isDirty)
- Useful for **debugging**
- Useful for **persistence** (localStorage, etc.)
- Useful for **serialization** (JSON.stringify)
- **Not** for modifying form (use form methods)

---

## Summary

`form.toObject()` **converts form to plain object**. Snapshot of all state for debugging/persistence.

```javascript
const snapshot = form.toObject()
// Plain object with values, errors, touched, etc.
```
