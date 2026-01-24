# `Validators.match()` - Field Matching Validator

**Quick Start (30 seconds)**
```javascript
const form = Forms.createForm({
  password: '',
  confirmPassword: ''
}, {
  validators: {
    password: Forms.v.minLength(8),
    confirmPassword: Forms.v.match('password', 'Passwords must match')
  }
});

form.setValue('password', 'secret123');
form.setValue('confirmPassword', 'secret456');
form.validateField('confirmPassword');
console.log(form.errors.confirmPassword); // "Passwords must match"

form.setValue('confirmPassword', 'secret123');
form.validateField('confirmPassword');
console.log(form.errors.confirmPassword); // null (valid)
```

---

## **What is `Validators.match()`?**

`Validators.match()` is a **validator factory** that creates a validator function to ensure a field value matches another field's value (cross-field validation).

**Key characteristics:**
- **Cross-Field Validation**: Compares with another field
- **Strict Equality**: Uses `===` for comparison
- **Receives All Values**: Has access to all form values
- **Custom Messages**: Accepts custom error message
- **Common Use**: Password confirmation, email confirmation

---

## **Syntax**

```javascript
Validators.match(fieldName, message?)
```

### **Parameters**
- `fieldName` (required) - Name of field to match against
  - **Type**: `string`
- `message` (optional) - Custom error message
  - **Type**: `string`
  - **Default**: `'Must match {fieldName}'`

### **Returns**
- **Type**: `(value, allValues) => string | null`

---

## **Examples**

### **Example 1: Password Confirmation**
```javascript
const form = Forms.createForm({
  password: '',
  confirmPassword: ''
}, {
  validators: {
    confirmPassword: Forms.v.match('password', 'Passwords must match')
  }
});
```

### **Example 2: Email Confirmation**
```javascript
const form = Forms.createForm({
  email: '',
  confirmEmail: ''
}, {
  validators: {
    email: Forms.v.email(),
    confirmEmail: Forms.v.combine(
      Forms.v.email(),
      Forms.v.match('email', 'Email addresses must match')
    )
  }
});
```

### **Example 3: Bidirectional Validation**
```javascript
const form = Forms.createForm({
  password: '',
  confirmPassword: ''
}, {
  validators: {
    password: Forms.v.minLength(8),
    confirmPassword: Forms.v.match('password')
  }
});

// Re-validate confirmation when password changes
effect(() => {
  const passwordValue = form.values.password;

  if (form.touched.confirmPassword) {
    form.validateField('confirmPassword');
  }
});
```

### **Example 4: Custom Match Logic**
```javascript
// For more complex matching, use custom validator
const form = Forms.createForm({
  newPassword: '',
  confirmPassword: ''
}, {
  validators: {
    confirmPassword: Forms.v.custom((value, allValues) => {
      if (value !== allValues.newPassword) {
        return 'Passwords must match';
      }
      if (value === allValues.oldPassword) {
        return 'New password cannot be same as old password';
      }
      return null;
    })
  }
});
```

---

## **Common Patterns**

### **Pattern 1: Password Confirmation**
```javascript
validators: {
  confirmPassword: Forms.v.match('password', 'Passwords must match')
}
```

### **Pattern 2: Email Confirmation**
```javascript
validators: {
  confirmEmail: Forms.v.match('email', 'Email addresses must match')
}
```

### **Pattern 3: Combined Validation**
```javascript
validators: {
  confirmEmail: Forms.v.combine(
    Forms.v.required(),
    Forms.v.email(),
    Forms.v.match('email')
  )
}
```

---

## **Key Takeaways**

1. **Cross-Field**: Compares against another field's value
2. **Strict Equality**: Uses `===` comparison
3. **All Values Access**: Receives `allValues` parameter
4. **Common Use**: Password/email confirmation
5. **Re-validate**: Consider re-validating when target field changes
6. **Default Message**: Generates message with field name

---

## **Summary**

`Validators.match()` creates a validator that ensures a field value matches another field's value using strict equality. It's commonly used for password and email confirmation fields. The validator receives access to all form values and returns `null` when values match or an error message when they don't.
