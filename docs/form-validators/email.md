# `Validators.email()` - Email Format Validator

**Quick Start (30 seconds)**
```javascript
const form = Forms.createForm({ email: '' }, {
  validators: {
    email: Forms.v.email()
  }
});

form.setValue('email', 'invalid');
form.validateField('email');
console.log(form.errors.email); // "Invalid email address"

form.setValue('email', 'user@example.com');
form.validateField('email');
console.log(form.errors.email); // null (valid)
```

---

## **What is `Validators.email()`?**

`Validators.email()` is a **validator factory** that creates a validator function to ensure a field contains a valid email address format.

**Validation Pattern**: `/^[^\s@]+@[^\s@]+\.[^\s@]+$/`

**Key characteristics:**
- **Format Validation**: Checks basic email structure
- **Optional Field**: Returns `null` if value is empty
- **Custom Messages**: Accepts custom error message
- **Simple Pattern**: Basic email validation (not RFC-compliant)

---

## **Syntax**

```javascript
Validators.email(message?)
```

### **Parameters**
- `message` (optional) - Custom error message
  - **Type**: `string`
  - **Default**: `'Invalid email address'`

### **Returns**
- **Type**: `(value) => string | null`

---

## **Examples**

### **Example 1: Basic Email Validation**
```javascript
const form = Forms.createForm({ email: '' }, {
  validators: {
    email: Forms.v.email()
  }
});

form.setValue('email', 'user@example.com');
form.validateField('email');
console.log(form.errors.email); // null
```

### **Example 2: Required Email**
```javascript
const form = Forms.createForm({ email: '' }, {
  validators: {
    email: Forms.v.combine(
      Forms.v.required('Email is required'),
      Forms.v.email('Please enter a valid email')
    )
  }
});
```

---

## **Key Takeaways**

1. **Format Only**: Validates email format, not existence
2. **Optional**: Empty values pass validation
3. **Basic Pattern**: Simple regex, not RFC-compliant
4. **Combine with Required**: Use with `required()` for mandatory emails
5. **Custom Message**: Provide user-friendly error messages

---

## **Summary**

`Validators.email()` creates a validator that ensures an email field contains a valid email format using a simple regex pattern. It returns `null` for empty values or valid emails, and an error message for invalid formats.
