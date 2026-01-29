# `exists(state)` - Check if State Exists in Storage

**Quick Start (30 seconds)**
```javascript
const state = ReactiveUtils.reactive({
  data: null
});

ReactiveUtils.autoSave(state, 'myData');

// Check if data exists in storage
if (ReactiveUtils.exists(state)) {
  console.log('Data found in storage');
  ReactiveUtils.load(state);
} else {
  console.log('No saved data');
  initializeDefaults();
}
```

---

## **What is `exists(state)`?**

`exists(state)` is a **method added to reactive state objects** by `autoSave()` that checks if data exists in storage for the current state key.

**Key characteristics:**
- **Returns Boolean**: true if exists, false if not
- **Non-Destructive**: Doesn't modify state or storage
- **Checks Expiration**: Returns false if data expired
- **Fast Operation**: Only checks existence, doesn't load data
- **No Side Effects**: Read-only operation

---

## **Syntax**

```javascript
ReactiveUtils.exists(state)
```

### **Parameters**
- **`state`** (Object): Storage-enabled reactive state object

### **Returns**
- **Type**: `boolean`
- **`true`**: Data exists in storage and not expired
- **`false`**: No data or data expired

---

## **How it works**

```javascript
// When autoSave() is called
ReactiveUtils.autoSave(state, 'key', options);

// This method is added to state:
state.$exists = function() {
  return store.has(key);
};

// StorageWrapper.has() implementation:
StorageWrapper.prototype.has = function(key) {
  const item = storage.getItem(fullKey);
  if (!item) return false;

  // Check expiration
  const data = JSON.parse(item);
  if (data.expires && Date.now() > data.expires) {
    storage.removeItem(fullKey);
    return false;
  }

  return true;
};
```

**What happens:**
1. Checks if key exists in storage
2. If not found, returns false
3. If found, checks expiration
4. If expired, removes data and returns false
5. Otherwise returns true

---

## **Examples**

### **Example 1: Check Before Load**
```javascript
const settings = ReactiveUtils.reactive({
  theme: 'light',
  language: 'en'
});

ReactiveUtils.autoSave(settings, 'settings');

if (ReactiveUtils.exists(settings)) {
  ReactiveUtils.load(settings);
  console.log('Settings loaded');
} else {
  console.log('Using default settings');
}
```

### **Example 2: Conditional Initialization**
```javascript
const cache = ReactiveUtils.reactive({
  data: null,
  timestamp: null
});

ReactiveUtils.autoSave(cache, 'cache');

if (ReactiveUtils.exists(cache)) {
  ReactiveUtils.load(cache);
  console.log('Using cached data');
} else {
  fetchFreshData().then(data => {
    cache.data = data;
    cache.timestamp = Date.now();
  });
}
```

### **Example 3: User Onboarding**
```javascript
const userState = ReactiveUtils.reactive({
  hasSeenTutorial: false
});

ReactiveUtils.autoSave(userState, 'userState');

if (!ReactiveUtils.exists(userState)) {
  // First time user
  showWelcomeMessage();
  showTutorial();
}
```

### **Example 4: Session Check**
```javascript
const session = ReactiveUtils.reactive({
  token: null,
  userId: null
});

ReactiveUtils.autoSave(session, 'session');

if (ReactiveUtils.exists(session)) {
  ReactiveUtils.load(session);
  console.log('Existing session found');
  validateSession(session.token);
} else {
  console.log('No session');
  redirectToLogin();
}
```

### **Example 5: Cache Validation**
```javascript
const dataCache = ReactiveUtils.reactive({
  products: []
});

ReactiveUtils.autoSave(dataCache, 'products', {
  expires: 3600 // 1 hour
});

function loadProducts() {
  if (ReactiveUtils.exists(dataCache)) {
    ReactiveUtils.load(dataCache);
    console.log('Using cached products');
  } else {
    console.log('Cache expired or missing');
    fetchProducts();
  }
}
```

### **Example 6: Draft Detection**
```javascript
const draft = ReactiveUtils.reactive({
  title: '',
  content: ''
});

ReactiveUtils.autoSave(draft, 'draft');

window.addEventListener('load', () => {
  if (ReactiveUtils.exists(draft)) {
    if (confirm('Continue previous draft?')) {
      ReactiveUtils.load(draft);
    } else {
      ReactiveUtils.clear(draft);
    }
  }
});
```

### **Example 7: Multi-Profile Check**
```javascript
function profileExists(username) {
  const profile = ReactiveUtils.reactive({});
  ReactiveUtils.autoSave(profile, `profile:${username}`);

  return ReactiveUtils.exists(profile);
}

if (profileExists('john')) {
  console.log('Profile found');
} else {
  console.log('Profile not found');
}
```

### **Example 8: Conditional Save**
```javascript
const state = ReactiveUtils.reactive({
  data: []
});

ReactiveUtils.autoSave(state, 'state');

function saveIfNew() {
  if (!ReactiveUtils.exists(state)) {
    ReactiveUtils.save(state);
    console.log('Saved new state');
  } else {
    console.log('State already exists');
  }
}
```

### **Example 9: Data Migration**
```javascript
const oldData = ReactiveUtils.reactive({});
const newData = ReactiveUtils.reactive({});

ReactiveUtils.autoSave(oldData, 'data:v1');
ReactiveUtils.autoSave(newData, 'data:v2');

if (ReactiveUtils.exists(oldData) && !ReactiveUtils.exists(newData)) {
  console.log('Migrating data from v1 to v2');
  ReactiveUtils.load(oldData);
  migrateData(oldData, newData);
  ReactiveUtils.save(newData);
  ReactiveUtils.clear(oldData);
}
```

### **Example 10: Storage Status**
```javascript
const state = ReactiveUtils.reactive({
  value: null
});

ReactiveUtils.autoSave(state, 'state');

function getStorageStatus() {
  return {
    exists: ReactiveUtils.exists(state),
    info: ReactiveUtils.exists(state) ? ReactiveUtils.storageInfo(state) : null
  };
}

console.log(getStorageStatus());
```

---

## **Common Patterns**

### **Pattern 1: Check and Load**
```javascript
if (ReactiveUtils.exists(state)) {
  ReactiveUtils.load(state);
}
```

### **Pattern 2: Check and Initialize**
```javascript
if (!ReactiveUtils.exists(state)) {
  initializeDefaults();
}
```

### **Pattern 3: Conditional Actions**
```javascript
const exists = ReactiveUtils.exists(state);
if (exists) {
  handleExisting();
} else {
  handleNew();
}
```

### **Pattern 4: Check Before Clear**
```javascript
if (ReactiveUtils.exists(state)) {
  ReactiveUtils.clear(state);
}
```

### **Pattern 5: Existence Flag**
```javascript
const hasData = ReactiveUtils.exists(state);
console.log(`Has data: ${hasData}`);
```

---

## **When to Use**

| Scenario | Use exists() |
|----------|---------------|
| Check before load | ✓ Yes |
| Conditional initialization | ✓ Yes |
| First-time user detection | ✓ Yes |
| Cache validation | ✓ Yes |
| Session check | ✓ Yes |
| Data migration | ✓ Yes |
| Loading data | ✗ No (use load(state)) |
| Getting data | ✗ No (use storage.get()) |

---

## **vs. load(state)**

| Feature | `exists(state)` | `load(state)` |
|---------|------------|-----------|
| Checks existence | ✓ Yes | Implicitly |
| Loads data | ✗ No | ✓ Yes |
| Modifies state | ✗ No | ✓ Yes |
| Returns boolean | ✓ Yes | ✓ Yes |
| Performance | Fast | Slower |

```javascript
// Just check
if (ReactiveUtils.exists(state)) {
  // Data exists
}

// Check and load
if (ReactiveUtils.load(state)) {
  // Data was loaded
}
```

---

## **Return Value**

```javascript
const exists = ReactiveUtils.exists(state);

if (exists) {
  // Data exists in storage
  // And hasn't expired
} else {
  // No data found
  // Or data expired
}
```

---

## **Best Practices**

1. **Check before loading**
   ```javascript
   if (ReactiveUtils.exists(state)) {
     ReactiveUtils.load(state);
   } else {
     useDefaults();
   }
   ```

2. **Handle first-time users**
   ```javascript
   if (!ReactiveUtils.exists(state)) {
     showOnboarding();
   }
   ```

3. **Validate cache**
   ```javascript
   if (!ReactiveUtils.exists(cache)) {
     refreshCache();
   }
   ```

4. **Combine with user choice**
   ```javascript
   if (ReactiveUtils.exists(state) && confirm('Load saved data?')) {
     ReactiveUtils.load(state);
   }
   ```

5. **Check expiration**
   ```javascript
   // exists(state) automatically checks expiration
   if (ReactiveUtils.exists(cache)) {
     // Cache is valid
   } else {
     // Cache expired or missing
   }
   ```

6. **Use for migrations**
   ```javascript
   if (ReactiveUtils.exists(oldVersion) && !ReactiveUtils.exists(newVersion)) {
     migrateData();
   }
   ```

---

## **Key Takeaways**

1. **Returns Boolean**: true if exists, false if not
2. **Checks Expiration**: Returns false for expired data
3. **Non-Destructive**: Doesn't modify state or storage
4. **Fast**: Only checks existence, doesn't load
5. **No Side Effects**: Read-only operation
6. **Validation**: Good for conditional logic
7. **First-Time Detection**: Detect new users
8. **Cache Check**: Validate cache before use
9. **Session Validation**: Check for existing sessions
10. **Migration Helper**: Useful for data migrations

---

## **Summary**

`exists(state)` is a method added to reactive state objects by `autoSave()` that checks if data exists in storage for the current state key. When called, it returns true if data exists in storage and hasn't expired, or false if no data is found or if the data has expired. This is a non-destructive, read-only operation that doesn't modify the state object or storage. Use `exists(state)` before loading data to avoid unnecessary load operations, to detect first-time users who don't have saved data, to validate cache before using it, to check for existing sessions, or to implement conditional initialization logic. It's perfect for user onboarding (showing tutorials to new users), cache validation (checking if cached data is available), session management (checking for active sessions), and data migration (detecting old versions that need upgrading). The method automatically handles expiration checking, so expired data is treated as non-existent.
