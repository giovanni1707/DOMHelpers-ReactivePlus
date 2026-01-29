# `form.handleChange()` - Handle Input Change Event

## Quick Start (30 seconds)

```javascript
// Create form
const form1 = form({
  email: '',
  agree: false
});

// Bind to input events
document.getElementById('email').addEventListener('input', (e) => {
  form1.handleChange(e);
});

document.getElementById('agree').addEventListener('change', (e) => {
  form1.handleChange(e);
});

// User types in email input → handleChange called automatically
// form.values.email updated
// form.touched.email set to true

// Alternative: Use getFieldProps for automatic binding
const emailProps = form1.getFieldProps('email');
// emailProps.onChange calls handleChange automatically

// HTML example:
// <input
//   name="email"
//   oninput="form1.handleChange(event)"
// />
// 
// <input
//   type="checkbox"
//   name="agree"
//   onchange="form1.handleChange(event)"
// />
```

**That's it.** Handle input change events. Automatically extracts field name and value, updates form state.

---

## What is `form.handleChange()`?

`form.handleChange()` **handles input change events by extracting the field name and value from the event target**, then calling `setValue()` to update the form.

**Syntax:**
```javascript
form.handleChange(event);
```

**Parameters:**
- `event` - DOM input/change event object

**Returns:**
- `undefined` (side-effect function)

**Key Points:**
- Extracts field name from `event.target.name` or `event.target.id`
- Extracts value (handles checkboxes: uses `checked`, others: uses `value`)
- Calls `setValue(field, value)`
- Marks field as **touched**
- **Auto-validates** if validator exists
- Used with `addEventListener` or `getFieldProps()`

---

## Summary

`form.handleChange()` **handles input change events**. Extracts field/value, calls setValue.

```javascript
input.addEventListener('input', (e) => form.handleChange(e))
```
