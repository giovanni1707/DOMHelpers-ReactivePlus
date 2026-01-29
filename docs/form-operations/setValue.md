# `form.setValue()` - Set Single Field Value

## Quick Start (30 seconds)

```javascript
// Create form
const loginForm = form({
  username: '',
  password: ''
});

// Set single field value
loginForm.setValue('username', 'alice');

console.log(loginForm.values.username); // 'alice'
console.log(loginForm.touched.username); // true (auto-marked as touched)

// Chainable
loginForm
  .setValue('username', 'bob')
  .setValue('password', 'secret123');

// Auto-validates if validator exists
const signupForm = form(
  { email: '' },
  {
    validators: {
      email: (value) => value.includes('@') ? null : 'Invalid email'
    }
  }
);

signupForm.setValue('email', 'invalid');
console.log(signupForm.errors.email); // 'Invalid email'
```

**That's it.** Set a single form field value, automatically marking it as touched and validating if a validator exists.

---

## What is `form.setValue()`?

`form.setValue()` **sets the value of a single form field, marks it as touched, and automatically validates it if a validator is configured**. It's the primary method for updating individual form fields.

**Syntax:**
```javascript
form.setValue(fieldName, newValue);
```

**Parameters:**
- `field` - String name of the field
- `value` - New value for the field

**Returns:**
- The form itself (for chaining)

**Key Points:**
- Updates `form.values[field]`
- Marks field as **touched** (`form.touched[field] = true`)
- **Auto-validates** if validator exists for field
- **Chainable** (returns form)
- Triggers **reactivity**

---

## Summary

`form.setValue()` **sets a field value, marks it touched, and validates**. Primary method for updating form fields.

```javascript
form.setValue('email', 'user@example.com')
```
