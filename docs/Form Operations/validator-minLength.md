# `Validators.minLength()` - Minimum Length Validator

**Quick Start (30 seconds)**
```javascript
const form = Forms.createForm({ password: '' }, {
  validators: {
    password: Forms.v.minLength(8)
  }
});

form.setValue('password', 'abc');
form.validateField('password');
console.log(form.errors.password); // "Must be at least 8 characters"

form.setValue('password', 'abcd1234');
form.validateField('password');
console.log(form.errors.password); // null (valid)
```

---

## **What is `Validators.minLength()`?**

`Validators.minLength()` is a **validator factory** that creates a validator function to ensure a string field has a minimum length.

**Key characteristics:**
- **String Length**: Checks `value.length >= min`
- **Optional Field**: Returns `null` if value is empty
- **Custom Messages**: Accepts custom error message
- **Default Message**: Auto-generates message with min value

---

## **Syntax**

```javascript
Validators.minLength(min, message?)
```

### **Parameters**
- `min` (required) - Minimum length
  - **Type**: `number`
- `message` (optional) - Custom error message
  - **Type**: `string`
  - **Default**: `'Must be at least {min} characters'`

### **Returns**
- **Type**: `(value) => string | null`

---

## **Examples**

### **Example 1: Password Length**
```javascript
const form = Forms.createForm({ password: '' }, {
  validators: {
    password: Forms.v.combine(
      Forms.v.required(),
      Forms.v.minLength(8, 'Password must be at least 8 characters')
    )
  }
});
```

### **Example 2: Username Length**
```javascript
const form = Forms.createForm({ username: '' }, {
  validators: {
    username: Forms.v.minLength(3, 'Username too short')
  }
});
```

---

## **Key Takeaways**

1. **String Length**: Validates character count, not byte size
2. **Optional**: Empty values pass validation
3. **Inclusive**: Min length is inclusive (>= not >)
4. **Combine with Required**: Use with `required()` for mandatory fields
5. **Auto Message**: Generates helpful default message

---

## **Summary**

`Validators.minLength()` creates a validator that ensures a string field has at least a minimum number of characters. It returns `null` for empty values or valid lengths, and an error message for strings that are too short.
