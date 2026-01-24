# `form.getError()` - Get Field Error Message

## Quick Start (30 seconds)

```javascript
// Create form
const loginForm = form({
  username: '',
  password: ''
});

// Set errors
loginForm.setErrors({
  username: 'Username is required',
  password: 'Password must be at least 8 characters'
});

// Get error messages
const usernameError = loginForm.getError('username');
console.log(usernameError); // 'Username is required'

const passwordError = loginForm.getError('password');
console.log(passwordError); // 'Password must be at least 8 characters'

// Returns null for fields without errors
const emailError = loginForm.getError('email');
console.log(emailError); // null

// Display errors in UI
effect(() => {
  const error = loginForm.getError('username');
  const errorElement = document.getElementById('username-error');
  errorElement.textContent = error || '';
  errorElement.style.display = error ? 'block' : 'none';
});
```

**That's it.** Get the error message for a specific form field. Returns null if no error.

---

## What is `form.getError()`?

`form.getError()` **returns the error message for a form field, or null if the field has no error**. It's a convenience method equivalent to `form.errors[field] || null`.

**Syntax:**
```javascript
const errorMessage = form.getError('fieldName');
```

**Parameters:**
- `field` - String name of the field

**Returns:**
- Error message string or `null` if no error

**Key Points:**
- Returns error message or **null**
- Equivalent to `form.errors[field] || null`
- **Reactive** when used in effects
- More semantic than accessing errors directly
- Use with `hasError()` or `shouldShowError()` for UI

---

## Summary

`form.getError()` **returns field's error message or null**. Semantic accessor for error messages.

```javascript
const error = form.getError('email')
// Same as: form.errors.email || null
```
