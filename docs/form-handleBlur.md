# `form.handleBlur()` - Handle Input Blur Event

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

// Bind to blur event
document.getElementById('email').addEventListener('blur', (e) => {
  form1.handleBlur(e);
});

// User clicks away from input → handleBlur called
// form.touched.email set to true
// Validates email field

// Alternative: Use getFieldProps
const emailProps = form1.getFieldProps('email');
// emailProps.onBlur calls handleBlur automatically

// HTML example:
// <input
//   name="email"
//   onblur="form1.handleBlur(event)"
// />
```

**That's it.** Handle input blur events (when user leaves field). Marks field as touched and validates.

---

## What is `form.handleBlur()`?

`form.handleBlur()` **handles input blur events by marking the field as touched and validating it** if a validator exists.

**Syntax:**
```javascript
form.handleBlur(event);
```

**Parameters:**
- `event` - DOM blur event object

**Returns:**
- `undefined` (side-effect function)

**Key Points:**
- Extracts field name from `event.target.name` or `event.target.id`
- Calls `setTouched(field)`
- Calls `validateField(field)` if validator exists
- Used with `addEventListener` or `getFieldProps()`
- Better UX - validate when user leaves field

---

## Summary

`form.handleBlur()` **handles blur events**. Marks touched, validates field.

```javascript
input.addEventListener('blur', (e) => form.handleBlur(e))
```
