# `form.reset()` - Reset Form to Initial or New Values

## Quick Start (30 seconds)

```javascript
// Create form
const editForm = form({
  name: 'Alice',
  email: 'alice@example.com',
  age: 28
});

// User makes changes
editForm.setValues({
  name: 'Bob',
  email: 'bob@example.com',
  age: 35
});

editForm.setErrors({ email: 'Invalid format' });
editForm.touchAll();

// Reset to initial values
editForm.reset();

console.log(editForm.values);
// { name: 'Alice', email: 'alice@example.com', age: 28 }

console.log(editForm.errors); // {}
console.log(editForm.touched); // {}
console.log(editForm.isSubmitting); // false

// Reset to new values
editForm.reset({
  name: 'Charlie',
  email: 'charlie@example.com',
  age: 42
});

console.log(editForm.values.name); // 'Charlie'

// Chainable
editForm
  .reset()
  .setValue('name', 'New Name');
```

**That's it.** Reset form to initial values (or new values). Clears errors, touched state, and submission state.

---

## What is `form.reset()`?

`form.reset()` **resets the form to its initial values (or optionally to new values)**. It clears all errors, touched states, and resets the submission flag.

**Syntax:**
```javascript
form.reset();              // Reset to initial values
form.reset(newValues);     // Reset to new values
```

**Parameters:**
- `newValues` (optional) - Object with new initial values (default: original initial values)

**Returns:**
- The form itself (for chaining)

**Key Points:**
- Resets `form.values` to initial (or new) values
- Clears **all errors** (`form.errors = {}`)
- Clears **all touched** (`form.touched = {}`)
- Sets `form.isSubmitting = false`
- **Batched** for performance
- **Chainable** (returns form)
- Triggers **reactivity** once

---

## Summary

`form.reset()` **resets form to initial values**. Clears errors, touched, submission state.

```javascript
form.reset()              // Back to initial
form.reset({ name: '' })  // Reset to new values
```
