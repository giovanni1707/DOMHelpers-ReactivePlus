# `form.resetField()` - Reset Single Field

## Quick Start (30 seconds)

```javascript
// Create form
const form1 = form({
  username: 'alice',
  email: 'alice@example.com',
  password: ''
});

// User makes changes
form1.setValue('username', 'bob');
form1.setError('username', 'Taken');
form1.setTouched('username');

console.log(form1.values.username); // 'bob'
console.log(form1.errors.username); // 'Taken'
console.log(form1.touched.username); // true

// Reset single field
form1.resetField('username');

console.log(form1.values.username); // 'alice' (back to initial)
console.log(form1.errors.username); // undefined (cleared)
console.log(form1.touched.username); // undefined (cleared)

// Other fields unchanged
console.log(form1.values.email); // 'alice@example.com' (unchanged)

// Chainable
form1
  .resetField('username')
  .resetField('email')
  .validate();
```

**That's it.** Reset a single field to its initial value. Clears its error and touched state.

---

## What is `form.resetField()`?

`form.resetField()` **resets a single form field to its initial value** and clears its error and touched state. Other fields remain unchanged.

**Syntax:**
```javascript
form.resetField('fieldName');
```

**Parameters:**
- `field` - String name of the field to reset

**Returns:**
- The form itself (for chaining)

**Key Points:**
- Resets field to **initial value**
- Clears field **error**
- Clears field **touched** state
- Other fields **unchanged**
- **Batched** for performance
- **Chainable** (returns form)
- Triggers **reactivity** once

---

## Summary

`form.resetField()` **resets single field to initial value**. Clears error and touched.

```javascript
form.resetField('email')
```
