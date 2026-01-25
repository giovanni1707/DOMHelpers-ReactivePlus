# `reactiveStorage(storageType, namespace)` - Reactive Storage Wrapper

**Quick Start (30 seconds)**
```javascript
// Create reactive storage wrapper
const storage = ReactiveUtils.reactiveStorage('localStorage', 'myApp');

// Use like regular storage, but reactive!
storage.set('user', { name: 'John', age: 30 });
storage.set('theme', 'dark');

// Reactive effects automatically track storage access
ReactiveUtils.effect(() => {
  const user = storage.get('user');
  console.log('User:', user); // Runs when user changes
});

// Changes trigger effects
storage.set('user', { name: 'Jane', age: 25 });
// Logs: "User: { name: 'Jane', age: 25 }"

// Cross-tab changes also trigger effects!
```

---

## **What is `reactiveStorage(storageType, namespace)`?**

`reactiveStorage()` is a **utility function** that creates a reactive wrapper around browser storage (localStorage or sessionStorage), making storage operations reactive so that effects and computed properties automatically re-run when storage values change.

**Key characteristics:**
- **Reactive**: Storage access triggers reactive updates
- **Cross-Tab**: Updates from other tabs trigger effects
- **Namespace**: Optional namespace for key organization
- **Storage API**: Familiar get/set/remove/keys/clear interface
- **Expiration**: Built-in expiration support
- **Type Safety**: JSON serialization/deserialization
- **Event Driven**: Uses storage events for cross-tab sync

---

## **Syntax**

```javascript
ReactiveUtils.reactiveStorage(storageType, namespace)
```

### **Parameters**
- **`storageType`** (string, optional): Storage type - `'localStorage'` or `'sessionStorage'` (default: `'localStorage'`)
- **`namespace`** (string, optional): Namespace prefix for keys (default: `''`)

### **Returns**
- **Type**: Reactive storage wrapper object

### **Methods**
- **`get(key)`** - Get value from storage
- **`set(key, value, options)`** - Set value in storage
- **`remove(key)`** - Remove key from storage
- **`has(key)`** - Check if key exists
- **`keys()`** - Get all keys (filtered by namespace)
- **`clear()`** - Clear all keys (in namespace)
- **`size()`** - Get number of keys (in namespace)

---

## **How it works**

```javascript
function reactiveStorage(storageType = 'localStorage', namespace = '') {
  const store = new StorageWrapper(storageType, namespace);

  // Create reactive state to track changes
  const reactiveState = ReactiveUtils.state({
    _version: 0,
    _keys: new Set(store.keys())
  });

  // Wrap storage with reactive proxy
  const proxy = new Proxy(store, {
    get(target, prop) {
      // Track access for reactivity
      if (['get', 'has', 'keys', 'size'].includes(prop)) {
        const _ = reactiveState._version; // Track dependency
      }
      return (...args) => {
        const result = target[prop](...args);
        return result;
      };
    }
  });

  // Listen for cross-tab changes
  window.addEventListener('storage', (event) => {
    if (event.storageArea === window[storageType]) {
      reactiveState._version++; // Notify reactive system
      reactiveState._keys = new Set(store.keys());
    }
  });

  return proxy;
}
```

**What happens:**
1. Creates StorageWrapper for storage operations
2. Creates reactive state to track storage version
3. Wraps storage with Proxy to intercept operations
4. Tracks storage access in reactive effects
5. Listens for storage events (cross-tab changes)
6. Increments version to trigger reactive updates

---

## **Examples**

### **Example 1: Basic Reactive Storage**
```javascript
const storage = ReactiveUtils.reactiveStorage('localStorage');

// Set values
storage.set('count', 0);
storage.set('name', 'App');

// Reactive effect
ReactiveUtils.effect(() => {
  const count = storage.get('count');
  document.getElementById('count').textContent = count;
});

// Update triggers effect
storage.set('count', 5);
// DOM updates automatically
```

### **Example 2: Namespaced Storage**
```javascript
const userStorage = ReactiveUtils.reactiveStorage('localStorage', 'user');
const appStorage = ReactiveUtils.reactiveStorage('localStorage', 'app');

// Keys are namespaced
userStorage.set('name', 'John');
appStorage.set('theme', 'dark');

// Stored as:
// "user:name" => "John"
// "app:theme" => "dark"

console.log(userStorage.keys()); // ['name']
console.log(appStorage.keys());  // ['theme']
```

### **Example 3: Reactive UI Updates**
```javascript
const prefs = ReactiveUtils.reactiveStorage('localStorage', 'prefs');

// Initialize
prefs.set('theme', 'light');
prefs.set('fontSize', 14);

// Reactive UI bindings
ReactiveUtils.effect(() => {
  const theme = prefs.get('theme');
  document.body.className = theme + '-theme';
});

ReactiveUtils.effect(() => {
  const fontSize = prefs.get('fontSize');
  document.body.style.fontSize = fontSize + 'px';
});

// UI updates automatically
prefs.set('theme', 'dark');
prefs.set('fontSize', 16);
```

### **Example 4: Cross-Tab Synchronization**
```javascript
const shared = ReactiveUtils.reactiveStorage('localStorage', 'shared');

// Effect in Tab 1
ReactiveUtils.effect(() => {
  const messages = shared.get('messages') || [];
  console.log('Messages:', messages);
  updateUI(messages);
});

// Change in Tab 2
shared.set('messages', ['Hello', 'World']);

// Tab 1 effect automatically runs!
```

### **Example 5: Check Existence**
```javascript
const cache = ReactiveUtils.reactiveStorage('localStorage', 'cache');

ReactiveUtils.effect(() => {
  if (cache.has('userData')) {
    const user = cache.get('userData');
    console.log('Using cached user data');
  } else {
    console.log('No cached data, fetching...');
    fetchUserData().then(data => {
      cache.set('userData', data);
    });
  }
});
```

### **Example 6: List All Keys**
```javascript
const storage = ReactiveUtils.reactiveStorage('localStorage', 'myApp');

storage.set('setting1', 'value1');
storage.set('setting2', 'value2');
storage.set('setting3', 'value3');

ReactiveUtils.effect(() => {
  const keys = storage.keys();
  console.log('Stored keys:', keys);
  // ['setting1', 'setting2', 'setting3']
});

storage.set('setting4', 'value4');
// Effect runs again
```

### **Example 7: Storage Size**
```javascript
const storage = ReactiveUtils.reactiveStorage('localStorage', 'data');

ReactiveUtils.effect(() => {
  const size = storage.size();
  document.getElementById('itemCount').textContent = `${size} items`;
});

storage.set('item1', { data: 'value1' });
storage.set('item2', { data: 'value2' });
// DOM updates to "2 items"

storage.remove('item1');
// DOM updates to "1 item"
```

### **Example 8: Clear Storage**
```javascript
const tempStorage = ReactiveUtils.reactiveStorage('sessionStorage', 'temp');

tempStorage.set('data1', 'value1');
tempStorage.set('data2', 'value2');

ReactiveUtils.effect(() => {
  const size = tempStorage.size();
  if (size === 0) {
    console.log('Storage is empty');
  } else {
    console.log(`Storage has ${size} items`);
  }
});

// Clear all
tempStorage.clear();
// Logs: "Storage is empty"
```

### **Example 9: Expiring Data**
```javascript
const cache = ReactiveUtils.reactiveStorage('localStorage', 'cache');

// Set with expiration (expires in 60 seconds)
cache.set('tempData', { value: 'expires soon' }, { expires: 60 });

ReactiveUtils.effect(() => {
  const data = cache.get('tempData');
  if (data) {
    console.log('Data still valid:', data);
  } else {
    console.log('Data expired or not found');
  }
});

// After 60 seconds, effect will log "Data expired or not found"
```

### **Example 10: Remove Items**
```javascript
const storage = ReactiveUtils.reactiveStorage('localStorage');

storage.set('temp1', 'value1');
storage.set('temp2', 'value2');

ReactiveUtils.effect(() => {
  if (storage.has('temp1')) {
    console.log('temp1 exists');
  } else {
    console.log('temp1 removed');
  }
});

// Remove triggers effect
storage.remove('temp1');
// Logs: "temp1 removed"
```

### **Example 11: Shopping Cart Sync**
```javascript
const cartStorage = ReactiveUtils.reactiveStorage('localStorage', 'cart');

const cartUI = {
  render() {
    ReactiveUtils.effect(() => {
      const items = cartStorage.get('items') || [];
      const total = cartStorage.get('total') || 0;

      document.getElementById('cartItems').innerHTML = items
        .map(item => `<li>${item.name} - $${item.price}</li>`)
        .join('');

      document.getElementById('cartTotal').textContent = `$${total}`;
    });
  }
};

cartUI.render();

// Add item
const currentItems = cartStorage.get('items') || [];
currentItems.push({ name: 'Product A', price: 29.99 });
cartStorage.set('items', currentItems);
cartStorage.set('total', currentItems.reduce((sum, i) => sum + i.price, 0));

// UI updates automatically
```

### **Example 12: Settings Panel**
```javascript
const settings = ReactiveUtils.reactiveStorage('localStorage', 'settings');

// Initialize defaults
if (!settings.has('theme')) settings.set('theme', 'light');
if (!settings.has('notifications')) settings.set('notifications', true);
if (!settings.has('language')) settings.set('language', 'en');

// Reactive settings panel
ReactiveUtils.effect(() => {
  const allKeys = settings.keys();
  const panel = document.getElementById('settingsPanel');

  panel.innerHTML = allKeys.map(key => {
    const value = settings.get(key);
    return `<div>${key}: ${JSON.stringify(value)}</div>`;
  }).join('');
});

// Changes reflect immediately
settings.set('theme', 'dark');
```

### **Example 13: Multi-Instance Storage**
```javascript
function createStorageInstance(namespace) {
  const storage = ReactiveUtils.reactiveStorage('localStorage', namespace);

  return {
    get(key) { return storage.get(key); },
    set(key, value) { storage.set(key, value); },
    clear() { storage.clear(); },
    size() { return storage.size(); }
  };
}

const user1 = createStorageInstance('user:1');
const user2 = createStorageInstance('user:2');

user1.set('name', 'Alice');
user2.set('name', 'Bob');

console.log(user1.get('name')); // 'Alice'
console.log(user2.get('name')); // 'Bob'
```

### **Example 14: Storage Monitor**
```javascript
const storage = ReactiveUtils.reactiveStorage('localStorage', 'app');

const monitor = ReactiveUtils.reactive({
  keyCount: 0,
  lastUpdate: null
});

ReactiveUtils.effect(() => {
  monitor.keyCount = storage.size();
  monitor.lastUpdate = new Date().toISOString();
});

// Monitor reactive to storage changes
ReactiveUtils.effect(() => {
  console.log(`Storage has ${monitor.keyCount} keys`);
  console.log(`Last updated: ${monitor.lastUpdate}`);
});

storage.set('key1', 'value1');
storage.set('key2', 'value2');
```

### **Example 15: Conditional Storage Operations**
```javascript
const cache = ReactiveUtils.reactiveStorage('localStorage', 'cache');

function getCachedOrFetch(key, fetchFn) {
  return new Promise((resolve) => {
    ReactiveUtils.effect(() => {
      if (cache.has(key)) {
        const cached = cache.get(key);
        console.log('Using cached value');
        resolve(cached);
      } else {
        console.log('Fetching fresh value');
        fetchFn().then(value => {
          cache.set(key, value, { expires: 300 }); // 5 min
          resolve(value);
        });
      }
    });
  });
}

// Usage
getCachedOrFetch('userData', () => fetch('/api/user').then(r => r.json()))
  .then(data => console.log('User data:', data));
```

---

## **Common Patterns**

### **Pattern 1: Create Reactive Storage**
```javascript
const storage = ReactiveUtils.reactiveStorage('localStorage');
```

### **Pattern 2: Namespaced Storage**
```javascript
const storage = ReactiveUtils.reactiveStorage('localStorage', 'myApp');
```

### **Pattern 3: Reactive Read**
```javascript
ReactiveUtils.effect(() => {
  const value = storage.get('key');
  console.log(value);
});
```

### **Pattern 4: Check and Load**
```javascript
if (storage.has('key')) {
  const value = storage.get('key');
}
```

### **Pattern 5: Iterate Keys**
```javascript
const keys = storage.keys();
keys.forEach(key => {
  console.log(key, storage.get(key));
});
```

---

## **Methods Reference**

### **`get(key)` - Get Value**
```javascript
const value = storage.get('key');
// Returns: value or null if not found/expired
```

### **`set(key, value, options)` - Set Value**
```javascript
storage.set('key', value, { expires: 3600 });
// Returns: void
// options: { expires: seconds }
```

### **`remove(key)` - Remove Key**
```javascript
storage.remove('key');
// Returns: void
```

### **`has(key)` - Check Existence**
```javascript
const exists = storage.has('key');
// Returns: boolean
```

### **`keys()` - Get All Keys**
```javascript
const keys = storage.keys();
// Returns: array of key names (filtered by namespace)
```

### **`clear()` - Clear Storage**
```javascript
storage.clear();
// Returns: void
// Clears all keys in namespace
```

### **`size()` - Get Key Count**
```javascript
const count = storage.size();
// Returns: number of keys (in namespace)
```

---

## **When to Use**

| Scenario | Use reactiveStorage() |
|----------|----------------------|
| UI needs to reflect storage changes | ✓ Yes |
| Cross-tab synchronization needed | ✓ Yes |
| Reactive effects depend on storage | ✓ Yes |
| Simple storage without reactivity | ✗ No (use native storage) |
| One-time read/write operations | ✗ No (use native storage) |
| Need automatic UI updates | ✓ Yes |

---

## **vs. autoSave()**

| Feature | `reactiveStorage()` | `autoSave()` |
|---------|-------------------|-------------|
| Purpose | Reactive storage wrapper | Auto-save reactive state |
| Storage Interface | get/set/remove/keys/clear | Automatic |
| Reactivity | Storage operations trigger effects | State changes trigger save |
| Methods Added | None (returns wrapper) | 7 methods ($save, $load, etc.) |
| Use Case | Manual storage with reactivity | Automatic state persistence |

```javascript
// reactiveStorage - manual operations
const storage = ReactiveUtils.reactiveStorage('localStorage');
storage.set('key', value);

// autoSave - automatic persistence
const state = ReactiveUtils.reactive({ key: value });
ReactiveUtils.autoSave(state, 'key');
state.key = newValue; // Auto-saves
```

---

## **Best Practices**

1. **Use namespaces for organization**
   ```javascript
   const storage = ReactiveUtils.reactiveStorage('localStorage', 'myApp');
   ```

2. **Check existence before reading**
   ```javascript
   if (storage.has('key')) {
     const value = storage.get('key');
   }
   ```

3. **Use sessionStorage for sensitive data**
   ```javascript
   const storage = ReactiveUtils.reactiveStorage('sessionStorage');
   ```

4. **Set expiration for temporary data**
   ```javascript
   storage.set('temp', data, { expires: 3600 });
   ```

5. **Clean up unused keys**
   ```javascript
   const oldKeys = storage.keys().filter(k => k.startsWith('temp'));
   oldKeys.forEach(k => storage.remove(k));
   ```

6. **Use separate namespaces for different features**
   ```javascript
   const userPrefs = ReactiveUtils.reactiveStorage('localStorage', 'prefs');
   const appCache = ReactiveUtils.reactiveStorage('localStorage', 'cache');
   ```

---

## **Key Takeaways**

1. **Reactive**: Storage operations trigger reactive updates
2. **Cross-Tab**: Changes in other tabs trigger effects
3. **Namespace**: Organize keys with namespace prefixes
4. **Storage API**: Familiar get/set/remove/keys/clear interface
5. **Expiration**: Built-in expiration support
6. **Event Driven**: Uses storage events for synchronization
7. **JSON**: Automatic serialization/deserialization
8. **Type Safe**: localStorage but reactive
9. **Manual**: Requires explicit get/set calls
10. **Wrapper**: Wraps native storage with reactivity

---

## **Summary**

`reactiveStorage(storageType, namespace)` creates a reactive wrapper around browser storage (localStorage or sessionStorage), making storage operations reactive so that effects and computed properties automatically re-run when storage values change. The function returns a storage wrapper object with familiar methods (get, set, remove, has, keys, clear, size) that trigger reactive updates when accessed within effects or computed properties. It supports cross-tab synchronization via storage events, meaning changes made in one browser tab automatically trigger reactive updates in other tabs. Optional namespace support allows organizing storage keys with prefixes, preventing key collisions between different features. Use reactiveStorage() when you need manual control over storage operations but want automatic UI updates when storage changes. It's ideal for settings panels, cross-tab data sharing, and cached data that needs to stay synchronized across the application. Unlike autoSave() which automatically persists reactive state, reactiveStorage() provides a reactive interface to storage itself, requiring explicit get/set operations but offering more fine-grained control.
