# `form.touchAll()` - Mark All Fields as Touched

## Quick Start (30 seconds)

```javascript
// Create form
const contactForm = form({
  name: '',
  email: '',
  phone: '',
  message: ''
});

// Mark all fields as touched
contactForm.touchAll();

console.log(contactForm.touched);
// { name: true, email: true, phone: true, message: true }

// Common pattern: touch all before validation
function handleSubmit() {
  contactForm
    .touchAll()
    .validate();
  
  if (contactForm.isValid) {
    // Submit
  }
}

// Chainable
contactForm
  .touchAll()
  .validate()
  .submit();
```

**That's it.** Mark all form fields as touched at once. Commonly used before form submission.

---

## What is `form.touchAll()`?

`form.touchAll()` **marks all form fields as touched in a single batched operation**. It's equivalent to calling `setTouchedFields()` with all field names.

**Syntax:**
```javascript
form.touchAll();
```

**Parameters:**
- None

**Returns:**
- The form itself (for chaining)

**Key Points:**
- Marks **all fields** in `form.values` as touched
- **Batched** for performance
- **Chainable** (returns form)
- Triggers **reactivity** once
- Automatically called by `submit()`
- Useful to show all validation errors at once

---

## Summary

`form.touchAll()` **marks all fields as touched**. Batched, chainable, used before submission.

```javascript
form.touchAll()
```
