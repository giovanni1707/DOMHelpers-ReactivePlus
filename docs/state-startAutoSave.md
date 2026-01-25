# `$startAutoSave()` - Start Automatic Saving

**Quick Start (30 seconds)**
```javascript
const state = ReactiveUtils.reactive({
  count: 0,
  data: []
});

ReactiveUtils.autoSave(state, 'myState', {
  autoSave: false // Start with auto-save disabled
});

// Changes don't auto-save
state.count = 1; // Not saved

// Enable auto-save
state.$startAutoSave();

// Now changes auto-save
state.count = 2; // Auto-saved
state.data = [1, 2, 3]; // Auto-saved
```

---

## **What is `$startAutoSave()`?**

`$startAutoSave()` is a **method added to reactive state objects** by `autoSave()` that enables or resumes automatic saving after it has been disabled.

**Key characteristics:**
- **Enables Auto-Save**: Activates automatic saving
- **Resumes Watching**: Re-establishes reactive effects
- **Immediate Effect**: Next change will auto-save
- **No Parameters**: Takes no arguments
- **Returns Void**: No return value
- **Idempotent**: Safe to call multiple times

---

## **Syntax**

```javascript
state.$startAutoSave()
```

### **Parameters**
- None

### **Returns**
- **Type**: `void`

---

## **How it works**

```javascript
// When autoSave() is called
ReactiveUtils.autoSave(state, 'key', options);

// This method is added to state:
state.$startAutoSave = function() {
  if (autoSaveEnabled) {
    return; // Already enabled
  }

  autoSaveEnabled = true;

  // Re-establish reactive effect
  effectCleanup = effect(() => {
    const value = toRaw(state);

    clearTimeout(saveTimeout);
    saveTimeout = setTimeout(() => {
      const toSave = options.onSave ? options.onSave(value) : value;
      store.set(key, toSave, { expires: options.expires });
    }, options.debounce || 0);
  });
};
```

**What happens:**
1. Checks if auto-save is already enabled
2. Sets internal flag to enable auto-save
3. Creates reactive effect to watch for changes
4. Future changes will trigger automatic saves
5. Respects debounce settings

---

## **Examples**

### **Example 1: Resume After Batch Updates**
```javascript
const data = ReactiveUtils.reactive({
  items: [],
  count: 0
});

ReactiveUtils.autoSave(data, 'data');

// Stop for batch updates
data.$stopAutoSave();

data.items = [1, 2, 3];
data.count = 3;
data.$save();

// Resume auto-save
data.$startAutoSave();

// Future changes auto-save
data.count = 4; // Auto-saved
```

### **Example 2: Start with Disabled Auto-Save**
```javascript
const settings = ReactiveUtils.reactive({
  theme: 'light'
});

ReactiveUtils.autoSave(settings, 'settings', {
  autoSave: false
});

// User enables auto-save
function enableAutoSave() {
  settings.$startAutoSave();
  console.log('Auto-save enabled');
}
```

### **Example 3: Online/Offline Toggle**
```javascript
const state = ReactiveUtils.reactive({
  data: [],
  isOnline: navigator.onLine
});

ReactiveUtils.autoSave(state, 'state');

window.addEventListener('online', () => {
  state.isOnline = true;

  // Sync and resume auto-save
  state.$save();
  state.$startAutoSave();

  console.log('Back online, auto-save enabled');
});

window.addEventListener('offline', () => {
  state.isOnline = false;
  state.$stopAutoSave();

  console.log('Offline, auto-save disabled');
});
```

### **Example 4: User Preference**
```javascript
const prefs = ReactiveUtils.reactive({
  autoSave: true
});

const editor = ReactiveUtils.reactive({
  content: ''
});

ReactiveUtils.autoSave(editor, 'editor');

function applyPreferences() {
  if (prefs.autoSave) {
    editor.$startAutoSave();
  } else {
    editor.$stopAutoSave();
  }
}

// Toggle via UI
document.getElementById('autoSave').addEventListener('change', (e) => {
  prefs.autoSave = e.target.checked;
  applyPreferences();
});
```

### **Example 5: After Import**
```javascript
const state = ReactiveUtils.reactive({
  records: []
});

ReactiveUtils.autoSave(state, 'records');

async function importData(file) {
  state.$stopAutoSave();

  const data = await parseFile(file);
  state.records = data;
  state.$save();

  // Resume auto-save for future edits
  state.$startAutoSave();

  console.log('Import complete, auto-save resumed');
}
```

### **Example 6: Conditional Auto-Save**
```javascript
const app = ReactiveUtils.reactive({
  mode: 'development',
  data: {}
});

ReactiveUtils.autoSave(app, 'app');

function updateMode(mode) {
  app.mode = mode;

  if (mode === 'production') {
    app.$startAutoSave();
    console.log('Production mode: auto-save enabled');
  } else {
    app.$stopAutoSave();
    console.log('Development mode: auto-save disabled');
  }
}
```

### **Example 7: Delayed Start**
```javascript
const state = ReactiveUtils.reactive({
  initialized: false,
  data: null
});

ReactiveUtils.autoSave(state, 'state', {
  autoSave: false
});

// Initialize without saving
state.data = getInitialData();
state.initialized = true;

// Now enable auto-save for user changes
state.$startAutoSave();
```

### **Example 8: Privacy Mode Toggle**
```javascript
const browsing = ReactiveUtils.reactive({
  history: [],
  privateMode: false
});

ReactiveUtils.autoSave(browsing, 'history');

function exitPrivateMode() {
  browsing.privateMode = false;
  browsing.$startAutoSave();

  console.log('Exited private mode');
  console.log('History will now be saved');
}

function enterPrivateMode() {
  browsing.privateMode = true;
  browsing.$stopAutoSave();

  console.log('Entered private mode');
  console.log('History will not be saved');
}
```

### **Example 9: After Validation**
```javascript
const form = ReactiveUtils.reactive({
  email: '',
  password: ''
});

ReactiveUtils.autoSave(form, 'form', {
  autoSave: false
});

async function validateAndEnable() {
  const valid = await validateForm(form);

  if (valid) {
    form.$startAutoSave();
    console.log('Form valid, auto-save enabled');
  } else {
    console.error('Form invalid, auto-save not enabled');
  }
}
```

### **Example 10: Feature Flag**
```javascript
const config = ReactiveUtils.reactive({
  features: {
    autoSave: false
  }
});

const appState = ReactiveUtils.reactive({
  data: []
});

ReactiveUtils.autoSave(appState, 'appState');

function checkFeatureFlags() {
  if (config.features.autoSave) {
    appState.$startAutoSave();
    console.log('Auto-save feature enabled');
  } else {
    appState.$stopAutoSave();
    console.log('Auto-save feature disabled');
  }
}
```

---

## **Common Patterns**

### **Pattern 1: Resume After Stop**
```javascript
state.$stopAutoSave();
// Do work
state.$startAutoSave();
```

### **Pattern 2: Enable on Init**
```javascript
ReactiveUtils.autoSave(state, 'key', { autoSave: false });
state.$startAutoSave(); // Enable when ready
```

### **Pattern 3: Toggle Based on Condition**
```javascript
if (shouldAutoSave) {
  state.$startAutoSave();
} else {
  state.$stopAutoSave();
}
```

### **Pattern 4: Enable After Import**
```javascript
importData();
state.$save();
state.$startAutoSave();
```

### **Pattern 5: User Control**
```javascript
function setAutoSave(enabled) {
  if (enabled) {
    state.$startAutoSave();
  } else {
    state.$stopAutoSave();
  }
}
```

---

## **When to Use**

| Scenario | Use $startAutoSave() |
|----------|---------------------|
| Resume after stop | ✓ Yes |
| Enable on demand | ✓ Yes |
| User preference | ✓ Yes |
| After validation | ✓ Yes |
| Online mode | ✓ Yes |
| Exit privacy mode | ✓ Yes |
| Feature flags | ✓ Yes |
| Auto-save already enabled | ✗ No (unnecessary) |

---

## **Important Notes**

### **Idempotent**
```javascript
state.$startAutoSave();
state.$startAutoSave(); // Safe to call multiple times
```

### **Immediate Effect**
```javascript
state.$startAutoSave();
state.value = 'new'; // Immediately auto-saved
```

### **Respects Settings**
```javascript
ReactiveUtils.autoSave(state, 'key', { debounce: 1000 });
state.$startAutoSave();
// Still uses 1000ms debounce
```

---

## **vs. autoSave: true**

| Feature | `$startAutoSave()` | `autoSave: true` option |
|---------|-------------------|------------------------|
| When set | Runtime | Initialization |
| Dynamic | ✓ Yes | ✗ No |
| Can toggle | ✓ Yes | ✗ No |

```javascript
// Option: Enabled at initialization
ReactiveUtils.autoSave(state, 'key', { autoSave: true });

// Method: Enable at runtime
ReactiveUtils.autoSave(state, 'key', { autoSave: false });
state.$startAutoSave(); // Enable when needed
```

---

## **Best Practices**

1. **Resume after batch operations**
   ```javascript
   state.$stopAutoSave();
   batchUpdate();
   state.$save();
   state.$startAutoSave(); // Resume
   ```

2. **Enable based on user preference**
   ```javascript
   if (userSettings.autoSave) {
     state.$startAutoSave();
   }
   ```

3. **Enable when online**
   ```javascript
   window.addEventListener('online', () => {
     state.$save(); // Sync first
     state.$startAutoSave(); // Then enable
   });
   ```

4. **Enable after validation**
   ```javascript
   if (isValid(state)) {
     state.$startAutoSave();
   }
   ```

5. **Pair with $stopAutoSave()**
   ```javascript
   state.$stopAutoSave();
   try {
     performOperation();
   } finally {
     state.$startAutoSave();
   }
   ```

6. **Check state before enabling**
   ```javascript
   if (!state.$storageInfo().autoSaveEnabled) {
     state.$startAutoSave();
   }
   ```

---

## **Key Takeaways**

1. **Enables Auto-Save**: Activates automatic saving
2. **Resumes Watching**: Re-establishes reactive effects
3. **Idempotent**: Safe to call multiple times
4. **Immediate**: Next change will auto-save
5. **Respects Settings**: Uses original debounce/options
6. **User Control**: Allows user to enable auto-save
7. **Dynamic**: Can toggle at runtime
8. **No Parameters**: Takes no arguments
9. **Returns Void**: No return value
10. **Pairs with Stop**: Use with $stopAutoSave() for control

---

## **Summary**

`$startAutoSave()` is a method added to reactive state objects by `autoSave()` that enables or resumes automatic saving after it has been disabled. When called, it re-establishes the reactive effect that watches for state changes and triggers saves according to the configured debounce settings. The method is idempotent, meaning it's safe to call multiple times without side effects. Use `$startAutoSave()` to resume auto-save after batch operations, to enable auto-save based on user preferences, when transitioning from offline to online mode, after data validation, when exiting privacy mode, or when feature flags enable auto-save functionality. It pairs with `$stopAutoSave()` to provide dynamic control over automatic saving behavior at runtime, unlike the autoSave option which is set at initialization and cannot be changed. The method takes no parameters and returns void, immediately affecting the state's auto-save behavior for all subsequent changes.
