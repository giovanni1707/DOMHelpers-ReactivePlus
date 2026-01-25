# `form.clearErrors()` - Clear All Field Errors

## Quick Start (30 seconds)

```javascript
// Create form with multiple errors
const registrationForm = form({
  username: '',
  email: '',
  password: ''
});

registrationForm.setErrors({
  username: 'Required',
  email: 'Invalid format',
  password: 'Too weak'
});

console.log(registrationForm.errors);
// {
//   username: 'Required',
//   email: 'Invalid format',
//   password: 'Too weak'
// }

// Clear all errors at once
registrationForm.clearErrors();

console.log(registrationForm.errors); // {}

// Chainable
registrationForm
  .setErrors({ email: 'Error', password: 'Error' })
  .clearErrors()
  .setError('username', 'New error');

console.log(registrationForm.errors);
// { username: 'New error' }
```

**That's it.** Remove all error messages from the form at once.

---

## What is `form.clearErrors()`?

`form.clearErrors()` **removes all error messages from the form** by resetting `form.errors` to an empty object.

**Syntax:**
```javascript
form.clearErrors();
```

**Parameters:**
- None

**Returns:**
- The form itself (for chaining)

**Key Points:**
- Resets `form.errors = {}`
- Removes **all field errors** at once
- **Chainable** (returns form)
- Triggers **reactivity**
- Useful before re-validation or form reset

---

## Summary

`form.clearErrors()` **removes all field errors**. Chainable, resets errors to empty object.

```javascript
form.clearErrors()
// form.errors is now {}
```
