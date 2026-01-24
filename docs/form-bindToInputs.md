# `form.bindToInputs()` - Bind Form to DOM Inputs

## Quick Start (30 seconds)

```javascript
// Create form
const contactForm = form({
  name: '',
  email: '',
  message: ''
});

// HTML:
// <input name="name" />
// <input name="email" />
// <textarea name="message"></textarea>

// Bind to all inputs with name attributes
contactForm.bindToInputs('input, textarea');

// Now form automatically syncs with inputs:
// - Initial values set on inputs
// - Input changes update form
// - Blur events trigger validation

// Or bind by container
// <form id="contact-form">
//   <input name="name" />
//   <input name="email" />
// </form>

contactForm.bindToInputs('#contact-form input');

// Or pass NodeList
const inputs = document.querySelectorAll('[data-form="contact"]');
contactForm.bindToInputs(inputs);

// Chainable
contactForm
  .bindToInputs('input')
  .validate();
```

**That's it.** Automatically bind form to DOM inputs. Two-way sync with automatic event handlers.

---

## What is `form.bindToInputs()`?

`form.bindToInputs()` **automatically binds form fields to DOM input elements** by setting initial values and adding event listeners for input and blur events.

**Syntax:**
```javascript
form.bindToInputs(selector);
form.bindToInputs(nodeList);
```

**Parameters:**
- `selector` - CSS selector string or NodeList/array of input elements

**Returns:**
- The form itself (for chaining)

**Key Points:**
- Sets **initial values** on inputs from form
- Handles **checkboxes** (uses `checked`)
- Handles **other inputs** (uses `value`)
- Adds `input` event → `handleChange()`
- Adds `blur` event → `handleBlur()`
- Matches fields by `name` or `id` attribute
- Skips inputs without `name` or `id`
- **Chainable** (returns form)

---

## Summary

`form.bindToInputs()` **auto-binds form to DOM inputs**. Sets values, adds event listeners.

```javascript
form.bindToInputs('input, textarea, select')
```
