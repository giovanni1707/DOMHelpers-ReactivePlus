# `autoSave(reactiveObj, key, options)` - Auto-Save Reactive State to Storage

**Quick Start (30 seconds)**
```javascript
// Basic auto-save to localStorage
const state = ReactiveUtils.reactive({ count: 0, name: 'John' });

ReactiveUtils.autoSave(state, 'myApp', {
  storage: 'localStorage',
  debounce: 500
});

// Changes automatically saved after 500ms
state.count = 5;
state.name = 'Jane';

// Reload page - state automatically restored!
console.log(state.count); // 5
console.log(state.name); // 'Jane'
```

---

## **What is `autoSave(reactiveObj, key, options)`?**

`autoSave()` is a **utility function** that automatically saves reactive state to browser storage (localStorage or sessionStorage) whenever it changes, with automatic loading on initialization and cross-tab synchronization support.

**Key characteristics:**
- **Auto-Save**: Automatically saves state on changes
- **Auto-Load**: Automatically loads saved state on initialization
- **Debouncing**: Configurable save delay to reduce writes
- **Cross-Tab Sync**: Synchronize state across browser tabs
- **Expiration**: Optional time-based expiration
- **Lifecycle Hooks**: onSave, onLoad, onSync, onError callbacks
- **Namespace Methods**: 7 methods available via ReactiveUtils (save, load, clear, etc.)
- **Namespace Support**: Organize storage with namespaces

---

## **Syntax**

```javascript
ReactiveUtils.autoSave(reactiveObj, key, options)
```

### **Parameters**
- **`reactiveObj`** (Object): Reactive state object to save
- **`key`** (string): Storage key for saving state
- **`options`** (Object, optional): Configuration options

### **Options**
```javascript
{
  storage: 'localStorage',        // 'localStorage' or 'sessionStorage'
  namespace: '',                  // Storage namespace prefix
  debounce: 0,                    // Debounce delay in ms (0 = immediate)
  autoLoad: true,                 // Auto-load on initialization
  autoSave: true,                 // Enable auto-save
  sync: false,                    // Cross-tab synchronization
  expires: null,                  // Expiration in seconds (null = never)
  onSave: null,                   // Callback(value) before save
  onLoad: null,                   // Callback(value) after load
  onSync: null,                   // Callback(value) on cross-tab sync
  onError: null                   // Callback(error) on errors
}
```

### **Returns**
- **Type**: `void` (modifies reactiveObj in place)

### **Namespace Methods for State Management**
After calling `autoSave()`, use these namespace methods to control storage:
- **`ReactiveUtils.save(state)`** - Force save immediately
- **`ReactiveUtils.load(state)`** - Force load from storage
- **`ReactiveUtils.clear(state)`** - Clear from storage
- **`ReactiveUtils.exists(state)`** - Check if exists in storage
- **`ReactiveUtils.stopAutoSave(state)`** - Stop automatic saving
- **`ReactiveUtils.startAutoSave(state)`** - Start automatic saving
- **`ReactiveUtils.storageInfo(state)`** - Get storage information

---

## **How it works**

```javascript
function autoSave(reactiveObj, key, options = {}) {
  const store = new StorageWrapper(storage, namespace);

  // 1. Auto-load initial data
  if (autoLoad) {
    const loaded = store.get(key);
    if (loaded !== null) {
      Object.assign(reactiveObj, onLoad ? onLoad(loaded) : loaded);
    }
  }

  // 2. Watch for changes and auto-save
  let saveTimeout;
  effect(() => {
    const value = toRaw(reactiveObj);

    clearTimeout(saveTimeout);
    saveTimeout = setTimeout(() => {
      const toSave = onSave ? onSave(value) : value;
      store.set(key, toSave, { expires });
    }, debounce);
  });

  // 3. Cross-tab sync (if enabled)
  if (sync) {
    window.addEventListener('storage', (e) => {
      if (e.key === fullKey) {
        const synced = onSync ? onSync(e.newValue) : e.newValue;
        Object.assign(reactiveObj, synced);
      }
    });
  }

  // 4. Add internal methods (accessed via ReactiveUtils namespace)
  reactiveObj.$save = () => store.set(key, toRaw(reactiveObj));
  reactiveObj.$load = () => Object.assign(reactiveObj, store.get(key));
  // ... more internal methods
}
```

**What happens:**
1. Loads existing data from storage on initialization (if autoLoad)
2. Sets up reactive effect to auto-save on changes (with debouncing)
3. Sets up cross-tab sync via storage events (if sync enabled)
4. Adds internal methods (use via ReactiveUtils.save(state), etc.)
5. Handles expiration, serialization, and error scenarios

---

## **Examples**

### **Example 1: Basic Auto-Save**
```javascript
const app = ReactiveUtils.reactive({
  theme: 'light',
  fontSize: 14
});

ReactiveUtils.autoSave(app, 'settings');

app.theme = 'dark';
app.fontSize = 16;

// Reload page
// app.theme is 'dark' and app.fontSize is 16
```

### **Example 2: Debounced Saves**
```javascript
const editor = ReactiveUtils.reactive({
  content: '',
  lastSaved: null
});

ReactiveUtils.autoSave(editor, 'draft', {
  debounce: 1000, // Save 1 second after last change
  onSave: (value) => {
    value.lastSaved = new Date().toISOString();
    console.log('Saving draft...');
    return value;
  }
});

// Type continuously
editor.content = 'H';
editor.content = 'He';
editor.content = 'Hel';
editor.content = 'Hell';
editor.content = 'Hello';
// Only saves once, 1 second after "Hello"
```

### **Example 3: Session Storage**
```javascript
const tempData = ReactiveUtils.reactive({
  sessionId: '12345',
  tempNotes: ''
});

ReactiveUtils.autoSave(tempData, 'session', {
  storage: 'sessionStorage', // Clears when tab closes
  autoLoad: true
});

tempData.tempNotes = 'Temporary notes';
// Available until tab closes
```

### **Example 4: Namespaced Storage**
```javascript
const userPrefs = ReactiveUtils.reactive({ lang: 'en', theme: 'light' });
const appState = ReactiveUtils.reactive({ sidebar: true, zoom: 100 });

ReactiveUtils.autoSave(userPrefs, 'preferences', {
  namespace: 'app:user'
  // Stored as: "app:user:preferences"
});

ReactiveUtils.autoSave(appState, 'state', {
  namespace: 'app:ui'
  // Stored as: "app:ui:state"
});
```

### **Example 5: Expiring Data**
```javascript
const cache = ReactiveUtils.reactive({
  data: null,
  fetchedAt: null
});

ReactiveUtils.autoSave(cache, 'api-cache', {
  expires: 3600, // Expire after 1 hour (3600 seconds)
  onLoad: (value) => {
    console.log('Cache loaded');
    return value;
  }
});

cache.data = { items: [1, 2, 3] };
cache.fetchedAt = Date.now();

// After 1 hour, data is automatically cleared
```

### **Example 6: Cross-Tab Synchronization**
```javascript
const sharedState = ReactiveUtils.reactive({
  messages: [],
  unreadCount: 0
});

ReactiveUtils.autoSave(sharedState, 'messages', {
  sync: true, // Enable cross-tab sync
  onSync: (value) => {
    console.log('Synced from another tab');
    showNotification('Messages updated');
    return value;
  }
});

// Change in Tab 1
sharedState.messages.push('New message');

// Tab 2 automatically updates!
```

### **Example 7: Transform on Save/Load**
```javascript
const user = ReactiveUtils.reactive({
  name: 'John',
  password: 'secret123',
  email: 'john@example.com'
});

ReactiveUtils.autoSave(user, 'user', {
  onSave: (value) => {
    // Don't save password
    const { password, ...safe } = value;
    return safe;
  },
  onLoad: (value) => {
    // Add default password field
    return { ...value, password: '' };
  }
});

// password is never saved to storage
```

### **Example 8: Manual Control Methods**
```javascript
const state = ReactiveUtils.reactive({ count: 0 });

ReactiveUtils.autoSave(state, 'counter', {
  autoSave: false // Disable auto-save initially
});

state.count = 10;
// Not saved yet

// Check if exists
if (ReactiveUtils.exists(state)) {
  console.log('Found existing data');
}

// Force save
ReactiveUtils.save(state);
console.log('Saved manually');

// Stop auto-save
ReactiveUtils.stopAutoSave(state);

// Start auto-save
ReactiveUtils.startAutoSave(state);

// Get info
console.log(ReactiveUtils.storageInfo(state));
// { key: 'counter', storage: 'localStorage', namespace: '', size: 15 }
```

### **Example 9: Error Handling**
```javascript
const state = ReactiveUtils.reactive({ data: [] });

ReactiveUtils.autoSave(state, 'bigData', {
  onError: (error) => {
    if (error.name === 'QuotaExceededError') {
      console.error('Storage quota exceeded!');
      // Clear old data or notify user
      ReactiveUtils.clear(state);
      alert('Storage full. Clearing old data.');
    } else {
      console.error('Storage error:', error);
    }
  }
});

// Try to save too much data
try {
  state.data = new Array(1000000).fill('large data');
} catch (e) {
  // onError handles it
}
```

### **Example 10: Shopping Cart**
```javascript
const cart = ReactiveUtils.reactive({
  items: [],
  total: 0
});

ReactiveUtils.autoSave(cart, 'cart', {
  debounce: 500,
  onSave: (value) => {
    console.log(`Saving cart with ${value.items.length} items`);
    return value;
  },
  onLoad: (value) => {
    console.log(`Loaded cart with ${value.items.length} items`);
    return value;
  }
});

// Add items
cart.items.push({ id: 1, name: 'Product A', price: 29.99 });
cart.items.push({ id: 2, name: 'Product B', price: 49.99 });
cart.total = cart.items.reduce((sum, item) => sum + item.price, 0);

// Auto-saved, persists across page reloads
```

### **Example 11: Form Draft Auto-Save**
```javascript
const formData = ReactiveUtils.reactive({
  title: '',
  body: '',
  tags: [],
  lastModified: null
});

ReactiveUtils.autoSave(formData, 'post-draft', {
  debounce: 2000, // Save 2s after typing stops
  onSave: (value) => {
    value.lastModified = new Date().toISOString();
    console.log('Draft saved');
    document.querySelector('.save-indicator').textContent = '✓ Saved';
    return value;
  }
});

// Auto-saves as user types
formData.title = 'My Blog Post';
formData.body = 'Content here...';
```

### **Example 12: Multi-User Preferences**
```javascript
function createUserPreferences(userId) {
  const prefs = ReactiveUtils.reactive({
    theme: 'system',
    notifications: true,
    language: 'en'
  });

  ReactiveUtils.autoSave(prefs, `prefs-${userId}`, {
    namespace: 'users',
    autoLoad: true
  });

  return prefs;
}

const user1Prefs = createUserPreferences('user123');
const user2Prefs = createUserPreferences('user456');

// Each user has separate preferences
user1Prefs.theme = 'dark';
user2Prefs.theme = 'light';
```

### **Example 13: Game State Persistence**
```javascript
const gameState = ReactiveUtils.reactive({
  level: 1,
  score: 0,
  inventory: [],
  position: { x: 0, y: 0 }
});

ReactiveUtils.autoSave(gameState, 'savegame', {
  debounce: 100,
  onSave: (value) => {
    value.savedAt = Date.now();
    return value;
  },
  onLoad: (value) => {
    console.log(`Loaded save from level ${value.level}`);
    return value;
  }
});

// Game progress auto-saves
gameState.score += 100;
gameState.level = 2;
gameState.inventory.push('sword');
```

### **Example 14: Conditional Auto-Save**
```javascript
const editor = ReactiveUtils.reactive({
  content: '',
  isDirty: false,
  isOnline: navigator.onLine
});

ReactiveUtils.autoSave(editor, 'document', {
  debounce: 1000,
  onSave: (value) => {
    if (!value.isOnline) {
      console.log('Offline - saving locally only');
    }
    return value;
  }
});

// Update online status
window.addEventListener('online', () => {
  editor.isOnline = true;
  ReactiveUtils.save(editor); // Sync when back online
});

window.addEventListener('offline', () => {
  editor.isOnline = false;
});
```

### **Example 15: Load with Fallback**
```javascript
const config = ReactiveUtils.reactive({
  apiUrl: 'https://api.example.com',
  timeout: 5000,
  retries: 3
});

ReactiveUtils.autoSave(config, 'config', {
  autoLoad: true,
  onLoad: (value) => {
    // Merge with defaults
    return {
      apiUrl: 'https://api.example.com',
      timeout: 5000,
      retries: 3,
      ...value
    };
  }
});

// Loaded config will have defaults for missing properties
```

---

## **Common Patterns**

### **Pattern 1: Basic Auto-Save**
```javascript
ReactiveUtils.autoSave(state, 'key');
```

### **Pattern 2: Debounced Auto-Save**
```javascript
ReactiveUtils.autoSave(state, 'key', { debounce: 500 });
```

### **Pattern 3: Cross-Tab Sync**
```javascript
ReactiveUtils.autoSave(state, 'key', { sync: true });
```

### **Pattern 4: Transform on Save**
```javascript
ReactiveUtils.autoSave(state, 'key', {
  onSave: (value) => sanitize(value)
});
```

### **Pattern 5: Manual Save**
```javascript
ReactiveUtils.save(state); // Force save
```

---

## **Storage Methods Reference**

### **`save(state)` - Force Save**
```javascript
ReactiveUtils.save(state);
// Returns: boolean
// Saves immediately, bypassing debounce
```

### **`load(state)` - Force Load**
```javascript
ReactiveUtils.load(state);
// Returns: boolean
// Loads from storage, overwriting current state
```

### **`clear(state)` - Clear Storage**
```javascript
ReactiveUtils.clear(state);
// Returns: boolean
// Removes from storage
```

### **`exists(state)` - Check Existence**
```javascript
const exists = ReactiveUtils.exists(state);
// Returns: boolean
// true if data exists in storage
```

### **`stopAutoSave(state)` - Stop Auto-Save**
```javascript
ReactiveUtils.stopAutoSave(state);
// Returns: Object (the state)
// Stops automatic saving
```

### **`startAutoSave(state)` - Start Auto-Save**
```javascript
ReactiveUtils.startAutoSave(state);
// Returns: Object (the state)
// Starts automatic saving
```

### **`storageInfo(state)` - Get Storage Info**
```javascript
const info = ReactiveUtils.storageInfo(state);
// Returns: { key, storage, namespace, size, exists }
// size in bytes (approximate)
```

---

## **Options Reference**

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `storage` | string | `'localStorage'` | Storage type ('localStorage' or 'sessionStorage') |
| `namespace` | string | `''` | Namespace prefix for storage key |
| `debounce` | number | `0` | Debounce delay in milliseconds (0 = immediate) |
| `autoLoad` | boolean | `true` | Auto-load saved data on initialization |
| `autoSave` | boolean | `true` | Enable automatic saving |
| `sync` | boolean | `false` | Enable cross-tab synchronization |
| `expires` | number\|null | `null` | Expiration time in seconds (null = never) |
| `onSave` | function\|null | `null` | Transform value before save `(value) => newValue` |
| `onLoad` | function\|null | `null` | Transform value after load `(value) => newValue` |
| `onSync` | function\|null | `null` | Called on cross-tab sync `(value) => newValue` |
| `onError` | function\|null | `null` | Error handler `(error) => void` |

---

## **Best Practices**

1. **Use debouncing for frequent updates**
   ```javascript
   ReactiveUtils.autoSave(editor, 'draft', { debounce: 1000 });
   ```

2. **Transform sensitive data**
   ```javascript
   ReactiveUtils.autoSave(user, 'user', {
     onSave: (value) => {
       const { password, ...safe } = value;
       return safe;
     }
   });
   ```

3. **Handle storage quota errors**
   ```javascript
   ReactiveUtils.autoSave(state, 'key', {
     onError: (error) => {
       if (error.name === 'QuotaExceededError') {
         // Handle quota exceeded
       }
     }
   });
   ```

4. **Use namespaces for organization**
   ```javascript
   ReactiveUtils.autoSave(state, 'data', {
     namespace: 'myApp:v1'
   });
   ```

5. **Set expiration for temporary data**
   ```javascript
   ReactiveUtils.autoSave(cache, 'temp', {
     expires: 3600 // 1 hour
   });
   ```

6. **Validate loaded data**
   ```javascript
   ReactiveUtils.autoSave(state, 'key', {
     onLoad: (value) => {
       return isValid(value) ? value : getDefaults();
     }
   });
   ```

7. **Use sessionStorage for sensitive data**
   ```javascript
   ReactiveUtils.autoSave(sensitiveData, 'key', {
     storage: 'sessionStorage'
   });
   ```

---

## **Key Takeaways**

1. **Auto-Save**: Automatically saves reactive state to storage
2. **Auto-Load**: Automatically loads saved state on initialization
3. **Debouncing**: Configurable delay reduces storage writes
4. **Cross-Tab**: Synchronize state across browser tabs
5. **Expiration**: Time-based automatic data expiration
6. **Lifecycle**: onSave, onLoad, onSync, onError hooks
7. **Methods**: 7 namespace methods for manual control (save, load, clear, etc.)
8. **Namespace**: Organize storage with key prefixes
9. **Transform**: Modify data before save/after load
10. **Error Handling**: Built-in quota and error management

---

## **Summary**

`autoSave(reactiveObj, key, options)` is a powerful utility that automatically persists reactive state to browser storage (localStorage or sessionStorage) with automatic loading, debouncing, and cross-tab synchronization. It watches the reactive object for changes and saves them to storage, with configurable debounce delays to reduce write frequency. The function supports automatic loading of saved data on initialization, making state persistence seamless across page reloads. Advanced features include cross-tab synchronization via storage events, time-based expiration, namespace organization, and lifecycle hooks (onSave, onLoad, onSync, onError) for transforming data and handling errors. Seven namespace methods are available for manual control: ReactiveUtils.save(state), load(state), clear(state), exists(state), stopAutoSave(state), startAutoSave(state), and storageInfo(state). Use debouncing for frequently changing data like editor content, enable cross-tab sync for shared application state, and implement proper error handling for storage quota issues. Transform sensitive data with onSave/onLoad callbacks to avoid storing passwords or tokens. The auto-save feature is perfect for user preferences, form drafts, shopping carts, game saves, and any state that should persist across sessions.
