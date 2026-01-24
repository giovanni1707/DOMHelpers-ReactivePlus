# `form.shouldShowError()` - Check if Should Show Error

## Quick Start (30 seconds)

```javascript
// Create form with validators
const form1 = form(
  { email: '' },
  {
    validators: {
      email: (v) => v.includes('@') ? null : 'Invalid email'
    }
  }
);

// User types invalid email
form1.setValue('email', 'invalid'); // Marks as touched, validates

// Check if should show error
console.log(form1.shouldShowError('email')); // true
// (touched: true AND has error: true)

// Before user interaction
const form2 = form({ email: '' });
form2.setError('email', 'Required');
console.log(form2.shouldShowError('email')); // false
// (has error but NOT touched)

// After user interacts
form2.setTouched('email');
console.log(form2.shouldShowError('email')); // true
// (touched AND has error)

// Use in UI
effect(() => {
  const errorDiv = document.getElementById('email-error');
  errorDiv.style.display = form1.shouldShowError('email') ? 'block' : 'none';
  errorDiv.textContent = form1.getError('email') || '';
});
```

**That's it.** Check if an error should be displayed to the user. Returns true only if field is both touched AND has an error.

---

## What is `form.shouldShowError()`?

`form.shouldShowError()` **returns true if a field should display its error to the user** — specifically, if the field is both touched (user has interacted with it) and has an error.

**Syntax:**
```javascript
const show = form.shouldShowError('fieldName');
```

**Parameters:**
- `field` - String name of the field

**Returns:**
- `true` if field is touched AND has error, `false` otherwise

**Key Points:**
- Combines `isTouched(field) && hasError(field)`
- Returns **boolean**
- **Reactive** when used in effects
- Better UX - don't show errors before interaction
- Common pattern for form validation UI

---

## Summary

`form.shouldShowError()` **returns true if field is touched AND has error**. UX helper for showing errors.

```javascript
if (form.shouldShowError('email')) {
  // Show error to user
}
// Same as: isTouched('email') && hasError('email')
```
