# `form.validateField()` - Validate Single Field

## Quick Start (30 seconds)

```javascript
// Create form with validators
const signupForm = form(
  { email: '', password: '' },
  {
    validators: {
      email: (value) => value.includes('@') ? null : 'Invalid email',
      password: (value) => value.length >= 8 ? null : 'Too short'
    }
  }
);

// Validate single field
const emailValid = signupForm.validateField('email');
console.log(emailValid); // false (empty email)
console.log(signupForm.errors.email); // 'Invalid email'

// Set value and validate
signupForm.setValue('email', 'user@example.com');
const valid = signupForm.validateField('email');
console.log(valid); // true
console.log(signupForm.errors.email); // undefined (no error)

// Validate field without validator (always valid)
const valid2 = signupForm.validateField('username');
console.log(valid2); // true (no validator = always valid)

// Use in event handlers
function handleEmailBlur() {
  signupForm.validateField('email');
}
```

**That's it.** Validate a single field using its configured validator. Returns true if valid, false if invalid.

---

## What is `form.validateField()`?

`form.validateField()` **validates a single form field using its configured validator**. If the validator returns an error, it's set on the field. If it returns null, the error is cleared.

**Syntax:**
```javascript
const isValid = form.validateField('fieldName');
```

**Parameters:**
- `field` - String name of the field to validate

**Returns:**
- `true` if field is valid (or has no validator)
- `false` if field is invalid

**Key Points:**
- Runs the validator for the field
- Sets or clears `form.errors[field]`
- Returns **boolean**
- Fields without validators are **always valid**
- Triggers **reactivity**
- Auto-called by `setValue()` and `handleBlur()`

---

## Summary

`form.validateField()` **validates a single field**. Returns boolean, sets/clears error.

```javascript
const valid = form.validateField('email')
// true if valid, false if invalid
```
