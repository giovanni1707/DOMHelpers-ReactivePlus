# `form.submit()` - Handle Form Submission

## Quick Start (30 seconds)

```javascript
// Create form with submit handler
const loginForm = form(
  { username: '', password: '' },
  {
    validators: {
      username: (v) => v ? null : 'Required',
      password: (v) => v.length >= 8 ? null : 'Too short'
    },
    onSubmit: async (values, form) => {
      // Submit to API
      const response = await fetch('/api/login', {
        method: 'POST',
        body: JSON.stringify(values)
      });
      return response.json();
    }
  }
);

// Submit form
const result = await loginForm.submit();

if (result.success) {
  console.log('Submitted:', result.result);
} else if (result.errors) {
  console.log('Validation failed:', result.errors);
} else {
  console.log('Submit error:', result.error);
}

// Custom submit handler
const result2 = await loginForm.submit(async (values, form) => {
  console.log('Custom submit:', values);
  return { token: 'abc123' };
});

// Automatic validation
loginForm.setValues({ username: 'alice', password: 'short' });
const result3 = await loginForm.submit();
// { success: false, errors: { password: 'Too short' } }
```

**That's it.** Submit the form - automatically validates, marks all as touched, and calls submit handler.

---

## What is `form.submit()`?

`form.submit()` **handles form submission by validating all fields and calling a submit handler**. It automatically marks all fields as touched, validates them, and only calls the handler if valid.

**Syntax:**
```javascript
const result = await form.submit();
const result = await form.submit(customHandler);
```

**Parameters:**
- `customHandler` (optional) - Async function `(values, form) => Promise<any>`
  - Uses this handler instead of the one from form options
  - Receives form values and form instance

**Returns:**
- Promise that resolves to:
  - `{ success: true, result: handlerResult }` if valid and submitted
  - `{ success: false, errors: form.errors }` if validation failed
  - `{ success: false, error: errorObject }` if handler threw error

**Key Points:**
- Calls `touchAll()` first (marks all fields as touched)
- Calls `validate()` to check all fields
- Only calls handler if **validation passes**
- Sets `form.isSubmitting = true` during submission
- Increments `form.submitCount` on success
- Returns **Promise** (async)
- Handles **errors** from submit handler

---

## Summary

`form.submit()` **submits form with validation**. Auto-validates, marks touched, calls handler.

```javascript
const result = await form.submit()
// Validates, then calls onSubmit handler
```
