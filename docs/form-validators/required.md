# `Validators.required()` - Required Field Validator

**Quick Start (30 seconds)**
```javascript
const form = Forms.createForm({ name: '', email: '' }, {
  validators: {
    name: Forms.v.required(),
    email: Forms.v.required('Email address is required')
  }
});

form.setValue('name', '');
form.validateField('name');
console.log(form.errors.name); // "This field is required"

form.setValue('name', '  ');  // Whitespace only
form.validateField('name');
console.log(form.errors.name); // "This field is required"

form.setValue('name', 'John');
form.validateField('name');
console.log(form.errors.name); // null (valid)
```

---

## **What is `Validators.required()`?**

`Validators.required()` is a **validator factory** that creates a validator function to ensure a field has a non-empty value. It rejects falsy values and strings containing only whitespace.

**Key characteristics:**
- **Factory Function**: Returns a validator function
- **Whitespace-Aware**: Trims strings before checking
- **Falsy Check**: Rejects `null`, `undefined`, `false`, `0`, `''`
- **Custom Messages**: Accepts custom error message
- **Most Common**: Usually the first validator in a chain

---

## **Syntax**

```javascript
Validators.required(message?)
```

### **Parameters**
- `message` (optional) - Custom error message
  - **Type**: `string`
  - **Default**: `'This field is required'`

### **Returns**
- **Type**: `(value) => string | null`
- **Returns**:
  - `null` - Value is valid (truthy and non-whitespace)
  - `string` - Error message (value is falsy or whitespace-only)

---

## **Validation Logic**

```javascript
function required(message = 'This field is required') {
  return (value) => {
    if (!value || (typeof value === 'string' && value.trim() === '')) {
      return message;
    }
    return null;
  };
}
```

**Fails when:**
- `value` is `null`
- `value` is `undefined`
- `value` is `false`
- `value` is `0`
- `value` is `''` (empty string)
- `value` is `'   '` (whitespace only)

**Passes when:**
- `value` is any truthy non-string value (e.g., `true`, `1`, `[]`, `{}`)
- `value` is a non-empty, non-whitespace string

---

## **Examples**

### **Example 1: Basic Required Field**
```javascript
const form = Forms.createForm({ username: '' }, {
  validators: {
    username: Forms.v.required()
  }
});

form.setValue('username', '');
form.validateField('username');
console.log(form.errors.username); // "This field is required"

form.setValue('username', 'john_doe');
form.validateField('username');
console.log(form.errors.username); // null
```

### **Example 2: Custom Error Message**
```javascript
const form = Forms.createForm({ email: '' }, {
  validators: {
    email: Forms.v.required('Please enter your email address')
  }
});

form.validateField('email');
console.log(form.errors.email); // "Please enter your email address"
```

### **Example 3: Multiple Required Fields**
```javascript
const form = Forms.createForm({
  firstName: '',
  lastName: '',
  email: ''
}, {
  validators: {
    firstName: Forms.v.required('First name is required'),
    lastName: Forms.v.required('Last name is required'),
    email: Forms.v.required('Email is required')
  }
});
```

### **Example 4: Whitespace Handling**
```javascript
const form = Forms.createForm({ name: '' }, {
  validators: {
    name: Forms.v.required()
  }
});

form.setValue('name', '   ');  // Only spaces
form.validateField('name');
console.log(form.errors.name); // "This field is required"

form.setValue('name', ' John ');  // Has content (will be trimmed)
form.validateField('name');
console.log(form.errors.name); // null (valid)
```

### **Example 5: Combined with Other Validators**
```javascript
const form = Forms.createForm({ password: '' }, {
  validators: {
    password: Forms.v.combine(
      Forms.v.required('Password is required'),
      Forms.v.minLength(8, 'Password must be at least 8 characters')
    )
  }
});

// Empty value fails required validator
form.setValue('password', '');
form.validateField('password');
console.log(form.errors.password); // "Password is required"

// Short value passes required but fails minLength
form.setValue('password', 'abc');
form.validateField('password');
console.log(form.errors.password); // "Password must be at least 8 characters"
```

### **Example 6: Number Fields**
```javascript
const form = Forms.createForm({ age: 0 }, {
  validators: {
    age: Forms.v.required('Age is required')
  }
});

// 0 is considered falsy, so it fails required validation
form.setValue('age', 0);
form.validateField('age');
console.log(form.errors.age); // "Age is required"

// For numeric fields, you might want a custom validator instead
const form2 = Forms.createForm({ age: 0 }, {
  validators: {
    age: Forms.v.custom((value) => {
      if (value === null || value === undefined || value === '') {
        return 'Age is required';
      }
      return null;
    })
  }
});
```

### **Example 7: Checkbox Required**
```javascript
const form = Forms.createForm({ agreedToTerms: false }, {
  validators: {
    agreedToTerms: Forms.v.custom((value) => {
      return value === true ? null : 'You must agree to the terms';
    })
  }
});

// Note: required() won't work well for checkboxes since false is falsy
```

### **Example 8: Select Field**
```javascript
const form = Forms.createForm({ country: '' }, {
  validators: {
    country: Forms.v.required('Please select a country')
  }
});

form.setValue('country', '');
form.validateField('country');
console.log(form.errors.country); // "Please select a country"

form.setValue('country', 'USA');
form.validateField('country');
console.log(form.errors.country); // null
```

### **Example 9: Conditional Required**
```javascript
const form = Forms.createForm({
  accountType: 'personal',
  companyName: ''
}, {
  validators: {
    companyName: Forms.v.custom((value, allValues) => {
      // Only required if account type is business
      if (allValues.accountType === 'business') {
        if (!value || value.trim() === '') {
          return 'Company name is required for business accounts';
        }
      }
      return null;
    })
  }
});
```

### **Example 10: Real-time Required Validation**
```javascript
const form = Forms.createForm({ email: '' }, {
  validators: {
    email: Forms.v.required('Email is required')
  }
});

effect(() => {
  const errorEl = querySelector('#email-error');

  if (form.touched.email && form.errors.email) {
    errorEl.textContent = form.errors.email;
    errorEl.style.display = 'block';
  } else {
    errorEl.style.display = 'none';
  }
});
```

---

## **Common Patterns**

### **Pattern 1: Simple Required**
```javascript
validators: {
  field: Forms.v.required()
}
```

### **Pattern 2: Required with Custom Message**
```javascript
validators: {
  field: Forms.v.required('This field cannot be empty')
}
```

### **Pattern 3: Required + Length Validation**
```javascript
validators: {
  field: Forms.v.combine(
    Forms.v.required(),
    Forms.v.minLength(3),
    Forms.v.maxLength(50)
  )
}
```

### **Pattern 4: Required + Format Validation**
```javascript
validators: {
  email: Forms.v.combine(
    Forms.v.required('Email is required'),
    Forms.v.email('Invalid email format')
  )
}
```

---

## **Edge Cases**

| Input Value | Valid? | Reason |
|-------------|--------|--------|
| `''` | ❌ | Empty string |
| `'   '` | ❌ | Whitespace only (trimmed to empty) |
| `' a '` | ✅ | Has content after trim |
| `0` | ❌ | Falsy value |
| `false` | ❌ | Falsy value |
| `null` | ❌ | Falsy value |
| `undefined` | ❌ | Falsy value |
| `'0'` | ✅ | Non-empty string |
| `[]` | ✅ | Truthy value |
| `{}` | ✅ | Truthy value |

---

## **Key Takeaways**

1. **Most Common Validator**: Used for fields that must have a value
2. **Whitespace-Aware**: Trims strings before checking
3. **Falsy Values Fail**: Rejects all falsy values including `0` and `false`
4. **Custom Messages**: Always provide user-friendly error messages
5. **First in Chain**: Usually the first validator when combining
6. **Not for Checkboxes**: Use custom validator for boolean fields
7. **Not for Numbers**: Be careful with numeric fields where `0` is valid
8. **Empty Check Only**: Doesn't validate format, only presence

---

## **Summary**

`Validators.required()` creates a validator that ensures a form field has a non-empty value. It rejects falsy values and strings containing only whitespace, returning either `null` (valid) or an error message (invalid). This is the most commonly used validator for ensuring required form fields are filled out. Be aware of edge cases with numeric fields (where `0` might be valid) and boolean fields (where `false` is valid), which may require custom validators instead.
