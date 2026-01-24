# `form.clearError()` - Clear Single Field Error

## Quick Start (30 seconds)

```javascript
// Create form with errors
const form1 = form({
  email: '',
  password: ''
});

form1.setErrors({
  email: 'Invalid email',
  password: 'Too short'
});

console.log(form1.errors);
// { email: 'Invalid email', password: 'Too short' }

// Clear single error
form1.clearError('email');

console.log(form1.errors);
// { password: 'Too short' }

// Chainable
form1
  .clearError('password')
  .setError('email', 'New error')
  .clearError('email');

console.log(form1.errors); // {}
```

**That's it.** Remove an error from a specific form field.

---

## What is `form.clearError()`?

`form.clearError()` **removes the error message for a specific form field** by deleting it from `form.errors`.

**Syntax:**
```javascript
form.clearError('fieldName');
```

**Parameters:**
- `field` - String name of the field

**Returns:**
- The form itself (for chaining)

**Key Points:**
- Deletes `form.errors[field]`
- Safe to call even if **no error exists**
- **Chainable** (returns form)
- Triggers **reactivity**
- Alternative to `setError(field, null)`

---

## Summary

`form.clearError()` **removes a field's error**. Chainable, safe to call anytime.

```javascript
form.clearError('email')
// Same as: form.setError('email', null)
```
