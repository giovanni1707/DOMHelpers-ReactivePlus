# `Validators.min()` - Minimum Value Validator

**Quick Start (30 seconds)**
```javascript
const form = Forms.createForm({ age: 0 }, {
  validators: {
    age: Forms.v.min(18)
  }
});

form.setValue('age', 15);
form.validateField('age');
console.log(form.errors.age); // "Must be at least 18"

form.setValue('age', 21);
form.validateField('age');
console.log(form.errors.age); // null (valid)
```

---

## **What is `Validators.min()`?**

`Validators.min()` is a **validator factory** that creates a validator function to ensure a numeric field has a minimum value.

**Key characteristics:**
- **Numeric Comparison**: Checks `Number(value) >= min`
- **Optional Field**: Returns `null` if value is empty/null
- **Custom Messages**: Accepts custom error message
- **Auto Conversion**: Converts value to number for comparison

---

## **Syntax**

```javascript
Validators.min(min, message?)
```

### **Parameters**
- `min` (required) - Minimum value
  - **Type**: `number`
- `message` (optional) - Custom error message
  - **Type**: `string`
  - **Default**: `'Must be at least {min}'`

### **Returns**
- **Type**: `(value) => string | null`

---

## **Examples**

### **Example 1: Age Validation**
```javascript
const form = Forms.createForm({ age: 0 }, {
  validators: {
    age: Forms.v.min(18, 'You must be at least 18 years old')
  }
});
```

### **Example 2: Price Validation**
```javascript
const form = Forms.createForm({ price: 0 }, {
  validators: {
    price: Forms.v.combine(
      Forms.v.required('Price is required'),
      Forms.v.min(0.01, 'Price must be positive')
    )
  }
});
```

---

## **Key Takeaways**

1. **Numeric Values**: For number inputs
2. **Optional**: Empty/null values pass
3. **Inclusive**: Min value is inclusive (>= not >)
4. **Auto Conversion**: Converts strings to numbers
5. **Common Use**: Age, price, quantity fields

---

## **Summary**

`Validators.min()` creates a validator that ensures a numeric field has at least a minimum value. It returns `null` for empty/null values or valid numbers, and an error message for numbers that are too small.
