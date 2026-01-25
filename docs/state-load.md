# `load(state)` - Load State from Storage

**Quick Start (30 seconds)**
```javascript
const state = ReactiveUtils.reactive({
  count: 0,
  name: 'Default'
});

ReactiveUtils.autoSave(state, 'myState', {
  autoLoad: false // Don't auto-load on init
});

// Load manually when needed
ReactiveUtils.load(state);
console.log(state.count); // Loaded from storage
console.log(state.name);  // Loaded from storage
```

---

## **What is `load(state)`?**

`load()` is a **namespace method**

`load(state)` is a **method added to reactive state objects** by `autoSave()` that loads state from storage, overwriting the current state values.

**Key characteristics:**
- **Load from Storage**: Reads data from storage
- **Overwrites State**: Replaces current state with stored values
- **Respects onLoad**: Runs onLoad transformation if configured
- **Returns Boolean**: Returns true if loaded, false if not found
- **Handles Expiration**: Returns false if data expired
- **Manual Control**: Explicit load operation

---

## **Syntax**

```javascript
ReactiveUtils.load(state)
```

### **Parameters**
- None

### **Returns**
- **Type**: `boolean`
- **`true`**: Successfully loaded from storage
- **`false`**: No data found or data expired

---

## **How it works**

```javascript
// When autoSave() is called
ReactiveUtils.autoSave(state, 'key', options);

// This method is added to state:
state.$load = function() {
  const loaded = store.get(key);

  if (loaded === null) {
    return false; // Not found or expired
  }

  // Apply onLoad transform if configured
  const value = options.onLoad ? options.onLoad(loaded) : loaded;

  // Overwrite current state
  Object.assign(state, value);

  return true; // Successfully loaded
};
```

**What happens:**
1. Attempts to read from storage
2. Returns false if not found or expired
3. Applies onLoad transformation if configured
4. Overwrites current state with loaded values
5. Returns true if successful

---

## **Examples**

### **Example 1: Manual Load**
```javascript
const settings = ReactiveUtils.reactive({
  theme: 'light',
  language: 'en'
});

ReactiveUtils.autoSave(settings, 'settings', {
  autoLoad: false
});

// Load when needed
if (settings.$load()) {
  console.log('Settings loaded');
} else {
  console.log('No saved settings');
}
```

### **Example 2: Reload from Storage**
```javascript
const data = ReactiveUtils.reactive({
  items: []
});

ReactiveUtils.autoSave(data, 'data');

// Make local changes
data.items = [1, 2, 3];

// Discard changes and reload
function revertChanges() {
  if (data.$load()) {
    console.log('Changes reverted');
  }
}
```

### **Example 3: Load with Fallback**
```javascript
const config = ReactiveUtils.reactive({
  apiUrl: 'https://api.example.com',
  timeout: 5000
});

ReactiveUtils.autoSave(config, 'config', {
  autoLoad: false
});

// Try to load, use defaults if not found
if (!config.$load()) {
  console.log('Using default configuration');
}
```

### **Example 4: Conditional Load**
```javascript
const cache = ReactiveUtils.reactive({
  data: null,
  timestamp: null
});

ReactiveUtils.autoSave(cache, 'cache', {
  autoLoad: false
});

function loadIfNeeded() {
  if (shouldUseCache()) {
    if (cache.$load()) {
      console.log('Using cached data');
      return true;
    }
  }
  console.log('Fetching fresh data');
  return false;
}
```

### **Example 5: Refresh Data**
```javascript
const state = ReactiveUtils.reactive({
  users: [],
  lastUpdated: null
});

ReactiveUtils.autoSave(state, 'users');

document.getElementById('refreshBtn').addEventListener('click', () => {
  ReactiveUtils.load(state);
  showNotification('Data refreshed');
});
```

### **Example 6: Load Different Profiles**
```javascript
const profile = ReactiveUtils.reactive({
  name: '',
  email: '',
  preferences: {}
});

function switchProfile(profileName) {
  ReactiveUtils.autoSave(profile, `profile:${profileName}`, {
    autoLoad: false
  });

  if (profile.$load()) {
    console.log(`Loaded profile: ${profileName}`);
  } else {
    console.log(`No profile found: ${profileName}`);
    // Initialize with defaults
  }
}
```

### **Example 7: Sync from Storage**
```javascript
const sharedState = ReactiveUtils.reactive({
  messages: [],
  unreadCount: 0
});

ReactiveUtils.autoSave(sharedState, 'messages');

// Sync from another tab's changes
document.getElementById('syncBtn').addEventListener('click', () => {
  if (sharedState.$load()) {
    console.log('Synced with latest data');
  }
});
```

### **Example 8: Load with Validation**
```javascript
const data = ReactiveUtils.reactive({
  value: null
});

ReactiveUtils.autoSave(data, 'data', {
  autoLoad: false,
  onLoad: (loaded) => {
    if (!isValid(loaded)) {
      console.error('Invalid data in storage');
      return getDefaults();
    }
    return loaded;
  }
});

data.$load(); // Will validate and use defaults if invalid
```

### **Example 9: Reload on Error**
```javascript
const state = ReactiveUtils.reactive({
  config: null
});

ReactiveUtils.autoSave(state, 'config');

function handleError() {
  console.log('Error occurred, reloading config');
  if (!ReactiveUtils.load(state)) {
    // Fallback to defaults
    state.config = getDefaultConfig();
  }
}
```

### **Example 10: Check Before Load**
```javascript
const editor = ReactiveUtils.reactive({
  content: '',
  hasUnsavedChanges: false
});

ReactiveUtils.autoSave(editor, 'document');

function loadDocument() {
  if (editor.hasUnsavedChanges) {
    if (confirm('Discard unsaved changes?')) {
      editor.$load();
      editor.hasUnsavedChanges = false;
    }
  } else {
    editor.$load();
  }
}
```

---

## **Common Patterns**

### **Pattern 1: Load and Check**
```javascript
if (ReactiveUtils.load(state)) {
  console.log('Loaded successfully');
}
```

### **Pattern 2: Load with Fallback**
```javascript
if (!ReactiveUtils.load(state)) {
  // Use defaults
  state.value = defaultValue;
}
```

### **Pattern 3: Reload Data**
```javascript
button.addEventListener('click', () => {
  ReactiveUtils.load(state);
});
```

### **Pattern 4: Conditional Load**
```javascript
if (shouldLoad) {
  ReactiveUtils.load(state);
}
```

### **Pattern 5: Discard Changes**
```javascript
function revert() {
  ReactiveUtils.load(state); // Reload from storage
}
```

---

## **When to Use**

| Scenario | Use $load() |
|----------|-------------|
| Manual load control | ✓ Yes |
| Reload from storage | ✓ Yes |
| Discard changes | ✓ Yes |
| Conditional loading | ✓ Yes |
| Auto-load disabled | ✓ Yes |
| Sync from other tabs | ✓ Yes |
| Automatic loading | ✗ No (use autoLoad: true) |

---

## **vs. Auto-Load**

| Feature | `load(state)` | Auto-Load |
|---------|----------|-----------|
| Trigger | Manual | Automatic |
| Timing | On demand | On initialization |
| Control | Explicit | Implicit |
| Returns Value | Yes (boolean) | No |

```javascript
// Auto-load (automatic on init)
ReactiveUtils.autoSave(state, 'key', { autoLoad: true });
// State loaded automatically

// Manual load (explicit)
ReactiveUtils.autoSave(state, 'key', { autoLoad: false });
ReactiveUtils.load(state); // Load when needed
```

---

## **Return Value**

```javascript
const loaded = ReactiveUtils.load(state);

if (loaded) {
  // Data was found and loaded
  console.log('Loaded from storage');
} else {
  // No data found or expired
  console.log('Using defaults');
}
```

---

## **Best Practices**

1. **Check return value**
   ```javascript
   if (!ReactiveUtils.load(state)) {
     // Handle missing data
     initializeDefaults();
   }
   ```

2. **Confirm before discarding changes**
   ```javascript
   if (hasChanges && !confirm('Discard changes?')) {
     return;
   }
   ReactiveUtils.load(state);
   ```

3. **Use with validation**
   ```javascript
   ReactiveUtils.autoSave(state, 'key', {
     onLoad: (value) => {
       return isValid(value) ? value : defaults;
     }
   });
   ReactiveUtils.load(state);
   ```

4. **Provide feedback**
   ```javascript
   if (ReactiveUtils.load(state)) {
     showNotification('Loaded successfully');
   } else {
     showNotification('No saved data');
   }
   ```

5. **Load on demand**
   ```javascript
   function getData() {
     if (!state.loaded) {
       ReactiveUtils.load(state);
       state.loaded = true;
     }
     return state.data;
   }
   ```

---

## **Key Takeaways**

1. **Manual**: Explicitly loads from storage
2. **Overwrites**: Replaces current state values
3. **Returns Boolean**: true if loaded, false if not found
4. **Respects onLoad**: Runs onLoad transformation
5. **Handles Expiration**: Returns false if expired
6. **Discards Changes**: Overwrites unsaved changes
7. **On Demand**: Load when needed, not automatically
8. **Sync Tool**: Can sync from other tabs
9. **Validation**: Use onLoad for validation
10. **Fallback**: Check return value for fallback logic

---

## **Summary**

`load(state)` is a method added to reactive state objects by `autoSave()` that loads state from storage, overwriting the current state values. When called, it attempts to read data from the configured storage (localStorage or sessionStorage), applies any onLoad transformation if configured, and overwrites the current state with the loaded values. The method returns true if data was successfully loaded, or false if no data was found or if the data had expired. Use `load(state)` when you need explicit control over load timing, such as when auto-load is disabled, when you want to reload/refresh data from storage, when discarding local changes, or when implementing profile switching. It's perfect for implementing "revert changes" functionality, manual sync buttons, conditional loading based on user actions, or multi-profile systems where different data sets are loaded on demand. The return value makes it easy to implement fallback logic when no saved data exists.
