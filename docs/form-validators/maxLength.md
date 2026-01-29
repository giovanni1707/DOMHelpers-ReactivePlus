# `Validators.maxLength()` - Maximum Length Validator

**Quick Start (30 seconds)**
```javascript
const form = Forms.createForm({ bio: '' }, {
  validators: {
    bio: Forms.v.maxLength(200)
  }
});

form.setValue('bio', 'a'.repeat(250));
form.validateField('bio');
console.log(form.errors.bio); // "Must be no more than 200 characters"

form.setValue('bio', 'Short bio');
form.validateField('bio');
console.log(form.errors.bio); // null (valid)
```

---

## **What is `Validators.maxLength()`?**

`Validators.maxLength()` is a **validator factory** that creates a validator function to ensure a string field does not exceed a maximum length.

**Key characteristics:**
- **String Length**: Checks `value.length <= max`
- **Optional Field**: Returns `null` if value is empty
- **Custom Messages**: Accepts custom error message
- **Default Message**: Auto-generates message with max value

---

## **Syntax**

```javascript
Validators.maxLength(max, message?)
```

### **Parameters**
- `max` (required) - Maximum length
  - **Type**: `number`
- `message` (optional) - Custom error message
  - **Type**: `string`
  - **Default**: `'Must be no more than {max} characters'`

### **Returns**
- **Type**: `(value) => string | null`

---

## **Examples**

### **Example 1: Bio Length Limit**
```javascript
const form = Forms.createForm({ bio: '' }, {
  validators: {
    bio: Forms.v.maxLength(500, 'Bio must be 500 characters or less')
  }
});
```

### **Example 2: Combined Min/Max**
```javascript
const form = Forms.createForm({ username: '' }, {
  validators: {
    username: Forms.v.combine(
      Forms.v.minLength(3),
      Forms.v.maxLength(20)
    )
  }
});
```

---

## **Key Takeaways**

1. **String Length**: Validates character count
2. **Optional**: Empty values pass validation
3. **Inclusive**: Max length is inclusive (<= not <)
4. **Common Use**: Text areas, short text fields
5. **Auto Message**: Generates helpful default message

---

## **Summary**

`Validators.maxLength()` creates a validator that ensures a string field does not exceed a maximum number of characters. It returns `null` for empty values or valid lengths, and an error message for strings that are too long.
