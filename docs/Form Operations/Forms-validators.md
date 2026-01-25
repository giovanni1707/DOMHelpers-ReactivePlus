# `Forms.validators` / `Forms.v` - Form Validation Functions

**Quick Start (30 seconds)**
```javascript
// Access validators
const { validators, v } = Forms; // v is shorthand

// Use in form creation
const form = Forms.createForm({
  email: '',
  password: '',
  confirmPassword: ''
}, {
  validators: {
    email: validators.email(),
    password: validators.minLength(8),
    confirmPassword: validators.match('password')
  }
});

// Or use shorthand
const form2 = Forms.createForm({ age: 0 }, {
  validators: {
    age: Forms.v.min(18, 'Must be 18 or older')
  }
});

// Validators also available via ReactiveUtils
const { validators } = ReactiveUtils;
```

---

## **What is `Forms.validators`?**

`Forms.validators` (aliased as `Forms.v`) is a **collection of pre-built validation functions** for common form validation scenarios. Each validator is a factory function that returns a validator function which checks a value and returns either `null` (valid) or an error message string (invalid).

**Key characteristics:**
- **Factory Functions**: Return validator functions
- **Composable**: Can be combined with `combine()`
- **Customizable**: All accept custom error messages
- **Null-Safe**: Most return `null` for empty values
- **Async-Capable**: Work with async validators via `custom()`

---

## **Available Validators**

| Validator | Purpose | Example |
|-----------|---------|---------|
| `required(message)` | Field must have a value | `validators.required()` |
| `email(message)` | Must be valid email format | `validators.email()` |
| `minLength(min, msg)` | String length >= min | `validators.minLength(8)` |
| `maxLength(max, msg)` | String length <= max | `validators.maxLength(100)` |
| `min(min, msg)` | Number value >= min | `validators.min(18)` |
| `max(max, msg)` | Number value <= max | `validators.max(100)` |
| `pattern(regex, msg)` | Matches regex pattern | `validators.pattern(/^\d{10}$/)` |
| `match(field, msg)` | Matches another field | `validators.match('password')` |
| `custom(fn)` | Custom validation logic | `validators.custom(myFn)` |
| `combine(...validators)` | Run multiple validators | `validators.combine(v1, v2)` |

---

## **Syntax**

```javascript
// Access via Forms
Forms.validators.required()
Forms.v.required()  // Shorthand

// Access via ReactiveUtils
ReactiveUtils.validators.required()

// Use in form
const form = Forms.createForm({ email: '' }, {
  validators: {
    email: Forms.validators.email('Please enter a valid email')
  }
});
```

---

## **Validator Return Values**

All validators return a function with this signature:
```javascript
(value, allValues?) => string | null
```

**Returns:**
- `null` - Validation passed (value is valid)
- `string` - Validation failed (error message)

**Parameters:**
- `value` - The field value being validated
- `allValues` - (Optional) All form values (for cross-field validation)

---

## **Why do validators exist?**

### ❌ **The Problem**
```javascript
// Manual validation is repetitive and error-prone
const form = Forms.createForm({ email: '', password: '' });

function validateEmail(value) {
  if (!value) return 'Email is required';
  if (!value.includes('@')) return 'Invalid email';
  return null;
}

function validatePassword(value) {
  if (!value) return 'Password is required';
  if (value.length < 8) return 'Password must be 8+ characters';
  return null;
}

// Have to write these for every field
```

### ✅ **The Solution**
```javascript
// Pre-built validators handle common cases
const form = Forms.createForm({ email: '', password: '' }, {
  validators: {
    email: Forms.validators.combine(
      Forms.validators.required(),
      Forms.validators.email()
    ),
    password: Forms.validators.combine(
      Forms.validators.required(),
      Forms.validators.minLength(8)
    )
  }
});

// Clean, declarative, reusable
```

---

## **Mental Model**

Think of validators as **quality checkpoints** that values must pass through:

```
User Input → Validator → Result
    ↓            ↓         ↓
  "test"    email()    "Invalid email"
    ↓            ↓         ↓
"a@b.com"  email()      null (valid)
```

**Validator pipeline:**
```
value → required() → minLength(3) → pattern(/\d/) → result
         ↓              ↓               ↓
       pass           pass             pass      → null
       fail           —                —         → "Required"
       pass           fail             —         → "Min 3 chars"
       pass           pass             fail      → "Must contain digit"
```

---

## **Examples**

### **Example 1: Basic Validators**
```javascript
const form = Forms.createForm({
  username: '',
  email: '',
  age: 0
}, {
  validators: {
    username: Forms.v.required('Username is required'),
    email: Forms.v.email(),
    age: Forms.v.min(18, 'Must be 18 or older')
  }
});
```

### **Example 2: Combining Validators**
```javascript
const form = Forms.createForm({ password: '' }, {
  validators: {
    password: Forms.v.combine(
      Forms.v.required(),
      Forms.v.minLength(8, 'Password must be 8+ characters'),
      Forms.v.pattern(/[A-Z]/, 'Must contain uppercase letter'),
      Forms.v.pattern(/[0-9]/, 'Must contain number')
    )
  }
});
```

### **Example 3: Custom Validator**
```javascript
const form = Forms.createForm({ username: '' }, {
  validators: {
    username: Forms.v.custom(async (value) => {
      if (!value) return 'Username required';

      const response = await fetch(`/api/check/${value}`);
      const data = await response.json();

      return data.available ? null : 'Username already taken';
    })
  }
});
```

### **Example 4: Cross-Field Validation**
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
```

### **Example 5: Pattern Validation**
```javascript
const form = Forms.createForm({
  phone: '',
  zipCode: '',
  creditCard: ''
}, {
  validators: {
    phone: Forms.v.pattern(/^\d{10}$/, '10 digits required'),
    zipCode: Forms.v.pattern(/^\d{5}(-\d{4})?$/, 'Invalid ZIP code'),
    creditCard: Forms.v.pattern(/^\d{16}$/, '16 digits required')
  }
});
```

---

## **Common Patterns**

### **Pattern 1: Required Field**
```javascript
validators: {
  field: Forms.v.required()
}
```

### **Pattern 2: Optional Field with Validation**
```javascript
validators: {
  // Only validates if value is provided
  website: Forms.v.pattern(/^https?:\/\//)
}
```

### **Pattern 3: Multiple Validations**
```javascript
validators: {
  field: Forms.v.combine(
    Forms.v.required(),
    Forms.v.minLength(3),
    Forms.v.maxLength(50)
  )
}
```

### **Pattern 4: Custom Error Messages**
```javascript
validators: {
  email: Forms.v.email('Please provide a valid email address'),
  age: Forms.v.min(18, 'You must be at least 18 years old')
}
```

---

## **Key Takeaways**

1. **Factory Functions**: Validators are functions that return validator functions
2. **Two Access Methods**: `Forms.validators` or shorthand `Forms.v`
3. **Null = Valid**: All validators return `null` when valid
4. **String = Error**: Validators return error message string when invalid
5. **Composable**: Use `combine()` to run multiple validators
6. **Customizable**: All accept custom error messages
7. **Null-Safe**: Most validators skip validation if value is empty
8. **Async Support**: Use `custom()` for async validation
9. **Cross-Field**: Use `match()` or `custom()` with `allValues` parameter
10. **Available Globally**: Via `Forms.validators`, `Forms.v`, or `ReactiveUtils.validators`

---

## **Summary**

`Forms.validators` (or `Forms.v` shorthand) provides a comprehensive set of pre-built validation functions for common form validation scenarios including required fields, email format, string length, numeric ranges, pattern matching, field matching, and custom logic. Each validator is a factory function that returns a validator function which accepts a value and returns either `null` (valid) or an error message string (invalid). Validators can be combined, customized with error messages, and used to create robust form validation with minimal code. See individual validator documentation for detailed examples and usage patterns.

---

## **See Also**

- `Validators.required()` - Require non-empty value
- `Validators.email()` - Validate email format
- `Validators.minLength()` - Minimum string length
- `Validators.maxLength()` - Maximum string length
- `Validators.min()` - Minimum numeric value
- `Validators.max()` - Maximum numeric value
- `Validators.pattern()` - Regex pattern matching
- `Validators.match()` - Match another field
- `Validators.custom()` - Custom validation logic
- `Validators.combine()` - Combine multiple validators
