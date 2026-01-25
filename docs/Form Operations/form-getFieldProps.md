# `form.getFieldProps()` - Get Field Props for Binding

## Quick Start (30 seconds)

```javascript
// Create form
const loginForm = form({
  username: '',
  password: ''
});

// Get field props
const usernameProps = loginForm.getFieldProps('username');

console.log(usernameProps);
// {
//   name: 'username',
//   value: '',
//   onChange: (e) => form.handleChange(e),
//   onBlur: (e) => form.handleBlur(e)
// }

// Use in React-style frameworks
function UsernameInput() {
  return <input {...usernameProps} />;
}

// Use with vanilla JS
const input = document.getElementById('username');
Object.assign(input, usernameProps);

// Manual binding
const props = loginForm.getFieldProps('password');
const passwordInput = document.getElementById('password');
passwordInput.name = props.name;
passwordInput.value = props.value;
passwordInput.addEventListener('input', props.onChange);
passwordInput.addEventListener('blur', props.onBlur);

// Values stay in sync
loginForm.setValue('username', 'alice');
console.log(usernameProps.value); // Still points to empty string
console.log(loginForm.values.username); // 'alice'
// Note: getFieldProps returns current value, not reactive
```

**That's it.** Get an object with props for easy field binding. Includes name, value, onChange, and onBlur.

---

## What is `form.getFieldProps()`?

`form.getFieldProps()` **returns an object with properties needed to bind a form field to an input element**, including the field name, current value, and event handlers.

**Syntax:**
```javascript
const props = form.getFieldProps('fieldName');
```

**Parameters:**
- `field` - String name of the field

**Returns:**
- Object with:
  - `name`: Field name string
  - `value`: Current field value (or empty string)
  - `onChange`: Function that calls `handleChange`
  - `onBlur`: Function that calls `handleBlur`

**Key Points:**
- Convenient for **binding inputs**
- Works with **React**, **Vue**, **vanilla JS**
- `value` is **snapshot** (not reactive binding)
- `onChange` calls `handleChange()`
- `onBlur` calls `handleBlur()`
- Use with spread operator `{...props}`

---

## Summary

`form.getFieldProps()` **returns props for input binding**. Includes name, value, onChange, onBlur.

```javascript
const props = form.getFieldProps('email')
// { name, value, onChange, onBlur }
```
