# `form.setError()` - Set Field Error

## Quick Start (30 seconds)

```javascript
// Create form
const loginForm = form({
  username: '',
  password: ''
});

// Set error for a field
loginForm.setError('username', 'Username is required');

console.log(loginForm.errors.username); // 'Username is required'
console.log(loginForm.hasError('username')); // true

// Clear error by passing null or empty string
loginForm.setError('username', null);
console.log(loginForm.errors.username); // undefined

loginForm.setError('username', '');
console.log(loginForm.errors.username); // undefined

// Chainable
loginForm
  .setError('username', 'Too short')
  .setError('password', 'Must contain a number');
```

**That's it.** Set an error message for a form field. Pass null/empty string to clear.

---

## What is `form.setError()`?

`form.setError()` **sets or clears an error message for a specific form field**. If an error is provided, it's stored in `form.errors[field]`. If null or empty, the error is removed.

**Syntax:**
```javascript
// Set error
form.setError('fieldName', 'Error message');

// Clear error
form.setError('fieldName', null);
form.setError('fieldName', '');
```

**Parameters:**
- `field` - String name of the field
- `error` - Error message string, or null/empty to clear

**Returns:**
- The form itself (for chaining)

**Key Points:**
- Sets `form.errors[field]` to error message
- Pass **null or empty string** to clear error
- **Chainable** (returns form)
- Triggers **reactivity**
- Typically called by validators automatically

---

## Summary

`form.setError()` **sets or clears a field error**. Pass message to set, null to clear.

```javascript
form.setError('email', 'Invalid email format')
form.setError('email', null) // Clear
```
