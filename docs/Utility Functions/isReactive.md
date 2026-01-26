# `isReactive(value)` - Check if Value is Reactive

**Quick Start (30 seconds)**
```javascript
const state = ReactiveUtils.state({ count: 0 });
const plain = { count: 0 };

console.log(isReactive(state)); // true
console.log(isReactive(plain)); // false
console.log(isReactive(null)); // false
console.log(isReactive(42)); // false
console.log(isReactive('hello')); // false
```

---

## **What is `isReactive(value)`?**

`isReactive(value)` is a **utility function** that checks whether a value is a reactive proxy created by the reactivity system.

**Key characteristics:**
- **Type Check**: Returns boolean
- **Safe**: Works with any value type
- **Fast**: Simple symbol check
- **Reliable**: Internal symbol-based detection
- **Namespace**: Available as `ReactiveUtils.isReactive()` and global `isReactive()`

---

## **Syntax**

```javascript
ReactiveUtils.isReactive(value)

// Or with shorthand
isReactive(value)
```

### **Parameters**
- **`value`** (any): Value to check

### **Returns**
- **Type**: `boolean`
- `true` if value is reactive proxy
- `false` otherwise

---

## **How it works**

```javascript
const IS_REACTIVE = Symbol('reactive');

function isReactive(v) {
  return !!(v && v[IS_REACTIVE]);
}
```

**What happens:**
1. Checks if value exists and is truthy
2. Checks for internal `IS_REACTIVE` symbol
3. Returns boolean result
4. Symbol added when reactive proxy is created

---

## **Examples**

### **Example 1: Basic Check**
```javascript
const state = ReactiveUtils.state({ count: 0 });
const plain = { count: 0 };

if (isReactive(state)) {
  console.log('State is reactive');
}

if (!isReactive(plain)) {
  console.log('Plain object is not reactive');
}
```

### **Example 2: Type Guarding**
```javascript
function updateValue(obj, key, value) {
  if (isReactive(obj)) {
    // Safe to assign - will trigger reactivity
    obj[key] = value;
  } else {
    console.warn('Object is not reactive');
    // Make it reactive first
    const reactive = ReactiveUtils.state(obj);
    reactive[key] = value;
    return reactive;
  }

  return obj;
}
```

### **Example 3: Validation**
```javascript
function createComponent(state) {
  if (!isReactive(state)) {
    throw new Error('State must be reactive. Use ReactiveUtils.state()');
  }

  // Continue with component setup
  return {
    state,
    update(key, value) {
      this.state[key] = value;
    }
  };
}

// Usage
const state = ReactiveUtils.state({ count: 0 });
const component = createComponent(state); // Works

const plain = { count: 0 };
// createComponent(plain); // Throws error
```

### **Example 4: Debugging**
```javascript
const app = ReactiveUtils.reactive({ count: 0 })
  .computed({
    double: function() { return this.count * 2; }
  })
  .build();

console.log('Is reactive:', isReactive(app)); // true
console.log('Double is reactive:', isReactive(app.double)); // false (it's a number)
```

### **Example 5: Conditional Processing**
```javascript
function processData(data) {
  if (isReactive(data)) {
    console.log('Processing reactive data');

    // Access with reactivity tracking
    return ReactiveUtils.effect(() => {
      console.log('Data updated:', data.value);
    });
  } else {
    console.log('Processing plain data');

    // Direct access, no reactivity
    console.log('Data:', data.value);
  }
}
```

### **Example 6: Array Elements**
```javascript
const state = ReactiveUtils.state({
  items: [
    { id: 1, name: 'Item 1' },
    { id: 2, name: 'Item 2' }
  ]
});

console.log(isReactive(state)); // true
console.log(isReactive(state.items)); // true (deep reactivity)
console.log(isReactive(state.items[0])); // true (nested objects are reactive)
```

### **Example 7: Form Validation**
```javascript
function validateFormState(form) {
  const errors = [];

  if (!isReactive(form)) {
    errors.push('Form must be reactive');
  }

  if (!isReactive(form.values)) {
    errors.push('Form values must be reactive');
  }

  if (!isReactive(form.errors)) {
    errors.push('Form errors must be reactive');
  }

  return errors;
}

const form = ReactiveUtils.form({ email: '', password: '' });
const errors = validateFormState(form);
console.log(errors); // [] (all reactive)
```

### **Example 8: State Factory**
```javascript
function ensureReactive(data) {
  if (isReactive(data)) {
    return data;
  }

  console.log('Converting to reactive...');
  return ReactiveUtils.state(data);
}

const plain = { count: 0 };
const state1 = ensureReactive(plain); // Creates reactive
const state2 = ensureReactive(state1); // Returns same object

console.log(state1 === state2); // true
```

### **Example 9: API Response Handling**
```javascript
async function fetchAndStore(url, target) {
  const response = await fetch(url);
  const data = await response.json();

  if (isReactive(target)) {
    // Direct assignment to reactive state
    Object.assign(target, data);
  } else {
    // Return new reactive state
    return ReactiveUtils.state(data);
  }
}

const state = ReactiveUtils.state({ user: null });
await fetchAndStore('/api/user', state); // Updates state

const newState = await fetchAndStore('/api/user', {}); // Returns reactive
```

### **Example 10: Testing Helper**
```javascript
function expectReactive(value, message = 'Expected reactive value') {
  if (!isReactive(value)) {
    throw new Error(message);
  }
}

function expectNonReactive(value, message = 'Expected non-reactive value') {
  if (isReactive(value)) {
    throw new Error(message);
  }
}

// Test
const state = ReactiveUtils.state({ count: 0 });

expectReactive(state); // Passes
expectNonReactive({ count: 0 }); // Passes

// expectReactive({ count: 0 }); // Throws error
// expectNonReactive(state); // Throws error
```

### **Example 11: Property Check**
```javascript
const state = ReactiveUtils.state({
  user: { name: 'John', age: 25 },
  settings: { theme: 'dark' }
});

function inspectState(obj) {
  console.log('Inspecting state:');
  console.log('  Root:', isReactive(obj));

  Object.entries(obj).forEach(([key, value]) => {
    console.log(`  ${key}:`, isReactive(value));
  });
}

inspectState(state);
// Root: true
// user: true (deep reactivity)
// settings: true
```

### **Example 12: Mixin Integration**
```javascript
function withReactivity(obj) {
  return {
    ...obj,
    isReactive() {
      return isReactive(this);
    },
    ensureReactive() {
      if (!this.isReactive()) {
        console.warn('Converting to reactive');
        return ReactiveUtils.state(this);
      }
      return this;
    }
  };
}

const plain = withReactivity({ count: 0 });
console.log(plain.isReactive()); // false

const state = ReactiveUtils.state(withReactivity({ count: 0 }));
console.log(state.isReactive()); // true
```

### **Example 13: Performance Optimization**
```javascript
function optimizedUpdate(target, updates) {
  if (isReactive(target)) {
    // Use batch for reactive updates
    ReactiveUtils.batch(() => {
      Object.assign(target, updates);
    });
  } else {
    // Direct assignment for non-reactive
    Object.assign(target, updates);
  }
}
```

### **Example 14: Developer Tools**
```javascript
function getStateInfo(value) {
  return {
    type: typeof value,
    isReactive: isReactive(value),
    isNull: value === null,
    isUndefined: value === undefined,
    isObject: typeof value === 'object' && value !== null,
    constructor: value?.constructor?.name,
    keys: isReactive(value) ? Object.keys(value) : null
  };
}

const state = ReactiveUtils.state({ count: 0, name: 'Test' });

console.log(getStateInfo(state));
// {
//   type: 'object',
//   isReactive: true,
//   isNull: false,
//   isUndefined: false,
//   isObject: true,
//   constructor: 'Object',
//   keys: ['count', 'name']
// }
```

### **Example 15: State Transfer**
```javascript
function transferState(source, target) {
  if (!isReactive(source)) {
    throw new Error('Source must be reactive');
  }

  if (!isReactive(target)) {
    throw new Error('Target must be reactive');
  }

  // Safe to transfer
  Object.keys(source).forEach(key => {
    target[key] = source[key];
  });

  console.log('State transferred successfully');
}

const state1 = ReactiveUtils.state({ count: 5 });
const state2 = ReactiveUtils.state({ count: 0 });

transferState(state1, state2);
console.log(state2.count); // 5
```

---

## **Common Patterns**

### **Pattern 1: Type Guard**
```javascript
if (isReactive(value)) {
  // Handle reactive
} else {
  // Handle non-reactive
}
```

### **Pattern 2: Ensure Reactive**
```javascript
const state = isReactive(data) ? data : ReactiveUtils.state(data);
```

### **Pattern 3: Validation**
```javascript
if (!isReactive(obj)) {
  throw new Error('Expected reactive object');
}
```

### **Pattern 4: Conditional Update**
```javascript
if (isReactive(target)) {
  target.value = newValue;
}
```

### **Pattern 5: Debug Check**
```javascript
console.log('Is reactive:', isReactive(state));
```

---

## **Return Values**

| Input | Returns |
|-------|---------|
| Reactive state | `true` |
| Plain object | `false` |
| `null` | `false` |
| `undefined` | `false` |
| Primitive (number, string, etc.) | `false` |
| Array (reactive) | `true` |
| Array (plain) | `false` |

---

## **Use Cases**

| Scenario | Use isReactive() |
|----------|-----------------|
| Type checking | ✓ Yes |
| Validation | ✓ Yes |
| Conditional logic | ✓ Yes |
| Debugging | ✓ Yes |
| API integration | ✓ Yes |
| Testing | ✓ Yes |
| Performance optimization | ✓ Yes |

---

## **Best Practices**

1. **Use for validation**
   ```javascript
   if (!isReactive(state)) {
     throw new Error('State must be reactive');
   }
   ```

2. **Guard before reactive operations**
   ```javascript
   if (isReactive(obj)) {
     obj.value = newValue; // Safe
   }
   ```

3. **Factory pattern**
   ```javascript
   function ensureReactive(data) {
     return isReactive(data) ? data : ReactiveUtils.state(data);
   }
   ```

4. **Debugging**
   ```javascript
   console.log('Reactive:', isReactive(state));
   ```

5. **Testing assertions**
   ```javascript
   expect(isReactive(state)).toBe(true);
   ```

---

## **Key Takeaways**

1. **Type Check**: Returns boolean indicating if value is reactive
2. **Safe**: Works with any value type
3. **Symbol-Based**: Uses internal symbol for detection
4. **Fast**: Simple property check
5. **Validation**: Essential for type validation
6. **Debugging**: Useful for debugging reactivity issues
7. **Namespace**: Available as `ReactiveUtils.isReactive()`
8. **Shorthand**: Available as global `isReactive()`
9. **Reliable**: Accurate detection of reactive proxies
10. **Versatile**: Works with objects, arrays, nested values

---

## **Summary**

`isReactive(value)` is a utility function that checks whether a value is a reactive proxy created by the reactivity system. It returns `true` for reactive state created with `ReactiveUtils.state()`, `ReactiveUtils.reactive()`, or other reactive constructors, and `false` for plain objects, primitives, and null/undefined values. The function uses an internal symbol to reliably detect reactive proxies, making it fast and accurate. Use `isReactive()` for type validation, conditional logic, debugging, and ensuring proper reactive state handling in your application. It's available both as `ReactiveUtils.isReactive()` and as a global `isReactive()` shorthand function. The check is essential for building robust applications that properly handle both reactive and non-reactive data.
