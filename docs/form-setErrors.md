# `form.setErrors()` - Set Multiple Field Errors

## Quick Start (30 seconds)

```javascript
// Create form
const signupForm = form({
  username: '',
  email: '',
  password: ''
});

// Set multiple errors at once
signupForm.setErrors({
  username: 'Username is required',
  email: 'Invalid email format',
  password: 'Password too short'
});

console.log(signupForm.errors);
// {
//   username: 'Username is required',
//   email: 'Invalid email format',
//   password: 'Password too short'
// }

// Clear specific errors by passing null
signupForm.setErrors({
  username: null,
  email: 'Still invalid'
});

console.log(signupForm.errors);
// {
//   email: 'Still invalid',
//   password: 'Password too short'
// }

// Chainable
signupForm
  .setErrors({ email: 'Required', password: 'Required' })
  .clearErrors();
```

**That's it.** Set multiple form field errors at once. Batched for performance.

---

## What is `form.setErrors()`?

`form.setErrors()` **sets multiple form field errors in a single batched operation**. Each field error is set using `setError()`.

**Syntax:**
```javascript
form.setErrors({
  field1: 'Error message 1',
  field2: 'Error message 2',
  field3: null  // Clear this error
});
```

**Parameters:**
- `errors` - Object with field names as keys and error messages (or null) as values

**Returns:**
- The form itself (for chaining)

**Key Points:**
- Calls `setError()` for each field
- **Batched** for performance (uses `batch()`)
- Pass **null** to clear specific errors
- **Chainable** (returns form)
- Triggers **reactivity** once (batched)
- Typically called after custom validation

---

## Summary

`form.setErrors()` **sets multiple field errors at once**. Batched, chainable, clears with null.

```javascript
form.setErrors({
  email: 'Invalid format',
  password: 'Too weak'
})
```
