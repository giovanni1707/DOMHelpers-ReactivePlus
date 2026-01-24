# `Validators.combine()` - Combine Multiple Validators

**Quick Start (30 seconds)**
```javascript
const form = Forms.createForm({ password: '' }, {
  validators: {
    password: Forms.v.combine(
      Forms.v.required('Password is required'),
      Forms.v.minLength(8, 'Password must be 8+ characters'),
      Forms.v.pattern(/[A-Z]/, 'Must contain uppercase letter'),
      Forms.v.pattern(/[0-9]/, 'Must contain number')
    )
  }
});

form.setValue('password', 'abc');
form.validateField('password');
console.log(form.errors.password); // "Password must be 8+ characters"

form.setValue('password', 'abcd1234');
form.validateField('password');
console.log(form.errors.password); // "Must contain uppercase letter"

form.setValue('password', 'Abcd1234');
form.validateField('password');
console.log(form.errors.password); // null (valid)
```

---

## **What is `Validators.combine()`?**

`Validators.combine()` is a **validator factory** that combines multiple validators into a single validator that runs them in sequence, returning the first error encountered.

**Key characteristics:**
- **Sequential Execution**: Runs validators in order
- **Fails Fast**: Returns first error, stops checking rest
- **Composable**: Combine any validators together
- **Order Matters**: Place most critical validators first
- **Clean Syntax**: Avoid nested function calls

---

## **Syntax**

```javascript
Validators.combine(...validators)
```

### **Parameters**
- `...validators` (required) - Validator functions to combine
  - **Type**: `...((value, allValues?) => string | null)[]`
  - **Any Number**: Pass as many validators as needed

### **Returns**
- **Type**: `(value, allValues) => string | null`
- **Behavior**: Returns first error or `null` if all pass

---

## **How It Works**

```javascript
function combine(...validators) {
  return (value, allValues) => {
    for (const validator of validators) {
      const error = validator(value, allValues);
      if (error) return error; // First error wins
    }
    return null; // All passed
  };
}
```

---

## **Examples**

### **Example 1: Required Email**
```javascript
const form = Forms.createForm({ email: '' }, {
  validators: {
    email: Forms.v.combine(
      Forms.v.required('Email is required'),
      Forms.v.email('Invalid email format')
    )
  }
});

// Empty: Shows "Email is required"
// "invalid": Shows "Invalid email format"
// "user@example.com": Valid (null)
```

### **Example 2: Password Strength**
```javascript
const form = Forms.createForm({ password: '' }, {
  validators: {
    password: Forms.v.combine(
      Forms.v.required('Password required'),
      Forms.v.minLength(8, 'Minimum 8 characters'),
      Forms.v.pattern(/[A-Z]/, 'Must have uppercase'),
      Forms.v.pattern(/[a-z]/, 'Must have lowercase'),
      Forms.v.pattern(/[0-9]/, 'Must have number'),
      Forms.v.pattern(/[!@#$%^&*]/, 'Must have special character')
    )
  }
});
```

### **Example 3: Username Validation**
```javascript
const form = Forms.createForm({ username: '' }, {
  validators: {
    username: Forms.v.combine(
      Forms.v.required(),
      Forms.v.minLength(3, 'Username must be 3+ characters'),
      Forms.v.maxLength(20, 'Username must be 20 or fewer characters'),
      Forms.v.pattern(/^[a-zA-Z0-9_]+$/, 'Only letters, numbers, and underscores allowed')
    )
  }
});
```

### **Example 4: Phone Number**
```javascript
const form = Forms.createForm({ phone: '' }, {
  validators: {
    phone: Forms.v.combine(
      Forms.v.required('Phone number is required'),
      Forms.v.pattern(/^\d{10}$/, 'Phone must be exactly 10 digits')
    )
  }
});
```

### **Example 5: Age Validation**
```javascript
const form = Forms.createForm({ age: 0 }, {
  validators: {
    age: Forms.v.combine(
      Forms.v.required('Age is required'),
      Forms.v.min(18, 'Must be 18 or older'),
      Forms.v.max(120, 'Invalid age')
    )
  }
});
```

### **Example 6: Confirm Password**
```javascript
const form = Forms.createForm({
  password: '',
  confirmPassword: ''
}, {
  validators: {
    password: Forms.v.combine(
      Forms.v.required(),
      Forms.v.minLength(8)
    ),
    confirmPassword: Forms.v.combine(
      Forms.v.required(),
      Forms.v.match('password', 'Passwords must match')
    )
  }
});
```

### **Example 7: With Custom Validator**
```javascript
const form = Forms.createForm({ username: '' }, {
  validators: {
    username: Forms.v.combine(
      Forms.v.required(),
      Forms.v.minLength(3),
      Forms.v.custom(async (value) => {
        const response = await fetch(`/api/check/${value}`);
        const { available } = await response.json();
        return available ? null : 'Username taken';
      })
    )
  }
});
```

### **Example 8: Order Matters**
```javascript
const form = Forms.createForm({ email: '' }, {
  validators: {
    email: Forms.v.combine(
      // Check required first (fast, synchronous)
      Forms.v.required('Email required'),

      // Then check format (fast, synchronous)
      Forms.v.email('Invalid email'),

      // Finally check availability (slow, async)
      Forms.v.custom(async (value) => {
        const response = await fetch(`/api/check-email/${value}`);
        const { available } = await response.json();
        return available ? null : 'Email already registered';
      })
    )
  }
});
```

---

## **Common Patterns**

### **Pattern 1: Required + Format**
```javascript
Forms.v.combine(
  Forms.v.required(),
  Forms.v.email()
)
```

### **Pattern 2: Required + Length**
```javascript
Forms.v.combine(
  Forms.v.required(),
  Forms.v.minLength(3),
  Forms.v.maxLength(50)
)
```

### **Pattern 3: Required + Pattern**
```javascript
Forms.v.combine(
  Forms.v.required(),
  Forms.v.pattern(/regex/, 'Message')
)
```

### **Pattern 4: Fast to Slow**
```javascript
Forms.v.combine(
  Forms.v.required(),        // Fast
  Forms.v.email(),           // Fast
  Forms.v.custom(asyncCheck) // Slow
)
```

---

## **Execution Flow**

```
Input: "abc"

Validator 1: required()         → null (passes)
Validator 2: minLength(8)       → "Min 8 chars" (FAILS - STOPS HERE)
Validator 3: pattern(/[A-Z]/)   → Not executed
Validator 4: pattern(/[0-9]/)   → Not executed

Result: "Min 8 chars"
```

```
Input: "Abcd1234"

Validator 1: required()         → null (passes)
Validator 2: minLength(8)       → null (passes)
Validator 3: pattern(/[A-Z]/)   → null (passes)
Validator 4: pattern(/[0-9]/)   → null (passes)

Result: null (all passed)
```

---

## **Key Takeaways**

1. **Sequential**: Runs validators in order, stops at first error
2. **Fails Fast**: Doesn't run remaining validators after error
3. **Order Matters**: Put cheap/fast validators first
4. **Any Validators**: Combine built-in, custom, or other combined validators
5. **Clean Code**: Better than nesting multiple validations
6. **Common Pattern**: Required + format + custom logic
7. **Performance**: Order impacts performance (fast → slow)
8. **Single Error**: Only shows one error at a time

---

## **Summary**

`Validators.combine()` creates a validator that runs multiple validators in sequence, returning the first error encountered or `null` if all pass. This allows you to build complex validation logic by composing simple validators together. The order of validators matters both for user experience (show most important errors first) and performance (run fast checks before slow async checks). It's the recommended way to apply multiple validation rules to a single field.
