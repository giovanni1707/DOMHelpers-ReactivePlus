# `form.setValues()` - Set Multiple Field Values

## Quick Start (30 seconds)

```javascript
// Create form
const profileForm = form({
  firstName: '',
  lastName: '',
  email: '',
  age: 0
});

// Set multiple fields at once
profileForm.setValues({
  firstName: 'Alice',
  lastName: 'Johnson',
  email: 'alice@example.com',
  age: 28
});

console.log(profileForm.values);
// {
//   firstName: 'Alice',
//   lastName: 'Johnson',
//   email: 'alice@example.com',
//   age: 28
// }

// All fields marked as touched
console.log(profileForm.touched);
// { firstName: true, lastName: true, email: true, age: true }

// Chainable
profileForm
  .setValues({ firstName: 'Bob', lastName: 'Smith' })
  .setValues({ email: 'bob@example.com' });
```

**That's it.** Set multiple form field values at once. All fields are marked as touched and validated.

---

## What is `form.setValues()`?

`form.setValues()` **sets multiple form field values in a single batched operation**. Each field is set using `setValue()`, marking it as touched and validating if validators exist.

**Syntax:**
```javascript
form.setValues({
  field1: value1,
  field2: value2,
  field3: value3
});
```

**Parameters:**
- `values` - Object with field names as keys and new values as values

**Returns:**
- The form itself (for chaining)

**Key Points:**
- Calls `setValue()` for each field
- **Batched** for performance (uses `batch()`)
- All fields marked as **touched**
- **Auto-validates** all fields with validators
- **Chainable** (returns form)
- Triggers **reactivity** once (batched)

---

## Summary

`form.setValues()` **sets multiple field values at once**. Batched, chainable, validates all fields.

```javascript
form.setValues({
  email: 'user@example.com',
  password: 'secret'
})
```
