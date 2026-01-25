# `getRaw(state)` - Get Raw Object from State

**Quick Start (30 seconds)**
```javascript
const state = ReactiveUtils.state({
  count: 0,
  name: 'Test'
});

// Get raw (non-reactive) version
const raw = ReactiveUtils.getRaw(state);

console.log(isReactive(state)); // true
console.log(isReactive(raw)); // false

// Or use shorthand
const raw2 = getRaw(state);

// Or use $raw property
const raw3 = state.$raw;

// All are the same
console.log(raw === raw2); // true
console.log(raw === raw3); // true
```

---

## **What is `getRaw(state)`?**

`getRaw(state)` is a **namespace method** that gets the raw, non-reactive object from reactive state. It's an alias/wrapper for `toRaw()` that also tries the `$raw` property first.

**Key characteristics:**
- **Extract Raw**: Gets underlying object without reactivity
- **Multiple Methods**: Tries `$raw` property then `toRaw()`
- **Namespace Method**: Available as `ReactiveUtils.getRaw()`
- **Shorthand**: Also available as global `getRaw()`
- **Alias**: Essentially the same as `toRaw()`

---

## **Syntax**

```javascript
ReactiveUtils.getRaw(state)

// Or with shorthand
getRaw(state)

// Or property accessor
state.$raw
```

### **Parameters**
- **`state`** (Object): Reactive state object

### **Returns**
- **Type**: Raw object
- Non-reactive underlying object

---

## **How it works**

```javascript
ReactiveUtils.getRaw = function(state) {
  if (!state) return state;

  // Try $raw property first
  if (state.$raw) {
    return state.$raw;
  }

  // Fall back to toRaw function
  if (ReactiveUtils.toRaw) {
    return ReactiveUtils.toRaw(state);
  }

  return state;
};
```

**What happens:**
1. Returns early if state is null/undefined
2. Tries `state.$raw` property first
3. Falls back to `ReactiveUtils.toRaw()`
4. Returns state as-is if neither works

---

## **Examples**

### **Example 1: Basic Usage**
```javascript
const state = ReactiveUtils.state({ count: 0 });

// Three ways to get raw
const raw1 = ReactiveUtils.getRaw(state);
const raw2 = getRaw(state);
const raw3 = state.$raw;

console.log(raw1 === raw2); // true
console.log(raw2 === raw3); // true
```

### **Example 2: Serialization**
```javascript
const user = ReactiveUtils.state({
  name: 'John',
  email: 'john@example.com',
  age: 25
});

function saveUser(userState) {
  const raw = getRaw(userState);
  localStorage.setItem('user', JSON.stringify(raw));
}

saveUser(user);
```

### **Example 3: API Integration**
```javascript
const form = ReactiveUtils.form({
  title: '',
  description: '',
  tags: []
});

async function submitForm(formState) {
  const raw = getRaw(formState);

  const response = await fetch('/api/posts', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      title: raw.values.title,
      description: raw.values.description,
      tags: raw.values.tags
    })
  });

  return response.json();
}
```

### **Example 4: State Comparison**
```javascript
const state1 = ReactiveUtils.state({ id: 1, name: 'Test' });
const state2 = ReactiveUtils.state({ id: 1, name: 'Test' });

function statesEqual(a, b) {
  const rawA = getRaw(a);
  const rawB = getRaw(b);

  return JSON.stringify(rawA) === JSON.stringify(rawB);
}

console.log(statesEqual(state1, state2)); // true
```

### **Example 5: Deep Clone**
```javascript
const original = ReactiveUtils.state({
  user: { name: 'John', age: 25 },
  settings: { theme: 'dark' }
});

function cloneState(state) {
  const raw = getRaw(state);
  const cloned = JSON.parse(JSON.stringify(raw));
  return ReactiveUtils.state(cloned);
}

const copy = cloneState(original);

copy.user.name = 'Jane';
console.log(original.user.name); // 'John' (unchanged)
```

### **Example 6: Export State**
```javascript
const appState = ReactiveUtils.state({
  user: { name: 'John', role: 'admin' },
  settings: { theme: 'dark', language: 'en' },
  data: { items: [1, 2, 3] }
});

function exportState(state) {
  const raw = getRaw(state);

  return {
    version: '1.0',
    timestamp: Date.now(),
    state: raw
  };
}

const exported = exportState(appState);
console.log(exported);
```

### **Example 7: Validation**
```javascript
const state = ReactiveUtils.state({
  email: 'john@example.com',
  password: 'secret123'
});

function validateState(state) {
  const raw = getRaw(state);
  const errors = [];

  if (!raw.email.includes('@')) {
    errors.push('Invalid email');
  }

  if (raw.password.length < 8) {
    errors.push('Password too short');
  }

  return errors;
}

const errors = validateState(state);
console.log(errors);
```

### **Example 8: Debugging**
```javascript
const state = ReactiveUtils.state({
  count: 5,
  items: [1, 2, 3],
  nested: { value: 42 }
});

function debugState(state) {
  const raw = getRaw(state);

  console.log('Reactive state:', state);
  console.log('Raw object:', raw);
  console.log('JSON:', JSON.stringify(raw, null, 2));
}

debugState(state);
```

### **Example 9: Performance**
```javascript
const largeState = ReactiveUtils.state({
  items: Array.from({ length: 10000 }, (_, i) => ({
    id: i,
    value: Math.random()
  }))
});

function processItems(state) {
  const raw = getRaw(state);
  let sum = 0;

  // Fast: no reactivity overhead
  for (const item of raw.items) {
    sum += item.value;
  }

  return sum;
}

console.time('process');
const result = processItems(largeState);
console.timeEnd('process');
```

### **Example 10: State Migration**
```javascript
const oldState = ReactiveUtils.state({
  userName: 'John', // Old property name
  userEmail: 'john@example.com'
});

function migrateState(state, mapping) {
  const raw = getRaw(state);
  const migrated = {};

  Object.entries(mapping).forEach(([newKey, oldKey]) => {
    migrated[newKey] = raw[oldKey];
  });

  return ReactiveUtils.state(migrated);
}

const newState = migrateState(oldState, {
  name: 'userName',
  email: 'userEmail'
});

console.log(getRaw(newState));
// { name: 'John', email: 'john@example.com' }
```

---

## **Common Patterns**

### **Pattern 1: Get Raw**
```javascript
const raw = getRaw(state);
```

### **Pattern 2: Serialize**
```javascript
JSON.stringify(getRaw(state));
```

### **Pattern 3: Using Property**
```javascript
const raw = state.$raw;
```

### **Pattern 4: Namespace**
```javascript
const raw = ReactiveUtils.getRaw(state);
```

### **Pattern 5: Clone**
```javascript
const clone = { ...getRaw(state) };
```

---

## **getRaw() vs toRaw() vs $raw**

| Method | Type | Usage |
|--------|------|-------|
| `getRaw(state)` | Function | Namespace method, tries $raw first |
| `toRaw(value)` | Function | Direct utility function |
| `state.$raw` | Property | Property accessor |

All three return the same raw object:

```javascript
const state = ReactiveUtils.state({ count: 0 });

const a = getRaw(state);
const b = toRaw(state);
const c = state.$raw;

console.log(a === b); // true
console.log(b === c); // true
```

---

## **When to Use**

| Scenario | Use getRaw() |
|----------|-------------|
| JSON serialization | ✓ Yes |
| API requests | ✓ Yes |
| Local storage | ✓ Yes |
| State comparison | ✓ Yes |
| Deep cloning | ✓ Yes |
| Performance optimization | ✓ Yes |
| External libraries | ✓ Yes |
| Testing | ✓ Yes |

---

## **Best Practices**

1. **Prefer $raw property for brevity**
   ```javascript
   const raw = state.$raw;
   ```

2. **Use getRaw for consistency**
   ```javascript
   const raw = getRaw(state); // Clearer intent
   ```

3. **Serialization**
   ```javascript
   const json = JSON.stringify(getRaw(state));
   ```

4. **API integration**
   ```javascript
   await fetch('/api', {
     body: JSON.stringify(getRaw(state))
   });
   ```

5. **Testing**
   ```javascript
   const raw = getRaw(testState);
   expect(raw.value).toBe(expectedValue);
   ```

---

## **Key Takeaways**

1. **Namespace Method**: Part of ReactiveUtils namespace
2. **Alias**: Similar to `toRaw()` with fallback logic
3. **Multiple Ways**: Function or `$raw` property
4. **Extract Raw**: Gets non-reactive object
5. **Shorthand**: Available as global `getRaw()`
6. **Same Result**: `getRaw()`, `toRaw()`, `$raw` all equivalent
7. **Fallback**: Tries `$raw` first, then `toRaw()`
8. **Safe**: Returns value as-is if not reactive
9. **Use Cases**: Serialization, APIs, comparison, performance
10. **Preference**: Use `$raw` property for conciseness

---

## **Summary**

`getRaw(state)` is a namespace method that extracts the raw, non-reactive object from reactive state. It's available as `ReactiveUtils.getRaw()` and as a global `getRaw()` shorthand. The function first tries to access the `$raw` property on the state, then falls back to calling `toRaw()`. It returns the underlying object without reactivity, enabling operations that don't trigger effects. Use `getRaw()` for JSON serialization, API requests, local storage, state comparison, deep cloning, and performance optimization. The method is essentially an alias for `toRaw()` with additional fallback logic, and both produce the same result as accessing `state.$raw` directly. For most cases, using the `$raw` property directly is more concise, but `getRaw()` provides a consistent namespace method approach.
