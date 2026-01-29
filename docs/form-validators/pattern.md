# `Validators.pattern()` - Pattern Matching Validator

**Quick Start (30 seconds)**
```javascript
const form = Forms.createForm({ phone: '', zipCode: '' }, {
  validators: {
    phone: Forms.v.pattern(/^\d{10}$/, '10 digits required'),
    zipCode: Forms.v.pattern(/^\d{5}(-\d{4})?$/, 'Invalid ZIP code')
  }
});

form.setValue('phone', '555-1234');
form.validateField('phone');
console.log(form.errors.phone); // "10 digits required"

form.setValue('phone', '5551234567');
form.validateField('phone');
console.log(form.errors.phone); // null (valid)
```

---

## **What is `Validators.pattern()`?**

`Validators.pattern()` is a **validator factory** that creates a validator function to ensure a field value matches a regular expression pattern.

**Key characteristics:**
- **Regex Validation**: Tests value against regex pattern
- **Optional Field**: Returns `null` if value is empty
- **Custom Messages**: Accepts custom error message
- **Flexible**: Works with any valid regex pattern

---

## **Syntax**

```javascript
Validators.pattern(regex, message?)
```

### **Parameters**
- `regex` (required) - Regular expression to test against
  - **Type**: `RegExp`
- `message` (optional) - Custom error message
  - **Type**: `string`
  - **Default**: `'Invalid format'`

### **Returns**
- **Type**: `(value) => string | null`

---

## **Examples**

### **Example 1: Phone Number**
```javascript
const form = Forms.createForm({ phone: '' }, {
  validators: {
    phone: Forms.v.pattern(/^\d{10}$/, 'Phone must be 10 digits')
  }
});
```

### **Example 2: ZIP Code**
```javascript
const form = Forms.createForm({ zipCode: '' }, {
  validators: {
    zipCode: Forms.v.pattern(/^\d{5}(-\d{4})?$/, 'Invalid ZIP code format')
  }
});
```

### **Example 3: Username (alphanumeric + underscore)**
```javascript
const form = Forms.createForm({ username: '' }, {
  validators: {
    username: Forms.v.pattern(/^[a-zA-Z0-9_]+$/, 'Only letters, numbers, and underscores allowed')
  }
});
```

### **Example 4: Credit Card**
```javascript
const form = Forms.createForm({ creditCard: '' }, {
  validators: {
    creditCard: Forms.v.pattern(/^\d{16}$/, 'Credit card must be 16 digits')
  }
});
```

### **Example 5: URL**
```javascript
const form = Forms.createForm({ website: '' }, {
  validators: {
    website: Forms.v.pattern(/^https?:\/\/.+/, 'Must be a valid URL starting with http:// or https://')
  }
});
```

### **Example 6: Hex Color**
```javascript
const form = Forms.createForm({ color: '' }, {
  validators: {
    color: Forms.v.pattern(/^#[0-9A-Fa-f]{6}$/, 'Must be a valid hex color (e.g., #FF5733)')
  }
});
```

### **Example 7: Password Strength (contains uppercase)**
```javascript
const form = Forms.createForm({ password: '' }, {
  validators: {
    password: Forms.v.combine(
      Forms.v.minLength(8),
      Forms.v.pattern(/[A-Z]/, 'Must contain at least one uppercase letter'),
      Forms.v.pattern(/[0-9]/, 'Must contain at least one number'),
      Forms.v.pattern(/[!@#$%^&*]/, 'Must contain at least one special character')
    )
  }
});
```

### **Example 8: Date Format (YYYY-MM-DD)**
```javascript
const form = Forms.createForm({ date: '' }, {
  validators: {
    date: Forms.v.pattern(/^\d{4}-\d{2}-\d{2}$/, 'Date must be in YYYY-MM-DD format')
  }
});
```

### **Example 9: IP Address**
```javascript
const form = Forms.createForm({ ip: '' }, {
  validators: {
    ip: Forms.v.pattern(
      /^((25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.){3}(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)$/,
      'Invalid IP address'
    )
  }
});
```

### **Example 10: Social Security Number**
```javascript
const form = Forms.createForm({ ssn: '' }, {
  validators: {
    ssn: Forms.v.pattern(/^\d{3}-\d{2}-\d{4}$/, 'SSN must be in format XXX-XX-XXXX')
  }
});
```

---

## **Common Patterns**

### **Pattern 1: Phone Number (US)**
```javascript
Forms.v.pattern(/^\d{10}$/, '10 digits required')
// Or with formatting:
Forms.v.pattern(/^\d{3}-\d{3}-\d{4}$/, 'Format: XXX-XXX-XXXX')
```

### **Pattern 2: Alphanumeric Only**
```javascript
Forms.v.pattern(/^[a-zA-Z0-9]+$/, 'Only letters and numbers allowed')
```

### **Pattern 3: No Special Characters**
```javascript
Forms.v.pattern(/^[a-zA-Z\s]+$/, 'Only letters and spaces allowed')
```

### **Pattern 4: URL**
```javascript
Forms.v.pattern(/^https?:\/\/.+/, 'Must be a valid URL')
```

---

## **Regex Quick Reference**

| Pattern | Matches | Example |
|---------|---------|---------|
| `^\d{10}$` | Exactly 10 digits | 5551234567 |
| `^[a-z]+$` | Lowercase letters only | hello |
| `^[A-Z]+$` | Uppercase letters only | HELLO |
| `^[a-zA-Z]+$` | Any letters | Hello |
| `^\d{3}-\d{2}-\d{4}$` | XXX-XX-XXXX format | 123-45-6789 |
| `^#[0-9A-Fa-f]{6}$` | Hex color | #FF5733 |
| `^https?://` | Starts with http(s):// | https://example.com |

---

## **Key Takeaways**

1. **Regex Testing**: Uses `regex.test(value)` method
2. **Optional**: Empty values pass validation
3. **Case Sensitive**: Use regex flags for case-insensitive matching
4. **Multiple Patterns**: Use `combine()` for multiple pattern checks
5. **Clear Messages**: Provide helpful error messages explaining the format
6. **Test Patterns**: Always test regex patterns before deploying
7. **Escape Special Chars**: Remember to escape special regex characters
8. **Common Use**: Phone numbers, postal codes, usernames, URLs

---

## **Summary**

`Validators.pattern()` creates a validator that ensures a field value matches a regular expression pattern. It's perfect for validating specific formats like phone numbers, postal codes, URLs, and custom patterns. The validator returns `null` for empty values or matching patterns, and an error message for non-matching values. Always provide clear error messages that explain the expected format to users.
