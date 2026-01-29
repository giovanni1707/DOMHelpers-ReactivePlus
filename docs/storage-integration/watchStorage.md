# `watchStorage(key, callback, options)` - Watch Storage Changes

**Quick Start (30 seconds)**
```javascript
// Watch for storage changes
const cleanup = ReactiveUtils.watchStorage('userPrefs', (newValue, oldValue) => {
  console.log('Preferences changed!');
  console.log('Old:', oldValue);
  console.log('New:', newValue);
  updateUI(newValue);
});

// Change storage (in this or another tab)
localStorage.setItem('userPrefs', JSON.stringify({ theme: 'dark' }));
// Callback runs automatically

// Stop watching
cleanup();
```

---

## **What is `watchStorage(key, callback, options)`?**

`watchStorage()` is a **utility function** that watches for changes to a specific storage key and executes a callback when the value changes, with support for cross-tab synchronization and reactive tracking.

**Key characteristics:**
- **Watch Changes**: Callback runs when storage value changes
- **Cross-Tab**: Detects changes from other browser tabs
- **Old/New Values**: Provides both old and new values to callback
- **Reactive**: Uses reactive effects for tracking
- **Cleanup**: Returns cleanup function to stop watching
- **Namespace**: Supports namespaced storage
- **Storage Types**: Works with localStorage and sessionStorage

---

## **Syntax**

```javascript
ReactiveUtils.watchStorage(key, callback, options)
```

### **Parameters**
- **`key`** (string): Storage key to watch
- **`callback`** (function): Callback function `(newValue, oldValue) => void`
- **`options`** (Object, optional): Configuration options

### **Options**
```javascript
{
  storage: 'localStorage',      // 'localStorage' or 'sessionStorage'
  namespace: '',                // Storage namespace prefix
  immediate: false              // Run callback immediately with current value
}
```

### **Returns**
- **Type**: Function - Cleanup function to stop watching

### **Callback Signature**
```javascript
function callback(newValue, oldValue) {
  // newValue: Current value from storage (or null)
  // oldValue: Previous value (or null)
}
```

---

## **How it works**

```javascript
function watchStorage(key, callback, options = {}) {
  const {
    storage = 'localStorage',
    namespace = '',
    immediate = false
  } = options;

  const reactiveStore = reactiveStorage(storage, namespace);
  let oldValue = reactiveStore.get(key);

  // Run immediately if requested
  if (immediate && oldValue !== null) {
    callback(oldValue, null);
  }

  // Create reactive effect to watch storage
  const cleanup = effect(() => {
    const newValue = reactiveStore.get(key);

    // Check if value actually changed
    if (JSON.stringify(newValue) !== JSON.stringify(oldValue)) {
      callback(newValue, oldValue);
      oldValue = newValue;
    }
  });

  return cleanup;
}
```

**What happens:**
1. Creates reactive storage wrapper for specified storage type
2. Gets initial value from storage
3. Optionally runs callback immediately with current value
4. Creates reactive effect that watches the storage key
5. Compares new value to old value on each check
6. Calls callback when value changes
7. Returns cleanup function to stop watching

---

## **Examples**

### **Example 1: Watch User Preferences**
```javascript
const cleanup = ReactiveUtils.watchStorage('userPrefs', (newValue, oldValue) => {
  console.log('Preferences updated');
  applyTheme(newValue.theme);
  setLanguage(newValue.language);
});

// Later: stop watching
cleanup();
```

### **Example 2: Cross-Tab Synchronization**
```javascript
// Tab 1: Watch for changes
ReactiveUtils.watchStorage('sharedData', (newValue) => {
  console.log('Data updated from another tab:', newValue);
  updateLocalState(newValue);
});

// Tab 2: Make changes
localStorage.setItem('sharedData', JSON.stringify({ count: 42 }));

// Tab 1 callback runs automatically!
```

### **Example 3: Immediate Execution**
```javascript
ReactiveUtils.watchStorage('settings', (newValue, oldValue) => {
  console.log('Settings:', newValue);
}, { immediate: true });

// Callback runs immediately with current value
// Then runs again on changes
```

### **Example 4: Namespaced Watching**
```javascript
ReactiveUtils.watchStorage('config', (newValue) => {
  console.log('App config changed:', newValue);
}, {
  namespace: 'myApp'
});

// Watches "myApp:config" key
```

### **Example 5: Session Storage**
```javascript
ReactiveUtils.watchStorage('tempData', (newValue, oldValue) => {
  console.log('Session data changed');
  console.log('From:', oldValue);
  console.log('To:', newValue);
}, {
  storage: 'sessionStorage'
});
```

### **Example 6: Theme Switching**
```javascript
const stopWatchingTheme = ReactiveUtils.watchStorage('theme', (theme) => {
  document.body.className = theme + '-theme';

  if (theme === 'dark') {
    loadDarkModeStyles();
  } else {
    loadLightModeStyles();
  }
}, { immediate: true });

// Apply theme immediately and on changes
```

### **Example 7: Shopping Cart Updates**
```javascript
ReactiveUtils.watchStorage('cart', (newCart, oldCart) => {
  const oldCount = oldCart?.items?.length || 0;
  const newCount = newCart?.items?.length || 0;

  console.log(`Cart updated: ${oldCount} → ${newCount} items`);

  updateCartBadge(newCount);
  recalculateTotal(newCart);
}, { namespace: 'shop' });
```

### **Example 8: Multiple Watchers**
```javascript
// Watch user data
const stopUser = ReactiveUtils.watchStorage('user', (user) => {
  updateUserDisplay(user);
});

// Watch notifications
const stopNotifications = ReactiveUtils.watchStorage('notifications', (notifs) => {
  updateNotificationBadge(notifs.length);
});

// Watch settings
const stopSettings = ReactiveUtils.watchStorage('settings', (settings) => {
  applySettings(settings);
});

// Clean up all watchers
function cleanup() {
  stopUser();
  stopNotifications();
  stopSettings();
}
```

### **Example 9: Conditional Updates**
```javascript
ReactiveUtils.watchStorage('data', (newValue, oldValue) => {
  if (!newValue) {
    console.log('Data was removed');
    resetToDefaults();
    return;
  }

  if (!oldValue) {
    console.log('Data was created');
    initializeFromData(newValue);
    return;
  }

  console.log('Data was updated');
  updateFromData(newValue);
});
```

### **Example 10: Notification System**
```javascript
ReactiveUtils.watchStorage('notifications', (newNotifs, oldNotifs) => {
  const oldCount = oldNotifs?.length || 0;
  const newCount = newNotifs?.length || 0;

  if (newCount > oldCount) {
    const newOnes = newNotifs.slice(oldCount);
    newOnes.forEach(notif => {
      showNotification(notif.title, notif.message);
    });
  }
});
```

### **Example 11: Auto-Logout on Auth Change**
```javascript
ReactiveUtils.watchStorage('authToken', (newToken, oldToken) => {
  if (oldToken && !newToken) {
    console.log('User logged out in another tab');
    redirectToLogin();
  }

  if (!oldToken && newToken) {
    console.log('User logged in another tab');
    loadUserData();
  }
});
```

### **Example 12: Form Draft Sync**
```javascript
let formState = { title: '', content: '' };

ReactiveUtils.watchStorage('formDraft', (draft) => {
  if (draft) {
    formState = draft;
    document.getElementById('title').value = draft.title;
    document.getElementById('content').value = draft.content;
    console.log('Form synced from storage');
  }
}, { immediate: true });

// Auto-restore form on page load
```

### **Example 13: Language Switching**
```javascript
ReactiveUtils.watchStorage('language', (lang, oldLang) => {
  if (lang !== oldLang) {
    console.log(`Language changed: ${oldLang} → ${lang}`);
    loadTranslations(lang).then(() => {
      retranslateUI();
    });
  }
}, { immediate: true });
```

### **Example 14: Cache Invalidation**
```javascript
const cache = new Map();

ReactiveUtils.watchStorage('cacheVersion', (newVersion, oldVersion) => {
  if (newVersion !== oldVersion) {
    console.log('Cache version changed, clearing cache');
    cache.clear();
    reloadData();
  }
});
```

### **Example 15: Feature Flags**
```javascript
ReactiveUtils.watchStorage('featureFlags', (flags) => {
  console.log('Feature flags updated:', flags);

  if (flags.darkMode) {
    enableDarkMode();
  }

  if (flags.betaFeatures) {
    showBetaFeatures();
  }

  if (flags.analytics) {
    enableAnalytics();
  }
}, { immediate: true, namespace: 'app' });
```

### **Example 16: Connection Status**
```javascript
ReactiveUtils.watchStorage('connectionStatus', (status, oldStatus) => {
  const indicator = document.getElementById('status');

  if (status === 'online' && oldStatus === 'offline') {
    indicator.textContent = '🟢 Online';
    indicator.className = 'online';
    syncPendingChanges();
  } else if (status === 'offline') {
    indicator.textContent = '🔴 Offline';
    indicator.className = 'offline';
  }
});
```

### **Example 17: Settings Panel Auto-Update**
```javascript
const settingsPanel = {
  init() {
    this.cleanup = ReactiveUtils.watchStorage('appSettings', (settings) => {
      this.render(settings);
    }, { immediate: true });
  },

  render(settings) {
    document.getElementById('theme').value = settings.theme;
    document.getElementById('fontSize').value = settings.fontSize;
    document.getElementById('notifications').checked = settings.notifications;
  },

  destroy() {
    this.cleanup();
  }
};

settingsPanel.init();
```

### **Example 18: Data Validation**
```javascript
ReactiveUtils.watchStorage('userData', (data, oldData) => {
  if (!isValidUserData(data)) {
    console.error('Invalid user data received');
    // Restore old data
    localStorage.setItem('userData', JSON.stringify(oldData));
    showError('Invalid data format');
    return;
  }

  updateUserProfile(data);
});

function isValidUserData(data) {
  return data && data.id && data.name && data.email;
}
```

### **Example 19: Debugging Storage Changes**
```javascript
// Debug mode: log all changes to specific key
if (DEBUG) {
  ReactiveUtils.watchStorage('debugKey', (newValue, oldValue) => {
    console.group('Storage Change: debugKey');
    console.log('Old:', oldValue);
    console.log('New:', newValue);
    console.log('Changed at:', new Date().toISOString());
    console.trace('Stack trace');
    console.groupEnd();
  });
}
```

### **Example 20: Graceful Cleanup**
```javascript
class StorageWatcher {
  constructor() {
    this.watchers = [];
  }

  watch(key, callback, options) {
    const cleanup = ReactiveUtils.watchStorage(key, callback, options);
    this.watchers.push(cleanup);
    return cleanup;
  }

  destroy() {
    this.watchers.forEach(cleanup => cleanup());
    this.watchers = [];
  }
}

// Usage
const watcher = new StorageWatcher();
watcher.watch('key1', callback1);
watcher.watch('key2', callback2);

// Clean up all
watcher.destroy();
```

---

## **Common Patterns**

### **Pattern 1: Basic Watch**
```javascript
const stop = ReactiveUtils.watchStorage('key', (newValue) => {
  console.log(newValue);
});
```

### **Pattern 2: With Cleanup**
```javascript
const cleanup = ReactiveUtils.watchStorage('key', callback);
// Later
cleanup();
```

### **Pattern 3: Immediate Execution**
```javascript
ReactiveUtils.watchStorage('key', callback, { immediate: true });
```

### **Pattern 4: Cross-Tab Sync**
```javascript
ReactiveUtils.watchStorage('shared', (value) => {
  updateLocalState(value);
});
```

### **Pattern 5: Compare Values**
```javascript
ReactiveUtils.watchStorage('key', (newVal, oldVal) => {
  if (newVal !== oldVal) {
    handleChange(newVal, oldVal);
  }
});
```

---

## **Callback Parameters**

### **`newValue`**
- Current value from storage
- `null` if key doesn't exist or expired
- Automatically deserialized from JSON

### **`oldValue`**
- Previous value
- `null` on first call (unless immediate mode)
- Useful for comparing changes

---

## **Return Value (Cleanup Function)**

```javascript
const cleanup = ReactiveUtils.watchStorage('key', callback);

// Stop watching
cleanup();
```

**What cleanup does:**
- Removes the reactive effect
- Stops watching for changes
- Prevents memory leaks
- Should be called when watcher no longer needed

---

## **When to Use**

| Scenario | Use watchStorage() |
|----------|-------------------|
| React to storage changes | ✓ Yes |
| Cross-tab synchronization | ✓ Yes |
| Theme switching | ✓ Yes |
| Settings updates | ✓ Yes |
| Auth state changes | ✓ Yes |
| One-time read | ✗ No (use storage.get()) |
| Automatic state sync | ✗ No (use autoSave()) |

---

## **vs. Storage Events**

| Feature | `watchStorage()` | Native storage event |
|---------|-----------------|---------------------|
| Cross-tab | ✓ Yes | ✓ Yes |
| Same-tab | ✓ Yes | ✗ No |
| Reactive | ✓ Yes | ✗ No |
| Old/New values | ✓ Yes | ✓ Yes (event.oldValue/newValue) |
| Cleanup | ✓ Simple function | Manual removeEventListener |
| Namespace support | ✓ Yes | ✗ No |

---

## **vs. autoSave()**

| Feature | `watchStorage()` | `autoSave()` |
|---------|-----------------|-------------|
| Purpose | Watch storage key | Auto-save state |
| Direction | Storage → Code | Code → Storage |
| Reactivity | Watches storage | Watches state |
| Use Case | React to external changes | Persist internal state |

```javascript
// watchStorage - watch for storage changes
ReactiveUtils.watchStorage('key', (value) => {
  state.data = value; // Update state from storage
});

// autoSave - save state to storage
ReactiveUtils.autoSave(state, 'key');
state.data = newValue; // Saves to storage
```

---

## **Best Practices**

1. **Always clean up watchers**
   ```javascript
   const cleanup = ReactiveUtils.watchStorage('key', callback);
   // On component unmount or cleanup
   cleanup();
   ```

2. **Use immediate for initial state**
   ```javascript
   ReactiveUtils.watchStorage('key', callback, { immediate: true });
   ```

3. **Check for null values**
   ```javascript
   ReactiveUtils.watchStorage('key', (value) => {
     if (value === null) {
       // Handle missing/expired data
       return;
     }
     processValue(value);
   });
   ```

4. **Use namespaces for organization**
   ```javascript
   ReactiveUtils.watchStorage('config', callback, { namespace: 'app' });
   ```

5. **Compare old and new values**
   ```javascript
   ReactiveUtils.watchStorage('key', (newVal, oldVal) => {
     if (hasSignificantChange(newVal, oldVal)) {
       handleChange(newVal);
     }
   });
   ```

6. **Validate data**
   ```javascript
   ReactiveUtils.watchStorage('key', (value) => {
     if (!isValid(value)) {
       console.error('Invalid data');
       return;
     }
     applyValue(value);
   });
   ```

7. **Debounce rapid changes if needed**
   ```javascript
   let timeout;
   ReactiveUtils.watchStorage('key', (value) => {
     clearTimeout(timeout);
     timeout = setTimeout(() => {
       processValue(value);
     }, 300);
   });
   ```

---

## **Key Takeaways**

1. **Watch Changes**: Callback runs when storage value changes
2. **Cross-Tab**: Detects changes from other browser tabs
3. **Reactive**: Uses reactive effects for automatic tracking
4. **Cleanup**: Returns function to stop watching
5. **Old/New**: Provides both old and new values
6. **Immediate**: Optional immediate execution with current value
7. **Namespace**: Supports namespaced storage keys
8. **Storage Types**: Works with localStorage and sessionStorage
9. **Automatic**: No manual polling required
10. **Memory Safe**: Cleanup prevents memory leaks

---

## **Summary**

`watchStorage(key, callback, options)` watches for changes to a specific storage key and executes a callback when the value changes, with support for cross-tab synchronization and reactive tracking. The function creates a reactive effect that monitors the specified storage key, comparing values on each check and calling the callback when changes are detected. It provides both old and new values to the callback, making it easy to track what changed. The function works across browser tabs using storage events, so changes made in one tab automatically trigger callbacks in other tabs. Options include storage type (localStorage/sessionStorage), namespace support for key organization, and immediate execution to run the callback with the current value on setup. The function returns a cleanup function that should be called to stop watching and prevent memory leaks. Use watchStorage() for reacting to storage changes, implementing cross-tab synchronization, updating UI when settings change, or monitoring authentication state. It's perfect for theme switching, settings panels, notifications, shopping cart synchronization, and any scenario where you need to react to storage changes from the current tab or other tabs.
