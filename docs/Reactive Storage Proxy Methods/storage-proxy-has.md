# `proxy.has(key)` - Check if Key Exists in Reactive Storage

**Quick Start (30 seconds)**
```javascript
// Create reactive storage
const storage = ReactiveUtils.reactiveStorage('localStorage', 'myApp');

// Set some values
storage.set('theme', 'dark');
storage.set('user', { name: 'John' });

// Check existence (tracks dependency)
console.log(storage.has('theme')); // true
console.log(storage.has('user'));  // true
console.log(storage.has('missing')); // false

// Reactive effect tracks has()
ReactiveUtils.effect(() => {
  if (storage.has('user')) {
    console.log('User data exists');
    const user = storage.get('user');
    displayUser(user);
  } else {
    console.log('No user data');
    showLoginPrompt();
  }
});
```

---

## **What is `proxy.has(key)`?**

`proxy.has()` is a **method on the reactive storage proxy** returned by `reactiveStorage()` that checks whether a key exists in browser storage and **tracks the access as a reactive dependency** so that effects re-run when the key's existence changes.

**Key characteristics:**
- **Checks Existence**: Returns true if key exists, false otherwise
- **Tracks Dependency**: Effects re-run when existence changes
- **Returns Boolean**: Always returns true or false
- **Reactive**: Creates dependency in tracking contexts
- **Fast**: Only checks existence, doesn't load value
- **Safe**: Never throws errors
- **Namespaced**: Handles namespace automatically

---

## **Syntax**

```javascript
proxy.has(key)
```

### **Parameters**
- **`key`** (string): Storage key to check (without namespace prefix)

### **Returns**
- **Type**: `boolean`
- **`true`**: Key exists in storage
- **`false`**: Key does not exist

---

## **How it works**

```javascript
// When you call proxy.has() inside an effect
const storage = ReactiveUtils.reactiveStorage('localStorage', 'app');

ReactiveUtils.effect(() => {
  const exists = storage.has('user');
  console.log(exists);
});

// Internally:
// 1. Proxy intercepts has() call
// 2. Tracks reactive dependency (_version)
const _ = reactiveState._version;

// 3. Constructs full key
const fullKey = namespace ? `${namespace}:${key}` : key;

// 4. Checks storage
const exists = localStorage.getItem(fullKey) !== null;

// 5. Returns boolean
return exists;
```

**What happens:**
1. Proxy intercepts the has() call
2. Accesses reactive state _version (creates dependency)
3. Constructs full storage key with namespace
4. Checks if item exists in browser storage
5. Returns true if exists, false otherwise
6. Effect now "subscribes" to existence changes

---

## **Examples**

### **Example 1: Basic Existence Check**
```javascript
const storage = ReactiveUtils.reactiveStorage('localStorage');

// Set some values
storage.set('name', 'Alice');
storage.set('age', 30);

// Check existence
console.log(storage.has('name'));    // true
console.log(storage.has('age'));     // true
console.log(storage.has('missing')); // false
```

### **Example 2: Conditional Loading**
```javascript
const cache = ReactiveUtils.reactiveStorage('localStorage', 'cache');

// Check before loading
if (cache.has('userData')) {
  console.log('Using cached data');
  const data = cache.get('userData');
  displayData(data);
} else {
  console.log('Fetching fresh data');
  fetchUserData().then(data => {
    cache.set('userData', data);
    displayData(data);
  });
}
```

### **Example 3: Reactive UI State**
```javascript
const auth = ReactiveUtils.reactiveStorage('localStorage', 'auth');

// Reactive login state
ReactiveUtils.effect(() => {
  const isLoggedIn = auth.has('token');

  if (isLoggedIn) {
    document.getElementById('loginBtn').style.display = 'none';
    document.getElementById('logoutBtn').style.display = 'block';
    document.getElementById('userPanel').style.display = 'block';
  } else {
    document.getElementById('loginBtn').style.display = 'block';
    document.getElementById('logoutBtn').style.display = 'none';
    document.getElementById('userPanel').style.display = 'none';
  }
});

// Login/logout updates UI automatically
function login(token) {
  auth.set('token', token);
  // Effect runs, UI updates
}

function logout() {
  auth.remove('token');
  // Effect runs, UI updates
}
```

### **Example 4: First-Time User Detection**
```javascript
const userState = ReactiveUtils.reactiveStorage('localStorage', 'user');

// Check if first-time user
if (!userState.has('hasSeenTutorial')) {
  showWelcomeTutorial();
  userState.set('hasSeenTutorial', true);
} else {
  console.log('Returning user');
}
```

### **Example 5: Feature Flag Check**
```javascript
const features = ReactiveUtils.reactiveStorage('localStorage', 'features');

// Reactive feature rendering
ReactiveUtils.effect(() => {
  const hasNewFeature = features.has('betaFeature');

  if (hasNewFeature) {
    document.getElementById('betaFeature').style.display = 'block';
  } else {
    document.getElementById('betaFeature').style.display = 'none';
  }
});

// Enable feature
function enableBetaFeature() {
  features.set('betaFeature', true);
}

// Disable feature
function disableBetaFeature() {
  features.remove('betaFeature');
}
```

### **Example 6: Notification Indicator**
```javascript
const notifications = ReactiveUtils.reactiveStorage('localStorage', 'notifications');

// Reactive notification badge
ReactiveUtils.effect(() => {
  const hasNotifications = notifications.has('unread');
  const badge = document.getElementById('notificationBadge');

  if (hasNotifications) {
    const unread = notifications.get('unread');
    badge.textContent = unread.length;
    badge.style.display = 'block';
  } else {
    badge.style.display = 'none';
  }
});
```

### **Example 7: Draft Detection**
```javascript
const drafts = ReactiveUtils.reactiveStorage('sessionStorage', 'drafts');

window.addEventListener('load', () => {
  if (drafts.has('formDraft')) {
    if (confirm('Continue previous draft?')) {
      const draft = drafts.get('formDraft');
      loadFormData(draft);
    } else {
      drafts.remove('formDraft');
    }
  }
});
```

### **Example 8: Settings Validation**
```javascript
const prefs = ReactiveUtils.reactiveStorage('localStorage', 'prefs');

// Initialize with defaults if missing
const defaults = {
  theme: 'light',
  fontSize: 14,
  notifications: true
};

Object.entries(defaults).forEach(([key, value]) => {
  if (!prefs.has(key)) {
    prefs.set(key, value);
    console.log(`Initialized ${key} with default:`, value);
  }
});
```

### **Example 9: Cache Availability**
```javascript
const cache = ReactiveUtils.reactiveStorage('localStorage', 'cache');

function getCachedOrFetch(key, fetchFn) {
  return new Promise((resolve) => {
    if (cache.has(key)) {
      console.log('Cache hit:', key);
      resolve(cache.get(key));
    } else {
      console.log('Cache miss:', key);
      fetchFn().then(data => {
        cache.set(key, data, { expires: 3600 });
        resolve(data);
      });
    }
  });
}

// Usage
getCachedOrFetch('products', () => fetch('/api/products').then(r => r.json()))
  .then(products => displayProducts(products));
```

### **Example 10: Multi-Profile Check**
```javascript
const profiles = ReactiveUtils.reactiveStorage('localStorage', 'profiles');

function switchProfile(profileName) {
  if (profiles.has(profileName)) {
    const profile = profiles.get(profileName);
    loadProfile(profile);
    console.log(`Loaded profile: ${profileName}`);
  } else {
    console.log(`Profile not found: ${profileName}`);
    createNewProfile(profileName);
  }
}

// List available profiles
function listProfiles() {
  const keys = profiles.keys();
  return keys.filter(key => profiles.has(key));
}
```

---

## **Common Patterns**

### **Pattern 1: Simple Check**
```javascript
if (storage.has('key')) {
  // Key exists
}
```

### **Pattern 2: Check and Load**
```javascript
if (storage.has('key')) {
  const value = storage.get('key');
  process(value);
}
```

### **Pattern 3: Check with Fallback**
```javascript
if (!storage.has('key')) {
  storage.set('key', defaultValue);
}
```

### **Pattern 4: Reactive Existence**
```javascript
ReactiveUtils.effect(() => {
  const exists = storage.has('key');
  updateUI(exists);
});
```

### **Pattern 5: Multiple Checks**
```javascript
const hasAll = ['key1', 'key2', 'key3'].every(key => storage.has(key));
```

---

## **When to Use**

| Scenario | Use proxy.has() |
|----------|-----------------|
| Check before loading | ✓ Yes |
| First-time user detection | ✓ Yes |
| Feature flag checking | ✓ Yes |
| Cache validation | ✓ Yes |
| Conditional initialization | ✓ Yes |
| Draft detection | ✓ Yes |
| Login state | ✓ Yes |
| Getting actual value | ✗ No (use get()) |

---

## **vs. get()**

Both can check existence, but with different approaches:

```javascript
// has() - just checks existence
if (storage.has('key')) {
  // Key exists (fast)
}

// get() - loads and checks
const value = storage.get('key');
if (value !== null) {
  // Key exists and has value (slower)
}
```

**Best Practice**: Use `has()` when you only need to check existence, use `get()` when you need the value.

---

## **Reactivity Details**

### **Creates Dependency**
```javascript
ReactiveUtils.effect(() => {
  const exists = storage.has('key');
  // Effect depends on key existence
  // Re-runs when key added/removed
});
```

### **Responds to set()**
```javascript
ReactiveUtils.effect(() => {
  if (storage.has('notification')) {
    showAlert();
  }
});

storage.set('notification', 'New message');
// Effect runs, alert shown
```

### **Responds to remove()**
```javascript
ReactiveUtils.effect(() => {
  if (storage.has('draft')) {
    showDraftIndicator();
  } else {
    hideDraftIndicator();
  }
});

storage.remove('draft');
// Effect runs, indicator hidden
```

---

## **Return Values**

### **Key Exists**
```javascript
storage.set('name', 'Alice');
console.log(storage.has('name')); // true
```

### **Key Doesn't Exist**
```javascript
console.log(storage.has('nonexistent')); // false
```

### **After Removal**
```javascript
storage.set('temp', 'value');
console.log(storage.has('temp')); // true

storage.remove('temp');
console.log(storage.has('temp')); // false
```

### **Empty Value**
```javascript
storage.set('empty', '');
console.log(storage.has('empty')); // true

storage.set('nullValue', null);
console.log(storage.has('nullValue')); // true
```

---

## **Best Practices**

1. **Check before loading**
   ```javascript
   if (storage.has('config')) {
     const config = storage.get('config');
     applyConfig(config);
   }
   ```

2. **Initialize missing keys**
   ```javascript
   if (!storage.has('settings')) {
     storage.set('settings', defaultSettings);
   }
   ```

3. **Use for existence logic**
   ```javascript
   ReactiveUtils.effect(() => {
     const isLoggedIn = storage.has('authToken');
     updateUIForAuthState(isLoggedIn);
   });
   ```

4. **Validate data presence**
   ```javascript
   const hasRequiredData = ['user', 'prefs', 'session']
     .every(key => storage.has(key));

   if (!hasRequiredData) {
     initializeApp();
   }
   ```

5. **Check before expensive operations**
   ```javascript
   if (storage.has('cache')) {
     return storage.get('cache');
   } else {
     return expensiveComputation();
   }
   ```

6. **Use for feature flags**
   ```javascript
   if (storage.has('experimentalFeature')) {
     enableExperimentalUI();
   }
   ```

---

## **vs. Native Storage Check**

| Feature | `proxy.has()` | `localStorage.getItem() !== null` |
|---------|--------------|----------------------------------|
| Tracks reactivity | ✓ Yes | ✗ No |
| Returns boolean | ✓ Yes | ✓ Yes |
| Namespace | ✓ Automatic | ✗ Manual |
| Performance | ✓ Fast | ✓ Fast |
| Expiration aware | ⚠ No | ⚠ No |

```javascript
// proxy.has() - reactive
const storage = ReactiveUtils.reactiveStorage('localStorage');
if (storage.has('key')) {
  // Tracks dependency, handles namespace
}

// localStorage - manual
if (localStorage.getItem('key') !== null) {
  // No reactivity, manual namespace
}
```

---

## **Key Takeaways**

1. **Checks Existence**: Returns true/false for key presence
2. **Tracks Dependency**: Creates reactive dependency
3. **Fast**: Doesn't load value, just checks
4. **Reactive**: Effects re-run on existence changes
5. **Boolean**: Always returns true or false
6. **Safe**: Never throws errors
7. **Namespaced**: Handles namespace automatically
8. **Responds to Changes**: Reacts to set() and remove()
9. **Initialization**: Good for first-time checks
10. **Conditional Logic**: Perfect for if/else logic

---

## **Summary**

`proxy.has(key)` is a method on the reactive storage proxy returned by `reactiveStorage()` that checks whether a key exists in browser storage and tracks the access as a reactive dependency, causing effects and computed properties to automatically re-run when the key's existence changes (when it's added via set() or removed via remove()). When called, it constructs the namespaced storage key and checks if the item exists in localStorage or sessionStorage, returning true if it exists or false if it doesn't. The key feature is that when used inside an effect or computed property, has() creates a reactive dependency on the key's existence, so the effect automatically re-runs whenever the key is added or removed. This makes it perfect for conditional UI rendering based on data availability, first-time user detection, feature flag checking, cache validation, authentication state, draft detection, and any scenario where UI or logic needs to react to whether data exists. Unlike get() which loads the value, has() only checks existence, making it faster when you don't need the actual value. Use proxy.has() when you need existence-based conditional logic with automatic reactive updates.
