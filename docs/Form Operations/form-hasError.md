# `form.hasError()` - Check if Field Has Error

## Quick Start (30 seconds)

```javascript
// Create form
const contactForm = form({
  name: '',
  email: ''
});

// Set an error
contactForm.setError('email', 'Invalid email format');

// Check if field has error
console.log(contactForm.hasError('email'));  // true
console.log(contactForm.hasError('name'));   // false

// Use in conditionals
if (contactForm.hasError('email')) {
  console.log('Email has an error!');
}

// Reactive in effects
effect(() => {
  const hasEmailError = contactForm.hasError('email');
  const errorEl = document.getElementById('email-error');
  errorEl.style.display = hasEmailError ? 'block' : 'none';
});
```

**That's it.** Check if a form field has an error message. Returns boolean.

---

## What is `form.hasError()`?

`form.hasError()` **returns true if a field has an error message, false otherwise**. It's a convenience method equivalent to `!!form.errors[field]`.

**Syntax:**
```javascript
const hasError = form.hasError('fieldName');
```

**Parameters:**
- `field` - String name of the field

**Returns:**
- `true` if field has error, `false` otherwise

**Key Points:**
- Equivalent to `!!form.errors[field]`
- Returns **boolean**
- **Reactive** when used in effects
- More semantic than checking errors directly
- Works with `shouldShowError()` for UX

---

## Summary

`form.hasError()` **returns true if field has an error**. Semantic boolean check.

```javascript
if (form.hasError('email')) { ... }
// Same as: if (form.errors.email) { ... }
```
