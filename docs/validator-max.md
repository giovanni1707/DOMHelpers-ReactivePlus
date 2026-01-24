# `Validators.max()` - Maximum Value Validator

**Quick Start (30 seconds)**
```javascript
const form = Forms.createForm({ quantity: 0 }, {
  validators: {
    quantity: Forms.v.max(100)
  }
});

form.setValue('quantity', 150);
form.validateField('quantity');
console.log(form.errors.quantity); // "Must be no more than 100"

form.setValue('quantity', 50);
form.validateField('quantity');
console.log(form.errors.quantity); // null (valid)
```

---

## **What is `Validators.max()`?**

`Validators.max()` is a **validator factory** that creates a validator function to ensure a numeric field does not exceed a maximum value.

**Key characteristics:**
- **Numeric Comparison**: Checks `Number(value) <= max`
- **Optional Field**: Returns `null` if value is empty/null
- **Custom Messages**: Accepts custom error message
- **Auto Conversion**: Converts value to number for comparison

---

## **Syntax**

```javascript
Validators.max(max, message?)
```

### **Parameters**
- `max` (required) - Maximum value
  - **Type**: `number`
- `message` (optional) - Custom error message
  - **Type**: `string`
  - **Default**: `'Must be no more than {max}'`

### **Returns**
- **Type**: `(value) => string | null`

---

## **Examples**

### **Example 1: Quantity Limit**
```javascript
const form = Forms.createForm({ quantity: 0 }, {
  validators: {
    quantity: Forms.v.max(999, 'Maximum quantity is 999')
  }
});
```

### **Example 2: Combined Min/Max**
```javascript
const form = Forms.createForm({ age: 0 }, {
  validators: {
    age: Forms.v.combine(
      Forms.v.min(18, 'Must be 18+'),
      Forms.v.max(120, 'Invalid age')
    )
  }
});
```

---

## **Key Takeaways**

1. **Numeric Values**: For number inputs
2. **Optional**: Empty/null values pass
3. **Inclusive**: Max value is inclusive (<= not <)
4. **Auto Conversion**: Converts strings to numbers
5. **Common Use**: Quantity, age, rating fields

---

## **Summary**

`Validators.max()` creates a validator that ensures a numeric field does not exceed a maximum value. It returns `null` for empty/null values or valid numbers, and an error message for numbers that are too large.
