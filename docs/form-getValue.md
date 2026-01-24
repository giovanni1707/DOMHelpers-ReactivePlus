# `form.getValue()` - Get Field Value

## Quick Start (30 seconds)

```javascript
// Create form
const userForm = form({
  name: 'Alice',
  email: 'alice@example.com',
  age: 28
});

// Get single field value
const name = userForm.getValue('name');
console.log(name); // 'Alice'

const email = userForm.getValue('email');
console.log(email); // 'alice@example.com'

// Returns undefined for non-existent fields
const phone = userForm.getValue('phone');
console.log(phone); // undefined

// Alternative: Direct property access
console.log(userForm.values.name); // 'Alice'
```

**That's it.** Get the value of a single form field.

---

## What is `form.getValue()`?

`form.getValue()` **returns the current value of a form field**. It's a convenience method equivalent to accessing `form.values[field]` directly.

**Syntax:**
```javascript
const value = form.getValue(fieldName);
```

**Parameters:**
- `field` - String name of the field

**Returns:**
- The field's current value or `undefined` if field doesn't exist

**Key Points:**
- Equivalent to `form.values[field]`
- Returns `undefined` for non-existent fields
- **Reactive** when used in effects
- Read-only operation (doesn't modify form)

---

## Summary

`form.getValue()` **returns a field's current value**. Convenience method for accessing field values.

```javascript
const value = form.getValue('email')
// Same as: form.values.email
```
