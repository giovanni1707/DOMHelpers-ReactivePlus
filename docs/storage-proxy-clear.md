# `proxy.clear()` - Clear All Keys from Reactive Storage

**Quick Start (30 seconds)**
```javascript
// Create reactive storage
const storage = ReactiveUtils.reactiveStorage('localStorage', 'myApp');

// Set multiple values
storage.set('theme', 'dark');
storage.set('fontSize', 16);
storage.set('language', 'en');
storage.set('notifications', true);

console.log(storage.keys()); // ['theme', 'fontSize', 'language', 'notifications']

// Clear all keys in namespace (triggers reactivity)
storage.clear();

console.log(storage.keys()); // []

// Reactive effect responds to clear()
ReactiveUtils.effect(() => {
  const keys = storage.keys();
  if (keys.length === 0) {
    console.log('Storage is empty');
  } else {
    console.log(`Storage has ${keys.length} keys`);
  }
});
```

---

## **What is `proxy.clear()`?**

`proxy.clear()` is a **method on the reactive storage proxy** returned by `reactiveStorage()` that removes all keys in the namespace from browser storage and triggers reactive updates in any effects or computed properties tracking that storage.

**Key characteristics:**
- **Removes All Keys**: Deletes all keys in namespace
- **Triggers Reactivity**: Notifies all tracking effects
- **Returns Boolean**: true on success, false on failure
- **Namespace Scoped**: Only clears keys in the namespace
- **Permanent**: Cannot be undone
- **Cross-Tab**: Triggers storage events
- **Safe**: Never throws errors
- **Efficient**: Single operation for bulk removal

---

## **Syntax**

```javascript
proxy.clear()
```

### **Parameters**
None

### **Returns**
- **Type**: `boolean`
- **`true`**: Successfully cleared
- **`false`**: Clear failed (rare error case)

---

## **How it works**

```javascript
// When you call proxy.clear()
const storage = ReactiveUtils.reactiveStorage('localStorage', 'app');
storage.clear();

// Internally (WITH namespace):
// 1. Get all keys in namespace
const keys = storage.keys();

// 2. Remove each key
keys.forEach(key => storage.remove(key));

// 3. Triggers reactivity for each removal
// (version incremented, keys updated)

// Internally (WITHOUT namespace):
// 1. Clears entire storage
localStorage.clear();

// 2. Triggers reactivity once
reactiveState._version++;
reactiveState._keys = new Set();
```

**What happens:**
1. If namespace exists: Gets all keys, removes each one
2. If no namespace: Clears entire storage
3. Increments internal reactive version number
4. Updates internal keys Set to empty
5. Triggers all effects tracking this storage
6. Fires storage events for cross-tab sync
7. Returns success status

---

## **Examples**

### **Example 1: Basic Clear**
```javascript
const storage = ReactiveUtils.reactiveStorage('localStorage', 'app');

// Set some data
storage.set('user', { name: 'John' });
storage.set('theme', 'dark');
storage.set('language', 'en');

console.log(storage.keys().length); // 3

// Clear all
storage.clear();

console.log(storage.keys().length); // 0
console.log(storage.get('user'));   // null
console.log(storage.get('theme'));  // null
```

### **Example 2: Reactive Clear**
```javascript
const cache = ReactiveUtils.reactiveStorage('localStorage', 'cache');

// Reactive cache display
ReactiveUtils.effect(() => {
  const keys = cache.keys();

  if (keys.length === 0) {
    document.getElementById('cacheStatus').textContent = 'Cache is empty';
    document.getElementById('cacheList').innerHTML = '';
  } else {
    document.getElementById('cacheStatus').textContent =
      `Cache has ${keys.length} items`;

    const html = keys.map(key => {
      const value = cache.get(key);
      return `<li>${key}: ${JSON.stringify(value)}</li>`;
    }).join('');
    document.getElementById('cacheList').innerHTML = html;
  }
});

// Clear cache button
document.getElementById('clearCacheBtn').addEventListener('click', () => {
  cache.clear();
  showNotification('Cache cleared');
  // Effect runs, UI updates to show empty state
});
```

### **Example 3: Logout and Clear**
```javascript
const auth = ReactiveUtils.reactiveStorage('localStorage', 'auth');

function login(credentials) {
  auth.set('token', credentials.token);
  auth.set('userId', credentials.userId);
  auth.set('sessionId', generateSessionId());
  auth.set('loginTime', Date.now());
}

function logout() {
  // Clear all auth data
  auth.clear();
  console.log('All authentication data cleared');
  redirectToLogin();
}

// Reactive auth state
ReactiveUtils.effect(() => {
  const keys = auth.keys();
  const isLoggedIn = keys.length > 0 && auth.has('token');

  if (isLoggedIn) {
    showDashboard();
  } else {
    showLoginPage();
  }
});
```

### **Example 4: Reset to Defaults**
```javascript
const prefs = ReactiveUtils.reactiveStorage('localStorage', 'prefs');

const defaults = {
  theme: 'light',
  fontSize: 14,
  language: 'en',
  notifications: true
};

function resetSettings() {
  // Clear all current settings
  prefs.clear();

  // Set defaults
  Object.entries(defaults).forEach(([key, value]) => {
    prefs.set(key, value);
  });

  console.log('Settings reset to defaults');
}

document.getElementById('resetBtn').addEventListener('click', resetSettings);
```

### **Example 5: Privacy Mode**
```javascript
const browsing = ReactiveUtils.reactiveStorage('localStorage', 'browsing');

function enablePrivacyMode() {
  // Clear all browsing data
  browsing.clear();
  console.log('Privacy mode enabled - all browsing data cleared');
}

function disablePrivacyMode() {
  console.log('Privacy mode disabled');
}

// Reactive privacy indicator
ReactiveUtils.effect(() => {
  const hasData = browsing.keys().length > 0;
  document.getElementById('privacyMode').textContent =
    hasData ? 'Normal Mode' : 'Privacy Mode Active';
});
```

### **Example 6: Clear Cache on Error**
```javascript
const cache = ReactiveUtils.reactiveStorage('localStorage', 'cache');

async function fetchData() {
  try {
    // Try cache first
    if (cache.has('apiData')) {
      return cache.get('apiData');
    }

    // Fetch fresh data
    const response = await fetch('/api/data');
    const data = await response.json();

    cache.set('apiData', data, { expires: 3600 });
    return data;

  } catch (error) {
    console.error('Fetch error, clearing corrupted cache');
    cache.clear(); // Clear potentially corrupted cache
    throw error;
  }
}
```

### **Example 7: Namespace Isolation**
```javascript
const userStorage = ReactiveUtils.reactiveStorage('localStorage', 'user');
const appStorage = ReactiveUtils.reactiveStorage('localStorage', 'app');

// Set data in both namespaces
userStorage.set('name', 'John');
userStorage.set('email', 'john@example.com');

appStorage.set('theme', 'dark');
appStorage.set('language', 'en');

// Clear only user namespace
userStorage.clear();

console.log(userStorage.keys()); // [] (cleared)
console.log(appStorage.keys());  // ['theme', 'language'] (intact)
```

### **Example 8: Clear Temporary Data**
```javascript
const temp = ReactiveUtils.reactiveStorage('sessionStorage', 'temp');

// Store temporary wizard data
temp.set('step1', formData1);
temp.set('step2', formData2);
temp.set('step3', formData3);

function completeWizard() {
  const allData = {
    step1: temp.get('step1'),
    step2: temp.get('step2'),
    step3: temp.get('step3')
  };

  submitData(allData).then(() => {
    // Clear temporary wizard data
    temp.clear();
    console.log('Wizard completed, temporary data cleared');
  });
}
```

### **Example 9: Bulk Clear with Confirmation**
```javascript
const storage = ReactiveUtils.reactiveStorage('localStorage', 'app');

function clearAllData() {
  const count = storage.keys().length;

  if (count === 0) {
    alert('Storage is already empty');
    return;
  }

  if (confirm(`Clear all ${count} stored items?`)) {
    storage.clear();
    alert('All data cleared');
  }
}

document.getElementById('clearAllBtn').addEventListener('click', clearAllData);
```

### **Example 10: Periodic Cache Clear**
```javascript
const cache = ReactiveUtils.reactiveStorage('localStorage', 'cache');

// Reactive cache monitor
ReactiveUtils.effect(() => {
  const keys = cache.keys();
  console.log(`Cache has ${keys.length} items`);
});

// Clear cache every hour
setInterval(() => {
  const count = cache.keys().length;
  if (count > 0) {
    cache.clear();
    console.log(`Cleared ${count} cache items`);
  }
}, 3600000);

// Clear cache on page load if old
window.addEventListener('load', () => {
  const lastClear = localStorage.getItem('lastCacheClear');
  const oneDay = 24 * 60 * 60 * 1000;

  if (!lastClear || Date.now() - parseInt(lastClear) > oneDay) {
    cache.clear();
    localStorage.setItem('lastCacheClear', Date.now().toString());
  }
});
```

---

## **Common Patterns**

### **Pattern 1: Simple Clear**
```javascript
storage.clear();
```

### **Pattern 2: Clear and Reset**
```javascript
storage.clear();
Object.entries(defaults).forEach(([k, v]) => storage.set(k, v));
```

### **Pattern 3: Clear with Confirmation**
```javascript
if (confirm('Clear all data?')) {
  storage.clear();
}
```

### **Pattern 4: Clear and Notify**
```javascript
const count = storage.keys().length;
storage.clear();
console.log(`Cleared ${count} items`);
```

### **Pattern 5: Conditional Clear**
```javascript
if (shouldClear) {
  storage.clear();
}
```

---

## **When to Use**

| Scenario | Use proxy.clear() |
|----------|-------------------|
| Logout flow | ✓ Yes |
| Reset to defaults | ✓ Yes |
| Privacy mode | ✓ Yes |
| Clear cache | ✓ Yes |
| Error recovery | ✓ Yes |
| Bulk deletion | ✓ Yes |
| Remove one key | ✗ No (use remove()) |
| Different namespace | ✗ No (creates own instance) |

---

## **Namespace Behavior**

### **With Namespace (Scoped)**
```javascript
const app = ReactiveUtils.reactiveStorage('localStorage', 'app');
app.set('theme', 'dark');

app.clear();
// Only clears keys in 'app' namespace
// Other namespaces unaffected
```

### **Without Namespace (Global)**
```javascript
const storage = ReactiveUtils.reactiveStorage('localStorage');
storage.clear();
// ⚠️ Clears ENTIRE localStorage
// All namespaces affected
```

**Best Practice**: Always use namespaces to prevent accidental global clears.

---

## **vs. Removing All Keys Manually**

Both achieve the same result, but clear() is more efficient:

```javascript
// clear() - efficient, one operation
storage.clear();

// Manual - inefficient, multiple operations
storage.keys().forEach(key => storage.remove(key));
```

**Best Practice**: Use `clear()` for bulk removal.

---

## **Return Value**

### **Successful Clear**
```javascript
const result = storage.clear();
console.log(result); // true
```

### **Empty Storage**
```javascript
storage.clear(); // Still returns true
storage.clear(); // Returns true even if already empty
```

---

## **Best Practices**

1. **Always use namespaces**
   ```javascript
   // Good: scoped clear
   const storage = ReactiveUtils.reactiveStorage('localStorage', 'myApp');
   storage.clear();

   // Risky: global clear
   const storage = ReactiveUtils.reactiveStorage('localStorage');
   storage.clear(); // Clears everything!
   ```

2. **Confirm before clearing**
   ```javascript
   if (confirm('Clear all settings?')) {
     storage.clear();
   }
   ```

3. **Clear and reset**
   ```javascript
   storage.clear();
   initializeDefaults();
   ```

4. **Provide feedback**
   ```javascript
   const count = storage.keys().length;
   storage.clear();
   showNotification(`Cleared ${count} items`);
   ```

5. **Use for logout**
   ```javascript
   function logout() {
     authStorage.clear();
     sessionStorage.clear();
     redirectToLogin();
   }
   ```

6. **Periodic cleanup**
   ```javascript
   setInterval(() => {
     cacheStorage.clear();
   }, 3600000); // Clear hourly
   ```

---

## **vs. Native Storage**

| Feature | `proxy.clear()` | `localStorage.clear()` |
|---------|----------------|------------------------|
| Triggers reactivity | ✓ Yes | ✗ No |
| Namespace scoped | ✓ Yes | ✗ No (global) |
| Returns boolean | ✓ Yes | ✗ No (void) |
| Safe | ✓ Never throws | ✓ Never throws |
| Efficient | ✓ Yes | ✓ Yes |

```javascript
// proxy.clear() - reactive, scoped
const storage = ReactiveUtils.reactiveStorage('localStorage', 'app');
storage.clear();
// Only clears 'app' namespace, triggers effects

// localStorage.clear() - global
localStorage.clear();
// Clears EVERYTHING, no reactivity
```

---

## **Key Takeaways**

1. **Removes All Keys**: Deletes all keys in namespace
2. **Triggers Reactivity**: Notifies tracking effects
3. **Returns Boolean**: Indicates success
4. **Namespace Scoped**: Only affects namespace keys
5. **Permanent**: Cannot be undone
6. **Safe**: Never throws errors
7. **Efficient**: Single operation
8. **Cross-Tab**: Fires storage events
9. **Use Namespaces**: Prevent accidental global clears
10. **Reset Helper**: Perfect for reset-to-defaults

---

## **Summary**

`proxy.clear()` is a method on the reactive storage proxy returned by `reactiveStorage()` that removes all keys in the namespace from browser storage and triggers reactive updates in any effects or computed properties tracking that storage. When called with a namespace, it retrieves all keys in that namespace and removes them individually; when called without a namespace, it clears the entire storage. The method increments an internal reactive version number and triggers all effects that depend on this storage to re-run, making it perfect for implementing logout flows, reset-to-defaults functionality, privacy modes that clear history, cache clearing, error recovery, and any scenario where you need to remove all stored data with automatic reactive updates. Unlike native localStorage.clear() which clears the entire storage globally, proxy.clear() respects namespaces, only removing keys in the specified namespace and leaving other namespaces intact. Always use namespaces to prevent accidental global clears, and use proxy.clear() when you need efficient bulk removal with automatic reactive UI updates.
