# `watchStorage()` Options - Configuration Reference

**Quick Start (30 seconds)**
```javascript
// Basic usage with all options
const cleanup = ReactiveUtils.watchStorage('theme', (newValue, oldValue) => {
  console.log('Theme changed:', newValue);
  applyTheme(newValue);
}, {
  storage: 'localStorage',      // Which storage to watch
  namespace: 'myApp',            // Key prefix (watches "myApp:theme")
  immediate: true                // Call callback now with current value
});

// Later: stop watching
cleanup();
```

---

## **All Options**

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `storage` | string | `'localStorage'` | Storage type to watch: 'localStorage' or 'sessionStorage' |
| `namespace` | string | `''` | Namespace prefix for the key |
| `immediate` | boolean | `false` | Call callback immediately with current value |

---

## **Option 1: `storage`**

### **Description**
Specifies which browser storage type to watch: `'localStorage'` (persistent) or `'sessionStorage'` (session-only).

### **Type**
`string`

### **Default**
`'localStorage'`

### **Values**
- `'localStorage'` - Watch localStorage (persists across sessions)
- `'sessionStorage'` - Watch sessionStorage (cleared on tab close)

### **Syntax**
```javascript
ReactiveUtils.watchStorage('key', callback, {
  storage: 'localStorage'
});
```

---

### **Examples**

#### **Watch localStorage (Default)**
```javascript
// Explicit localStorage
const cleanup = ReactiveUtils.watchStorage('userPrefs', (prefs) => {
  console.log('Preferences updated:', prefs);
  applyPreferences(prefs);
}, {
  storage: 'localStorage'
});

// Same as default (localStorage assumed)
const cleanup2 = ReactiveUtils.watchStorage('userPrefs', (prefs) => {
  applyPreferences(prefs);
});
```

#### **Watch sessionStorage**
```javascript
// Watch session-only data
const cleanup = ReactiveUtils.watchStorage('tempData', (data) => {
  console.log('Session data changed:', data);
  processSessionData(data);
}, {
  storage: 'sessionStorage'
});

// Good for: form drafts, temporary state, session tokens
```

#### **Different Storage Types**
```javascript
// Watch persistent theme
ReactiveUtils.watchStorage('theme', (theme) => {
  document.body.className = theme + '-theme';
}, {
  storage: 'localStorage'  // Persists forever
});

// Watch session-only auth token
ReactiveUtils.watchStorage('authToken', (token) => {
  if (token) {
    authenticateUser(token);
  } else {
    showLogin();
  }
}, {
  storage: 'sessionStorage'  // Cleared on tab close
});
```

#### **Conditional Storage Type**
```javascript
function watchUserData(isPersistent = true) {
  const storageType = isPersistent ? 'localStorage' : 'sessionStorage';

  return ReactiveUtils.watchStorage('userData', (data) => {
    updateUserDisplay(data);
  }, {
    storage: storageType
  });
}

// Persistent storage
const cleanup1 = watchUserData(true);

// Session-only storage
const cleanup2 = watchUserData(false);
```

#### **Multiple Watchers on Different Storage**
```javascript
// Watch localStorage for persistent data
const cleanupLocal = ReactiveUtils.watchStorage('settings', (settings) => {
  console.log('Settings changed:', settings);
}, {
  storage: 'localStorage'
});

// Watch sessionStorage for temporary data
const cleanupSession = ReactiveUtils.watchStorage('formDraft', (draft) => {
  console.log('Draft changed:', draft);
}, {
  storage: 'sessionStorage'
});

// Cleanup both
function cleanupAll() {
  cleanupLocal();
  cleanupSession();
}
```

---

### **When to Use Each Storage Type**

| Use Case | Storage Type | Reason |
|----------|-------------|--------|
| User preferences | `localStorage` | Persist across sessions |
| Theme settings | `localStorage` | Remember after browser restart |
| Cached data | `localStorage` | Keep between page loads |
| Form drafts | `sessionStorage` | Clear when tab closes |
| Session tokens | `sessionStorage` | Security - clear on tab close |
| Temporary wizard data | `sessionStorage` | Don't persist incomplete flows |
| Shopping cart | `localStorage` | Preserve cart items |
| Current page state | `sessionStorage` | Don't persist to other sessions |

---

## **Option 2: `namespace`**

### **Description**
Prefix added to the storage key for organization and collision prevention. The actual key watched will be `namespace:key`.

### **Type**
`string`

### **Default**
`''` (no namespace)

### **Syntax**
```javascript
ReactiveUtils.watchStorage('key', callback, {
  namespace: 'myApp'
});
// Watches "myApp:key" in storage
```

---

### **Examples**

#### **Without Namespace (Default)**
```javascript
ReactiveUtils.watchStorage('settings', (settings) => {
  console.log('Settings:', settings);
});
// Watches: "settings"
```

#### **With Namespace**
```javascript
ReactiveUtils.watchStorage('settings', (settings) => {
  console.log('App settings:', settings);
}, {
  namespace: 'myApp'
});
// Watches: "myApp:settings"
```

#### **Multiple Apps with Namespaces**
```javascript
// App 1: Main application
ReactiveUtils.watchStorage('config', (config) => {
  console.log('Main app config:', config);
}, {
  namespace: 'mainApp'
});
// Watches: "mainApp:config"

// App 2: Admin panel
ReactiveUtils.watchStorage('config', (config) => {
  console.log('Admin config:', config);
}, {
  namespace: 'adminPanel'
});
// Watches: "adminPanel:config"

// No collision - different namespaces!
```

#### **Feature-Based Namespaces**
```javascript
// User feature
ReactiveUtils.watchStorage('data', (data) => {
  updateUserData(data);
}, {
  namespace: 'user'
});
// Watches: "user:data"

// Cart feature
ReactiveUtils.watchStorage('data', (data) => {
  updateCartData(data);
}, {
  namespace: 'cart'
});
// Watches: "cart:data"

// Settings feature
ReactiveUtils.watchStorage('data', (data) => {
  updateSettings(data);
}, {
  namespace: 'settings'
});
// Watches: "settings:data"
```

#### **Multi-Tenant Applications**
```javascript
function watchTenantData(tenantId, callback) {
  return ReactiveUtils.watchStorage('data', callback, {
    namespace: `tenant:${tenantId}`
  });
}

// Tenant 1
const cleanup1 = watchTenantData('company-a', (data) => {
  console.log('Company A data:', data);
});
// Watches: "tenant:company-a:data"

// Tenant 2
const cleanup2 = watchTenantData('company-b', (data) => {
  console.log('Company B data:', data);
});
// Watches: "tenant:company-b:data"
```

#### **Hierarchical Namespaces**
```javascript
// Application level
ReactiveUtils.watchStorage('version', (version) => {
  console.log('App version:', version);
}, {
  namespace: 'app'
});
// Watches: "app:version"

// User level
ReactiveUtils.watchStorage('preferences', (prefs) => {
  console.log('User prefs:', prefs);
}, {
  namespace: 'app:user'
});
// Watches: "app:user:preferences"

// Session level
ReactiveUtils.watchStorage('state', (state) => {
  console.log('Session state:', state);
}, {
  namespace: 'app:session'
});
// Watches: "app:session:state"
```

#### **Library Isolation**
```javascript
// Your library uses namespace to avoid conflicts
class MyLibrary {
  constructor(appName) {
    this.namespace = `mylib:${appName}`;
  }

  watchConfig(callback) {
    return ReactiveUtils.watchStorage('config', callback, {
      namespace: this.namespace
    });
  }

  watchState(callback) {
    return ReactiveUtils.watchStorage('state', callback, {
      namespace: this.namespace
    });
  }
}

const lib = new MyLibrary('todoApp');
lib.watchConfig(config => console.log('Config:', config));
// Watches: "mylib:todoApp:config"
```

---

### **Namespace Best Practices**

1. **Use consistent naming**
   ```javascript
   // Good: consistent kebab-case
   namespace: 'my-app'
   namespace: 'user-settings'

   // Avoid: mixed styles
   namespace: 'myApp'  // camelCase
   namespace: 'user_settings'  // snake_case
   ```

2. **Use hierarchical structure**
   ```javascript
   namespace: 'app'
   namespace: 'app:user'
   namespace: 'app:user:preferences'
   ```

3. **Include app/library name**
   ```javascript
   // Library developer
   namespace: 'myLibrary:feature'

   // App developer
   namespace: 'todoApp:settings'
   ```

4. **Use for multi-tenancy**
   ```javascript
   namespace: `tenant:${tenantId}`
   namespace: `org:${orgId}`
   ```

---

## **Option 3: `immediate`**

### **Description**
When `true`, calls the callback immediately with the current value from storage before watching for changes.

### **Type**
`boolean`

### **Default**
`false`

### **Syntax**
```javascript
ReactiveUtils.watchStorage('key', callback, {
  immediate: true
});
```

---

### **Examples**

#### **Without Immediate (Default)**
```javascript
let callCount = 0;

ReactiveUtils.watchStorage('theme', (theme) => {
  callCount++;
  console.log(`Call #${callCount}:`, theme);
});

console.log('Watcher setup complete');
// Output: "Watcher setup complete"
// Callback NOT called yet

// Change the value
localStorage.setItem('theme', JSON.stringify({ value: 'dark', timestamp: Date.now() }));
// Output: "Call #1: dark"
// Callback called ONLY after change
```

#### **With Immediate**
```javascript
let callCount = 0;

ReactiveUtils.watchStorage('theme', (theme, oldTheme) => {
  callCount++;
  console.log(`Call #${callCount}:`, theme);
  console.log('Old value:', oldTheme);
}, {
  immediate: true
});

// Output immediately:
// "Call #1: light" (current value)
// "Old value: null" (no previous value on first call)

// Change the value
localStorage.setItem('theme', JSON.stringify({ value: 'dark', timestamp: Date.now() }));
// Output: "Call #2: dark"
// "Old value: light"
```

#### **Apply Theme on Load**
```javascript
// Apply theme immediately when page loads
ReactiveUtils.watchStorage('theme', (theme) => {
  if (!theme) {
    theme = 'light'; // Default
  }

  document.body.className = theme + '-theme';
  loadThemeStyles(theme);
}, {
  immediate: true
});

// Theme applied immediately, no flash of wrong theme!
```

#### **Initialize UI with Current Value**
```javascript
// Load user preferences and apply immediately
ReactiveUtils.watchStorage('userPrefs', (prefs) => {
  if (!prefs) {
    prefs = getDefaultPreferences();
  }

  document.body.style.fontSize = prefs.fontSize + 'px';
  document.documentElement.lang = prefs.language;
  applyTheme(prefs.theme);
}, {
  immediate: true,
  namespace: 'app'
});

// UI configured on page load
```

#### **Conditional Immediate**
```javascript
function watchWithOptionalImmediate(key, callback, loadImmediately = false) {
  return ReactiveUtils.watchStorage(key, callback, {
    immediate: loadImmediately
  });
}

// Load immediately
watchWithOptionalImmediate('settings', applySettings, true);

// Wait for changes
watchWithOptionalImmediate('notifications', showNotification, false);
```

#### **Initialize State from Storage**
```javascript
const app = {
  theme: null,
  language: null,
  fontSize: null
};

// Initialize all settings from storage
ReactiveUtils.watchStorage('appSettings', (settings) => {
  if (!settings) {
    settings = {
      theme: 'light',
      language: 'en',
      fontSize: 14
    };
  }

  // Update app state
  Object.assign(app, settings);

  // Apply to UI
  applyAllSettings(app);
}, {
  immediate: true
});

// app state populated immediately
console.log('Initial theme:', app.theme);
```

#### **Form Auto-Restore**
```javascript
// Restore form draft on page load
ReactiveUtils.watchStorage('formDraft', (draft) => {
  if (draft) {
    // Restore form fields
    document.getElementById('title').value = draft.title || '';
    document.getElementById('content').value = draft.content || '';
    document.getElementById('category').value = draft.category || '';

    console.log('Form draft restored');
  }
}, {
  storage: 'sessionStorage',
  immediate: true  // Restore on page load
});
```

#### **Authentication State**
```javascript
// Check auth status immediately on app start
ReactiveUtils.watchStorage('authToken', (token, oldToken) => {
  if (token) {
    console.log('User authenticated');
    initializeApp();
    loadUserData();
  } else {
    console.log('User not authenticated');
    showLoginPage();
  }

  if (oldToken && !token) {
    console.log('User logged out');
    cleanupApp();
  }
}, {
  storage: 'sessionStorage',
  immediate: true  // Check auth immediately
});
```

#### **Feature Flags**
```javascript
const features = {
  darkMode: false,
  betaFeatures: false,
  analytics: false
};

// Load feature flags immediately
ReactiveUtils.watchStorage('featureFlags', (flags) => {
  if (!flags) {
    flags = {};
  }

  // Update feature state
  features.darkMode = flags.darkMode || false;
  features.betaFeatures = flags.betaFeatures || false;
  features.analytics = flags.analytics || false;

  // Apply features
  if (features.darkMode) enableDarkMode();
  if (features.betaFeatures) showBetaFeatures();
  if (features.analytics) enableAnalytics();

  console.log('Features loaded:', features);
}, {
  namespace: 'app',
  immediate: true  // Configure features immediately
});
```

#### **Shopping Cart Initialization**
```javascript
let cartItems = [];

// Load cart items immediately
ReactiveUtils.watchStorage('cartItems', (items) => {
  cartItems = items || [];

  // Update cart UI
  updateCartCount(cartItems.length);
  updateCartTotal(calculateTotal(cartItems));
  renderCartItems(cartItems);
}, {
  namespace: 'shop',
  immediate: true  // Show saved cart on page load
});
```

---

### **Immediate vs Non-Immediate**

| Scenario | Use `immediate: true` | Use `immediate: false` |
|----------|----------------------|------------------------|
| Apply settings on load | ✓ Yes | ✗ No |
| Initialize UI state | ✓ Yes | ✗ No |
| Check auth status | ✓ Yes | ✗ No |
| Restore form drafts | ✓ Yes | ✗ No |
| Load user preferences | ✓ Yes | ✗ No |
| Only react to changes | ✗ No | ✓ Yes |
| Debug mode logging | ✗ No | ✓ Yes |
| Event-driven only | ✗ No | ✓ Yes |

---

## **Combining All Options**

### **Example 1: Complete Configuration**
```javascript
const cleanup = ReactiveUtils.watchStorage('userPreferences', (prefs, oldPrefs) => {
  console.log('Preferences updated');
  console.log('Old:', oldPrefs);
  console.log('New:', prefs);

  if (prefs) {
    applyTheme(prefs.theme);
    setLanguage(prefs.language);
    setFontSize(prefs.fontSize);
  }
}, {
  storage: 'localStorage',      // Persist across sessions
  namespace: 'myApp',            // Avoid key collisions
  immediate: true                // Apply on page load
});

// Watches: "myApp:userPreferences" in localStorage
// Callback runs immediately with current value
// Then runs on every change
```

### **Example 2: Session-Based with Namespace**
```javascript
const cleanup = ReactiveUtils.watchStorage('wizardState', (state) => {
  if (state) {
    loadWizardStep(state.currentStep);
    populateFormData(state.formData);
  } else {
    startWizardFromBeginning();
  }
}, {
  storage: 'sessionStorage',     // Clear on tab close
  namespace: `wizard:${wizardId}`,  // Unique per wizard
  immediate: true                // Restore on page load
});
```

### **Example 3: Multi-Tenant with All Options**
```javascript
function createTenantWatcher(tenantId, feature) {
  return ReactiveUtils.watchStorage(feature, (data, oldData) => {
    console.log(`[${tenantId}] ${feature} changed`);

    // Update tenant-specific UI
    updateTenantUI(tenantId, feature, data);

    // Log changes
    if (oldData !== data) {
      logTenantChange(tenantId, feature, oldData, data);
    }
  }, {
    storage: 'localStorage',
    namespace: `tenant:${tenantId}`,
    immediate: true
  });
}

// Watch different features for different tenants
const cleanup1 = createTenantWatcher('acme-corp', 'settings');
const cleanup2 = createTenantWatcher('acme-corp', 'data');
const cleanup3 = createTenantWatcher('beta-inc', 'settings');
```

---

## **Common Patterns**

### **Pattern 1: Default Options**
```javascript
ReactiveUtils.watchStorage('key', callback);
// storage: 'localStorage', namespace: '', immediate: false
```

### **Pattern 2: Immediate Load**
```javascript
ReactiveUtils.watchStorage('key', callback, { immediate: true });
```

### **Pattern 3: Namespaced**
```javascript
ReactiveUtils.watchStorage('key', callback, { namespace: 'app' });
```

### **Pattern 4: Session Storage**
```javascript
ReactiveUtils.watchStorage('key', callback, { storage: 'sessionStorage' });
```

### **Pattern 5: Full Configuration**
```javascript
ReactiveUtils.watchStorage('key', callback, {
  storage: 'localStorage',
  namespace: 'app',
  immediate: true
});
```

---

## **Option Validation**

All options are validated and have safe defaults:

```javascript
// Invalid storage type - uses default
ReactiveUtils.watchStorage('key', callback, {
  storage: 'invalidType'  // Falls back to 'localStorage'
});

// Invalid immediate - uses default
ReactiveUtils.watchStorage('key', callback, {
  immediate: 'yes'  // Falls back to false
});

// Empty namespace - no prefix
ReactiveUtils.watchStorage('key', callback, {
  namespace: ''  // No prefix added
});
```

---

## **Best Practices**

1. **Use immediate for initialization**
   ```javascript
   ReactiveUtils.watchStorage('theme', applyTheme, {
     immediate: true  // Apply theme on page load
   });
   ```

2. **Use namespaces to avoid collisions**
   ```javascript
   ReactiveUtils.watchStorage('config', callback, {
     namespace: 'myApp'  // Prevents conflicts
   });
   ```

3. **Choose appropriate storage type**
   ```javascript
   // Persistent data
   { storage: 'localStorage' }

   // Session-only data
   { storage: 'sessionStorage' }
   ```

4. **Combine options appropriately**
   ```javascript
   // Good: Load user prefs immediately from persistent storage
   {
     storage: 'localStorage',
     namespace: 'app',
     immediate: true
   }
   ```

5. **Handle null values with immediate**
   ```javascript
   ReactiveUtils.watchStorage('data', (data) => {
     if (!data) {
       data = getDefaults();
     }
     applyData(data);
   }, { immediate: true });
   ```

---

## **Key Takeaways**

1. **storage**: Choose localStorage (persistent) or sessionStorage (session-only)
2. **namespace**: Prefix keys to avoid collisions and organize data
3. **immediate**: Call callback immediately with current value for initialization
4. **Defaults**: localStorage, no namespace, no immediate execution
5. **Safe**: Invalid options fall back to safe defaults
6. **Flexible**: Combine options for different use cases
7. **Performance**: Options cached, not re-evaluated
8. **Consistent**: Same options work across all storage watchers
9. **Typed**: Options validated for type safety
10. **Optional**: All options have sensible defaults

---

## **Summary**

The `watchStorage()` function accepts three configuration options that control which storage to watch, how to organize keys, and when to execute the callback. Use `storage` to choose between persistent localStorage (survives browser restarts) and sessionStorage (cleared on tab close). Use `namespace` to add a prefix to storage keys for organization, collision prevention, multi-tenancy, or library isolation. Use `immediate` to call the callback immediately with the current storage value on setup, perfect for initializing UI state, applying user preferences on page load, checking authentication status, or restoring form drafts. All options have sensible defaults (localStorage, no namespace, no immediate) and can be combined to create watchers that fit your specific use case. The options are validated and fall back to safe defaults if invalid values are provided, making the API robust and easy to use.
