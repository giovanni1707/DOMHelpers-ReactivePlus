# `form.validate()` - Validate All Fields

## Quick Start (30 seconds)

```javascript
// Create form with validators
const form1 = form(
  { email: '', password: '', username: '' },
  {
    validators: {
      email: (v) => v.includes('@') ? null : 'Invalid email',
      password: (v) => v.length >= 8 ? null : 'Too short',
      username: (v) => v.length >= 3 ? null : 'Too short'
    }
  }
);

// Validate all fields
const isValid = form1.validate();
console.log(isValid); // false (all fields empty/invalid)

console.log(form1.errors);
// {
//   email: 'Invalid email',
//   password: 'Too short',
//   username: 'Too short'
// }

// Set values and validate
form1.setValues({
  email: 'user@example.com',
  password: 'secret123',
  username: 'alice'
});

const valid = form1.validate();
console.log(valid); // true
console.log(form1.errors); // {}

// Common pattern: validate before submit
function handleSubmit() {
  if (form1.validate()) {
    // All fields valid - submit
  } else {
    // Show errors
  }
}
```

**That's it.** Validate all form fields at once. Returns true if all valid, false if any invalid.

---

## What is `form.validate()`?

`form.validate()` **validates all form fields that have validators configured**. It calls `validateField()` for each validator, returning true only if all fields are valid.

**Syntax:**
```javascript
const isValid = form.validate();
```

**Parameters:**
- None

**Returns:**
- `true` if all fields are valid
- `false` if any field is invalid

**Key Points:**
- Validates **all fields** with validators
- **Batched** for performance
- Sets/clears `form.errors` for all fields
- Returns **boolean** (true = all valid)
- Triggers **reactivity** once
- Auto-called by `submit()`
- Fields without validators are skipped

---

## Summary

`form.validate()` **validates all fields**. Returns true if all valid. Batched.

```javascript
if (form.validate()) {
  // All fields valid
}
```
