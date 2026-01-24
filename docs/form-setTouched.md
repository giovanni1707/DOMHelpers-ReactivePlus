# `form.setTouched()` - Mark Field as Touched

## Quick Start (30 seconds)

```javascript
// Create form
const form1 = form({
  email: '',
  password: ''
});

// Mark field as touched
form1.setTouched('email');
console.log(form1.touched.email); // true

// Mark as untouched (pass false)
form1.setTouched('email', false);
console.log(form1.touched.email); // undefined

// Default is true
form1.setTouched('password');
console.log(form1.touched.password); // true

// Chainable
form1
  .setTouched('email')
  .setTouched('password')
  .validate();

// Used with shouldShowError()
form1.setError('email', 'Invalid');
console.log(form1.shouldShowError('email')); // true (touched + has error)
```

**That's it.** Mark a field as touched or untouched. Used to control when to show validation errors.

---

## What is `form.setTouched()`?

`form.setTouched()` **marks a field as touched (or untouched) to track user interaction**. Fields are typically marked touched when the user focuses on them, helping control when validation errors should be displayed.

**Syntax:**
```javascript
form.setTouched('fieldName');           // Mark as touched
form.setTouched('fieldName', true);     // Mark as touched
form.setTouched('fieldName', false);    // Mark as untouched
```

**Parameters:**
- `field` - String name of the field
- `touched` (optional) - Boolean, defaults to `true`

**Returns:**
- The form itself (for chaining)

**Key Points:**
- Sets `form.touched[field] = true` or deletes it
- Used with `shouldShowError()` for UX
- **Chainable** (returns form)
- Triggers **reactivity**
- `setValue()` automatically marks as touched

---

## Summary

`form.setTouched()` **marks a field as touched or untouched**. Controls when validation errors display.

```javascript
form.setTouched('email')         // Touched
form.setTouched('email', false)  // Untouched
```
