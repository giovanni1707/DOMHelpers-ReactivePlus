# `Validators.custom()` - Custom Validator

**Quick Start (30 seconds)**
```javascript
const form = Forms.createForm({ username: '' }, {
  validators: {
    username: Forms.v.custom(async (value) => {
      if (!value) return 'Username is required';
      if (value.length < 3) return 'Username must be 3+ characters';

      const response = await fetch(`/api/check-username/${value}`);
      const data = await response.json();

      return data.available ? null : 'Username already taken';
    })
  }
});
```

---

## **What is `Validators.custom()`?**

`Validators.custom()` is a **validator factory** that wraps a custom validation function, allowing you to implement any validation logic you need.

**Key characteristics:**
- **Full Control**: Implement any validation logic
- **Async Support**: Can be async/return promises
- **All Values Access**: Receives current and all form values
- **Flexible**: No constraints on validation rules
- **Return null or string**: null = valid, string = error

---

## **Syntax**

```javascript
Validators.custom(validatorFn)
```

### **Parameters**
- `validatorFn` (required) - Custom validation function
  - **Type**: `(value, allValues?) => string | null | Promise<string | null>`
  - **Returns**: `null` (valid) or error message string (invalid)

### **Returns**
- **Type**: The validator function itself

---

## **Examples**

### **Example 1: Username Availability**
```javascript
const form = Forms.createForm({ username: '' }, {
  validators: {
    username: Forms.v.custom(async (value) => {
      if (!value) return null; // Optional field

      const response = await fetch(`/api/users/check/${value}`);
      const { available } = await response.json();

      return available ? null : 'Username already taken';
    })
  }
});
```

### **Example 2: Password Strength**
```javascript
const form = Forms.createForm({ password: '' }, {
  validators: {
    password: Forms.v.custom((value) => {
      if (!value) return 'Password is required';

      const hasUpper = /[A-Z]/.test(value);
      const hasLower = /[a-z]/.test(value);
      const hasNumber = /[0-9]/.test(value);
      const hasSpecial = /[!@#$%^&*]/.test(value);

      if (value.length < 8) return 'Password must be 8+ characters';
      if (!hasUpper) return 'Password must include uppercase letter';
      if (!hasLower) return 'Password must include lowercase letter';
      if (!hasNumber) return 'Password must include number';
      if (!hasSpecial) return 'Password must include special character';

      return null;
    })
  }
});
```

### **Example 3: Conditional Validation**
```javascript
const form = Forms.createForm({
  accountType: 'personal',
  companyName: ''
}, {
  validators: {
    companyName: Forms.v.custom((value, allValues) => {
      // Only required if business account
      if (allValues.accountType === 'business') {
        if (!value || value.trim() === '') {
          return 'Company name required for business accounts';
        }
      }
      return null;
    })
  }
});
```

### **Example 4: Date Range Validation**
```javascript
const form = Forms.createForm({
  startDate: '',
  endDate: ''
}, {
  validators: {
    endDate: Forms.v.custom((value, allValues) => {
      if (!value) return null;

      const start = new Date(allValues.startDate);
      const end = new Date(value);

      if (end < start) {
        return 'End date must be after start date';
      }

      return null;
    })
  }
});
```

### **Example 5: Checkbox Agreement**
```javascript
const form = Forms.createForm({ agreedToTerms: false }, {
  validators: {
    agreedToTerms: Forms.v.custom((value) => {
      return value === true ? null : 'You must agree to the terms';
    })
  }
});
```

### **Example 6: File Upload Validation**
```javascript
const form = Forms.createForm({ avatar: null }, {
  validators: {
    avatar: Forms.v.custom((file) => {
      if (!file) return null; // Optional

      const maxSize = 2 * 1024 * 1024; // 2MB
      const allowedTypes = ['image/jpeg', 'image/png', 'image/gif'];

      if (file.size > maxSize) {
        return 'File must be less than 2MB';
      }

      if (!allowedTypes.includes(file.type)) {
        return 'File must be JPEG, PNG, or GIF';
      }

      return null;
    })
  }
});
```

### **Example 7: Credit Card Luhn Check**
```javascript
const form = Forms.createForm({ creditCard: '' }, {
  validators: {
    creditCard: Forms.v.custom((value) => {
      if (!value) return null;

      // Luhn algorithm
      const digits = value.replace(/\D/g, '');
      let sum = 0;
      let isEven = false;

      for (let i = digits.length - 1; i >= 0; i--) {
        let digit = parseInt(digits[i]);

        if (isEven) {
          digit *= 2;
          if (digit > 9) digit -= 9;
        }

        sum += digit;
        isEven = !isEven;
      }

      return sum % 10 === 0 ? null : 'Invalid credit card number';
    })
  }
});
```

### **Example 8: Multiple Error Checks**
```javascript
const form = Forms.createForm({ email: '' }, {
  validators: {
    email: Forms.v.custom((value) => {
      if (!value) return 'Email is required';

      if (value.length < 5) return 'Email too short';
      if (value.length > 100) return 'Email too long';
      if (!value.includes('@')) return 'Email must contain @';
      if (!value.includes('.')) return 'Email must contain domain';

      const parts = value.split('@');
      if (parts[0].length === 0) return 'Email must have username';
      if (parts[1].length === 0) return 'Email must have domain';

      return null;
    })
  }
});
```

---

## **Common Patterns**

### **Pattern 1: Async API Validation**
```javascript
Forms.v.custom(async (value) => {
  const response = await fetch(`/api/validate/${value}`);
  const { valid, error } = await response.json();
  return valid ? null : error;
})
```

### **Pattern 2: Conditional Requirement**
```javascript
Forms.v.custom((value, allValues) => {
  if (allValues.someField === 'required') {
    return value ? null : 'This field is required';
  }
  return null;
})
```

### **Pattern 3: Complex Logic**
```javascript
Forms.v.custom((value) => {
  // Your complex validation logic
  if (complexCheck(value)) return null;
  return 'Validation failed';
})
```

---

## **Key Takeaways**

1. **Full Flexibility**: Implement any validation logic
2. **Async Capable**: Can use `async/await` or return promises
3. **All Values Access**: Receives all form values for cross-field validation
4. **Return Convention**: `null` = valid, `string` = error
5. **Common Use**: API checks, complex rules, conditional validation
6. **Combine with Others**: Can be used with `combine()`

---

## **Summary**

`Validators.custom()` creates a validator from your own custom validation function, giving you complete control over validation logic. It supports both synchronous and asynchronous validation, has access to all form values for cross-field validation, and can implement any validation rules you need. The validator function should return `null` for valid values or an error message string for invalid values.
