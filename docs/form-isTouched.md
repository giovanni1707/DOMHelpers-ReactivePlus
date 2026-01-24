# `form.isTouched()` - Check if Field is Touched

## Quick Start (30 seconds)

```javascript
// Create form
const form1 = form({
  email: '',
  password: ''
});

// Check if touched
console.log(form1.isTouched('email')); // false

// Mark as touched
form1.setTouched('email');
console.log(form1.isTouched('email')); // true

// Use in conditionals
if (form1.isTouched('email')) {
  console.log('User has interacted with email field');
}

// Reactive in effects
effect(() => {
  if (form1.isTouched('email') && form1.hasError('email')) {
    console.log('Show error to user');
  }
});

// Alternative: use shouldShowError()
if (form1.shouldShowError('email')) {
  // Show error (combines isTouched + hasError)
}
```

**That's it.** Check if a form field has been touched (interacted with). Returns boolean.

---

## What is `form.isTouched()`?

`form.isTouched()` **returns true if a field has been touched, false otherwise**. It's a convenience method equivalent to `!!form.touched[field]`.

**Syntax:**
```javascript
const touched = form.isTouched('fieldName');
```

**Parameters:**
- `field` - String name of the field

**Returns:**
- `true` if field is touched, `false` otherwise

**Key Points:**
- Equivalent to `!!form.touched[field]`
- Returns **boolean**
- **Reactive** when used in effects
- Works with `hasError()` for UX
- Use `shouldShowError()` for combined check

---

## Summary

`form.isTouched()` **returns true if field is touched**. Boolean check for user interaction.

```javascript
if (form.isTouched('email')) { ... }
// Same as: if (form.touched.email) { ... }
```
