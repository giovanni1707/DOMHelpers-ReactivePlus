# `form.setTouchedFields()` - Mark Multiple Fields as Touched

## Quick Start (30 seconds)

```javascript
// Create form
const registrationForm = form({
  username: '',
  email: '',
  password: '',
  confirmPassword: ''
});

// Mark multiple fields as touched
registrationForm.setTouchedFields(['username', 'email', 'password']);

console.log(registrationForm.touched);
// { username: true, email: true, password: true }

// Chainable
registrationForm
  .setTouchedFields(['username', 'email'])
  .validate();

// Use before validation to show all errors
registrationForm
  .setTouchedFields(Object.keys(registrationForm.values))
  .validate();
```

**That's it.** Mark multiple fields as touched in one batched operation.

---

## What is `form.setTouchedFields()`?

`form.setTouchedFields()` **marks multiple form fields as touched in a single batched operation**. Each field is marked using `setTouched()`.

**Syntax:**
```javascript
form.setTouchedFields(['field1', 'field2', 'field3']);
```

**Parameters:**
- `fields` - Array of field name strings

**Returns:**
- The form itself (for chaining)

**Key Points:**
- Calls `setTouched()` for each field
- **Batched** for performance
- All fields marked as **touched** (not untouched)
- **Chainable** (returns form)
- Triggers **reactivity** once
- Useful before validation or submission

---

## Summary

`form.setTouchedFields()` **marks multiple fields as touched**. Batched, chainable.

```javascript
form.setTouchedFields(['email', 'password'])
```
