# `toRaw(value)` - Get Raw Non-Reactive Value

**Quick Start (30 seconds)**
```javascript
const state = ReactiveUtils.state({
  count: 0,
  user: { name: 'John', age: 25 }
});

// Get raw (non-reactive) version
const raw = toRaw(state);

console.log(isReactive(state)); // true
console.log(isReactive(raw)); // false

// Modifications to raw don't trigger reactivity
raw.count = 10; // No effects triggered

// Raw is the underlying object
console.log(raw === state); // false
console.log(raw.count); // 10
console.log(state.count); // 10 (same data, but state is reactive proxy)
```

---

## **What is `toRaw(value)`?**

`toRaw(value)` is a **utility function** that extracts the raw, non-reactive object from a reactive proxy.

**Key characteristics:**
- **Extract Raw**: Gets underlying object without reactivity
- **No Tracking**: Operations on raw don't trigger effects
- **Safe**: Returns value as-is if not reactive
- **Performance**: Direct access without proxy overhead
- **Namespace**: Available as `ReactiveUtils.toRaw()` and global `toRaw()`

---

## **Syntax**

```javascript
ReactiveUtils.toRaw(value)

// Or with shorthand
toRaw(value)
```

### **Parameters**
- **`value`** (any): Reactive proxy or any value

### **Returns**
- **Type**: Same type as input
- Raw object if value is reactive
- Original value if not reactive

---

## **How it works**

```javascript
const RAW = Symbol('raw');

function toRaw(v) {
  return (v && v[RAW]) || v;
}
```

**What happens:**
1. Checks if value has internal `RAW` symbol
2. Returns raw object if symbol exists
3. Returns original value otherwise
4. Raw object stored when proxy created

---

## **Examples**

### **Example 1: Basic Usage**
```javascript
const state = ReactiveUtils.state({ count: 0 });
const raw = toRaw(state);

console.log('State count:', state.count); // 0
console.log('Raw count:', raw.count); // 0

raw.count = 5;
console.log('Raw count:', raw.count); // 5
console.log('State count:', state.count); // 5 (same object)
```

### **Example 2: Avoid Triggering Effects**
```javascript
const state = ReactiveUtils.state({ count: 0 });

let effectCount = 0;
ReactiveUtils.effect(() => {
  effectCount++;
  console.log('Effect triggered:', state.count);
});

// This triggers the effect
state.count = 1;
console.log('Effect count:', effectCount); // 2

// This does NOT trigger the effect
const raw = toRaw(state);
raw.count = 2;
console.log('Effect count:', effectCount); // Still 2

// But state has the new value
console.log('State count:', state.count); // 2
```

### **Example 3: Performance Optimization**
```javascript
const largeState = ReactiveUtils.state({
  items: Array.from({ length: 10000 }, (_, i) => ({
    id: i,
    value: Math.random()
  }))
});

// Slow: Triggers reactivity for each access
function sumReactive(state) {
  let sum = 0;
  for (const item of state.items) {
    sum += item.value; // Each access tracked
  }
  return sum;
}

// Fast: No reactivity overhead
function sumRaw(state) {
  const raw = toRaw(state);
  let sum = 0;
  for (const item of raw.items) {
    sum += item.value; // Direct access
  }
  return sum;
}

console.time('reactive');
sumReactive(largeState);
console.timeEnd('reactive');

console.time('raw');
sumRaw(largeState);
console.timeEnd('raw'); // Typically faster
```

### **Example 4: Serialization**
```javascript
const state = ReactiveUtils.state({
  user: { name: 'John', age: 25 },
  settings: { theme: 'dark' }
});

// Convert to JSON
function serialize(state) {
  const raw = toRaw(state);
  return JSON.stringify(raw, null, 2);
}

const json = serialize(state);
console.log(json);
// {
//   "user": { "name": "John", "age": 25 },
//   "settings": { "theme": "dark" }
// }
```

### **Example 5: External Library Integration**
```javascript
const state = ReactiveUtils.state({
  data: [1, 2, 3, 4, 5]
});

// Some libraries don't work well with Proxies
function processWithExternalLib(state) {
  const raw = toRaw(state);

  // Library expects plain object
  return externalLibrary.process(raw.data);
}

// Or pass to API
async function saveToAPI(state) {
  const raw = toRaw(state);

  await fetch('/api/save', {
    method: 'POST',
    body: JSON.stringify(raw)
  });
}
```

### **Example 6: Deep Clone**
```javascript
const state = ReactiveUtils.state({
  user: { name: 'John', age: 25 },
  settings: { theme: 'dark' }
});

function deepClone(state) {
  const raw = toRaw(state);
  return JSON.parse(JSON.stringify(raw));
}

const clone = deepClone(state);

clone.user.name = 'Jane';
console.log(state.user.name); // 'John' (unchanged)
console.log(clone.user.name); // 'Jane'
```

### **Example 7: Comparison**
```javascript
const state1 = ReactiveUtils.state({ id: 1, name: 'Test' });
const state2 = ReactiveUtils.state({ id: 1, name: 'Test' });

// Reactive proxies are different objects
console.log(state1 === state2); // false

// But raw objects might be the same (if from same source)
const raw1 = toRaw(state1);
const raw2 = toRaw(state2);

// Deep comparison on raw objects
function deepEqual(a, b) {
  return JSON.stringify(toRaw(a)) === JSON.stringify(toRaw(b));
}

console.log(deepEqual(state1, state2)); // true
```

### **Example 8: Batch Operations Without Reactivity**
```javascript
const state = ReactiveUtils.state({
  items: []
});

ReactiveUtils.effect(() => {
  console.log('Items updated:', state.items.length);
});

// Add many items without triggering effect each time
function batchAdd(state, newItems) {
  const raw = toRaw(state);

  // Modify raw directly (no effects)
  newItems.forEach(item => {
    raw.items.push(item);
  });

  // Trigger effect once manually
  state.items = raw.items;
}

batchAdd(state, [1, 2, 3, 4, 5]);
// Effect triggered only once
```

### **Example 9: Debugging**
```javascript
const state = ReactiveUtils.state({
  count: 0,
  nested: { value: 42 }
});

function debugState(state) {
  const raw = toRaw(state);

  console.log('Reactive state:', state);
  console.log('Raw object:', raw);
  console.log('Is reactive:', isReactive(state));
  console.log('Raw is reactive:', isReactive(raw));
  console.log('Same object?', state === raw);
}

debugState(state);
```

### **Example 10: Local Storage**
```javascript
const state = ReactiveUtils.state({
  preferences: {
    theme: 'dark',
    fontSize: 14
  }
});

function saveToLocalStorage(state, key) {
  const raw = toRaw(state);
  localStorage.setItem(key, JSON.stringify(raw));
}

function loadFromLocalStorage(key) {
  const data = localStorage.getItem(key);
  if (data) {
    return ReactiveUtils.state(JSON.parse(data));
  }
  return null;
}

saveToLocalStorage(state, 'prefs');
const loaded = loadFromLocalStorage('prefs');
```

### **Example 11: Snapshot**
```javascript
const state = ReactiveUtils.state({ count: 0, name: 'Test' });

function takeSnapshot(state) {
  return { ...toRaw(state) };
}

const snapshot1 = takeSnapshot(state);

state.count = 10;
const snapshot2 = takeSnapshot(state);

console.log('Snapshot 1:', snapshot1); // { count: 0, name: 'Test' }
console.log('Snapshot 2:', snapshot2); // { count: 10, name: 'Test' }
```

### **Example 12: Testing**
```javascript
function setupTest() {
  const state = ReactiveUtils.state({ value: 0 });

  return {
    state,
    raw: toRaw(state),

    // Update without triggering effects (for setup)
    silentUpdate(key, value) {
      this.raw[key] = value;
    },

    // Update with reactivity (for testing)
    reactiveUpdate(key, value) {
      this.state[key] = value;
    }
  };
}

const test = setupTest();

test.silentUpdate('value', 10);
console.log(test.state.value); // 10 (updated but no effects)

test.reactiveUpdate('value', 20);
console.log(test.state.value); // 20 (updated with effects)
```

### **Example 13: Form Submission**
```javascript
const form = ReactiveUtils.form({
  email: '',
  password: '',
  rememberMe: false
});

async function submitForm(formState) {
  // Extract raw values for submission
  const raw = toRaw(formState);

  const payload = {
    email: raw.values.email,
    password: raw.values.password,
    rememberMe: raw.values.rememberMe
  };

  const response = await fetch('/api/login', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(payload)
  });

  return response.json();
}
```

### **Example 14: Utility Function**
```javascript
function isEqual(a, b) {
  const rawA = toRaw(a);
  const rawB = toRaw(b);

  return JSON.stringify(rawA) === JSON.stringify(rawB);
}

const state1 = ReactiveUtils.state({ count: 5 });
const state2 = ReactiveUtils.state({ count: 5 });
const plain = { count: 5 };

console.log(isEqual(state1, state2)); // true
console.log(isEqual(state1, plain)); // true
```

### **Example 15: Migration Helper**
```javascript
function migrateState(oldState, newStructure) {
  const raw = toRaw(oldState);

  // Create new state with migrated data
  const migrated = {};

  Object.keys(newStructure).forEach(key => {
    if (key in raw) {
      migrated[key] = raw[key];
    } else {
      migrated[key] = newStructure[key];
    }
  });

  return ReactiveUtils.state(migrated);
}

const old = ReactiveUtils.state({ count: 5 });
const newState = migrateState(old, {
  count: 0,
  name: 'Default',
  active: true
});

console.log(toRaw(newState));
// { count: 5, name: 'Default', active: true }
```

---

## **Common Patterns**

### **Pattern 1: Extract Raw**
```javascript
const raw = toRaw(state);
```

### **Pattern 2: Serialize**
```javascript
JSON.stringify(toRaw(state));
```

### **Pattern 3: Performance**
```javascript
const raw = toRaw(state);
// Heavy operations on raw
processData(raw);
```

### **Pattern 4: Clone**
```javascript
const clone = { ...toRaw(state) };
```

### **Pattern 5: Compare**
```javascript
JSON.stringify(toRaw(a)) === JSON.stringify(toRaw(b));
```

---

## **When to Use**

| Scenario | Use toRaw() |
|----------|------------|
| JSON serialization | ✓ Yes |
| External library integration | ✓ Yes |
| Performance-critical loops | ✓ Yes |
| Deep cloning | ✓ Yes |
| Comparison | ✓ Yes |
| Local storage | ✓ Yes |
| API requests | ✓ Yes |
| Debugging | ✓ Yes |
| Testing | ✓ Yes |

---

## **Best Practices**

1. **Use for serialization**
   ```javascript
   const json = JSON.stringify(toRaw(state));
   ```

2. **Performance optimization**
   ```javascript
   const raw = toRaw(largeState);
   // Heavy processing on raw
   ```

3. **External APIs**
   ```javascript
   await fetch('/api', {
     body: JSON.stringify(toRaw(state))
   });
   ```

4. **Avoid in effects**
   ```javascript
   // Don't do this in effects
   ReactiveUtils.effect(() => {
     const raw = toRaw(state); // Defeats purpose
     console.log(raw.count);
   });
   ```

5. **Testing setup**
   ```javascript
   const raw = toRaw(testState);
   raw.value = testValue; // Silent update
   ```

---

## **Key Takeaways**

1. **Extract Raw**: Gets non-reactive object from proxy
2. **No Tracking**: Operations don't trigger effects
3. **Performance**: Faster for heavy operations
4. **Serialization**: Essential for JSON, storage, APIs
5. **Safe**: Returns value as-is if not reactive
6. **Same Data**: Raw references same underlying object
7. **Namespace**: Available as `ReactiveUtils.toRaw()`
8. **Shorthand**: Available as global `toRaw()`
9. **Use Cases**: Serialization, APIs, performance, testing
10. **Caution**: Don't use in reactive contexts

---

## **Summary**

`toRaw(value)` is a utility function that extracts the raw, non-reactive object from a reactive proxy. It returns the underlying object without reactivity, allowing operations that don't trigger effects or dependency tracking. Use `toRaw()` for JSON serialization, external library integration, performance-critical operations, API requests, local storage, deep cloning, and testing. The function is safe to use with any value - it returns the raw object if the value is reactive, or the original value if not. Operations on the raw object modify the same data as the reactive proxy but don't trigger reactive effects. Available as both `ReactiveUtils.toRaw()` and global `toRaw()`. Essential for scenarios where you need to work with the data without reactivity overhead.
