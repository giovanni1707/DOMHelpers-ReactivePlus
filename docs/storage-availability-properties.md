# Storage Availability Properties - `hasLocalStorage` & `hasSessionStorage`

**Quick Start (30 seconds)**
```javascript
// Check localStorage availability
if (ReactiveUtils.hasLocalStorage) {
  console.log('✓ localStorage is available');
  const state = ReactiveUtils.reactive({ data: [] });
  ReactiveUtils.autoSave(state, 'data');
} else {
  console.log('✗ localStorage is NOT available');
  showWarning('Storage disabled');
}

// Check sessionStorage availability
if (ReactiveUtils.hasSessionStorage) {
  console.log('✓ sessionStorage is available');
} else {
  console.log('✗ sessionStorage is NOT available');
}

// Use both
console.log('Storage status:', {
  localStorage: ReactiveUtils.hasLocalStorage,
  sessionStorage: ReactiveUtils.hasSessionStorage
});
```

---

## **What are `hasLocalStorage` & `hasSessionStorage`?**

`hasLocalStorage` and `hasSessionStorage` are **boolean properties** that indicate whether localStorage and sessionStorage are available and functional in the current environment.

**Key characteristics:**
- **Read-Only**: Boolean properties (not functions)
- **Pre-Computed**: Evaluated once at initialization
- **Convenience**: No need to call function
- **Reliable**: Uses `isStorageAvailable()` internally
- **Fast**: No runtime overhead

---

## **Syntax**

```javascript
// Properties (no parentheses)
ReactiveUtils.hasLocalStorage
ReactiveUtils.hasSessionStorage
```

### **Type**
- **`hasLocalStorage`**: `boolean`
- **`hasSessionStorage`**: `boolean`

### **Values**
- **`true`**: Storage is available and functional
- **`false`**: Storage is unavailable, disabled, or non-functional

---

## **How it works**

```javascript
// Initialized once when library loads
const hasLocalStorage = isStorageAvailable('localStorage');
const hasSessionStorage = isStorageAvailable('sessionStorage');

// Available as properties
ReactiveUtils.hasLocalStorage = hasLocalStorage;
ReactiveUtils.hasSessionStorage = hasSessionStorage;
```

**What happens:**
1. Library calls `isStorageAvailable()` for both types at initialization
2. Results are stored as boolean properties
3. Properties are available for immediate use
4. No need to call functions repeatedly

---

## **Examples**

### **Example 1: Simple Check**
```javascript
if (ReactiveUtils.hasLocalStorage) {
  console.log('localStorage works!');
}

if (ReactiveUtils.hasSessionStorage) {
  console.log('sessionStorage works!');
}
```

### **Example 2: Conditional Setup**
```javascript
const state = ReactiveUtils.reactive({
  preferences: {}
});

if (ReactiveUtils.hasLocalStorage) {
  ReactiveUtils.autoSave(state, 'preferences', {
    storage: 'localStorage'
  });
  console.log('Preferences will persist across sessions');
} else if (ReactiveUtils.hasSessionStorage) {
  ReactiveUtils.autoSave(state, 'preferences', {
    storage: 'sessionStorage'
  });
  console.log('Preferences will persist during session only');
} else {
  console.warn('No storage available');
}
```

### **Example 3: Feature Flags**
```javascript
const features = {
  offlineMode: ReactiveUtils.hasLocalStorage,
  sessionPersistence: ReactiveUtils.hasSessionStorage,
  caching: ReactiveUtils.hasLocalStorage || ReactiveUtils.hasSessionStorage
};

console.log('Available features:', features);

if (features.offlineMode) {
  enableOfflineMode();
}
```

### **Example 4: User Warning**
```javascript
function checkStorageAndWarn() {
  if (!ReactiveUtils.hasLocalStorage) {
    showWarning(
      'localStorage is disabled. ' +
      'Your preferences will not be saved across sessions.'
    );
  }

  if (!ReactiveUtils.hasSessionStorage) {
    showWarning(
      'sessionStorage is disabled. ' +
      'Some features may not work correctly.'
    );
  }
}
```

### **Example 5: Storage Status Dashboard**
```javascript
function showStorageStatus() {
  const status = {
    'localStorage': ReactiveUtils.hasLocalStorage ? '✓ Available' : '✗ Unavailable',
    'sessionStorage': ReactiveUtils.hasSessionStorage ? '✓ Available' : '✗ Unavailable'
  };

  console.table(status);

  document.getElementById('storageStatus').innerHTML = `
    <div>
      <p>localStorage: ${status.localStorage}</p>
      <p>sessionStorage: ${status.sessionStorage}</p>
    </div>
  `;
}
```

### **Example 6: Fallback Strategy**
```javascript
function selectStorageType() {
  if (ReactiveUtils.hasLocalStorage) {
    return 'localStorage';
  } else if (ReactiveUtils.hasSessionStorage) {
    return 'sessionStorage';
  } else {
    return null; // Use memory-only
  }
}

const storageType = selectStorageType();

if (storageType) {
  ReactiveUtils.autoSave(appState, 'app', { storage: storageType });
} else {
  console.warn('No persistent storage available');
  enableMemoryMode();
}
```

### **Example 7: Private Mode Detection**
```javascript
function detectPrivateBrowsing() {
  const isPrivate = !ReactiveUtils.hasLocalStorage;

  if (isPrivate) {
    console.log('Likely in private/incognito mode');

    document.body.classList.add('private-mode');
    document.getElementById('privateModeWarning').style.display = 'block';
  }

  return isPrivate;
}
```

### **Example 8: Initialize App with Storage Check**
```javascript
function initApp() {
  console.log('=== Storage Availability ===');
  console.log(`localStorage:    ${ReactiveUtils.hasLocalStorage ? '✓' : '✗'}`);
  console.log(`sessionStorage:  ${ReactiveUtils.hasSessionStorage ? '✓' : '✗'}`);

  if (!ReactiveUtils.hasLocalStorage && !ReactiveUtils.hasSessionStorage) {
    showFatalError('No storage available. App cannot function.');
    return false;
  }

  setupApp();
  return true;
}
```

### **Example 9: Conditional Feature Loading**
```javascript
const app = {
  features: {
    persistence: ReactiveUtils.hasLocalStorage,
    sessionCache: ReactiveUtils.hasSessionStorage,
    offlineMode: ReactiveUtils.hasLocalStorage
  },

  init() {
    if (this.features.persistence) {
      this.enablePersistence();
    }

    if (this.features.sessionCache) {
      this.enableSessionCache();
    }

    if (!this.features.offlineMode) {
      this.disableOfflineFeatures();
    }
  }
};
```

### **Example 10: Debug Panel**
```javascript
function createDebugPanel() {
  const panel = document.createElement('div');
  panel.id = 'debugPanel';
  panel.innerHTML = `
    <h3>Storage Debug Info</h3>
    <table>
      <tr>
        <td>localStorage:</td>
        <td>${ReactiveUtils.hasLocalStorage ? '✓ Available' : '✗ Unavailable'}</td>
      </tr>
      <tr>
        <td>sessionStorage:</td>
        <td>${ReactiveUtils.hasSessionStorage ? '✓ Available' : '✗ Unavailable'}</td>
      </tr>
      <tr>
        <td>Cookies:</td>
        <td>${navigator.cookieEnabled ? '✓ Enabled' : '✗ Disabled'}</td>
      </tr>
    </table>
  `;

  document.body.appendChild(panel);
}
```

### **Example 11: Graceful Degradation**
```javascript
class DataStore {
  constructor() {
    this.storageType = this.detectStorage();
    this.useMemory = !this.storageType;
    this.memoryStore = {};
  }

  detectStorage() {
    if (ReactiveUtils.hasLocalStorage) {
      return 'localStorage';
    } else if (ReactiveUtils.hasSessionStorage) {
      return 'sessionStorage';
    }
    return null;
  }

  save(key, value) {
    if (this.useMemory) {
      this.memoryStore[key] = value;
    } else {
      window[this.storageType].setItem(key, JSON.stringify(value));
    }
  }

  load(key) {
    if (this.useMemory) {
      return this.memoryStore[key];
    } else {
      const item = window[this.storageType].getItem(key);
      return item ? JSON.parse(item) : null;
    }
  }
}
```

### **Example 12: Conditional Imports**
```javascript
async function loadApp() {
  console.log('Checking storage...');

  if (ReactiveUtils.hasLocalStorage) {
    // Load full offline-capable version
    const { OfflineApp } = await import('./offline-app.js');
    return new OfflineApp();
  } else if (ReactiveUtils.hasSessionStorage) {
    // Load session-only version
    const { SessionApp } = await import('./session-app.js');
    return new SessionApp();
  } else {
    // Load minimal version
    const { MinimalApp } = await import('./minimal-app.js');
    return new MinimalApp();
  }
}
```

### **Example 13: Settings Validation**
```javascript
function validateSettings(settings) {
  if (settings.enablePersistence && !ReactiveUtils.hasLocalStorage) {
    console.warn('Persistence requested but localStorage unavailable');
    settings.enablePersistence = false;
  }

  if (settings.enableSessionCache && !ReactiveUtils.hasSessionStorage) {
    console.warn('Session cache requested but sessionStorage unavailable');
    settings.enableSessionCache = false;
  }

  return settings;
}
```

### **Example 14: Analytics**
```javascript
function trackStorageCapabilities() {
  const capabilities = {
    hasLocalStorage: ReactiveUtils.hasLocalStorage,
    hasSessionStorage: ReactiveUtils.hasSessionStorage,
    userAgent: navigator.userAgent,
    cookiesEnabled: navigator.cookieEnabled
  };

  // Send to analytics
  analytics.track('storage_capabilities', capabilities);

  // Log unusual scenarios
  if (!capabilities.hasLocalStorage && !capabilities.hasSessionStorage) {
    analytics.track('no_storage_available', capabilities);
  }
}
```

### **Example 15: Configuration**
```javascript
const config = {
  storage: {
    type: ReactiveUtils.hasLocalStorage ? 'localStorage' :
          ReactiveUtils.hasSessionStorage ? 'sessionStorage' :
          'memory',

    available: ReactiveUtils.hasLocalStorage || ReactiveUtils.hasSessionStorage,

    features: {
      persistence: ReactiveUtils.hasLocalStorage,
      sessionOnly: ReactiveUtils.hasSessionStorage && !ReactiveUtils.hasLocalStorage,
      degraded: !ReactiveUtils.hasLocalStorage && !ReactiveUtils.hasSessionStorage
    }
  }
};

console.log('App configuration:', config);
```

---

## **Common Patterns**

### **Pattern 1: Simple Check**
```javascript
if (ReactiveUtils.hasLocalStorage) {
  // Use localStorage
}
```

### **Pattern 2: Fallback Chain**
```javascript
if (ReactiveUtils.hasLocalStorage) {
  useLocalStorage();
} else if (ReactiveUtils.hasSessionStorage) {
  useSessionStorage();
} else {
  useMemory();
}
```

### **Pattern 3: Both Required**
```javascript
if (ReactiveUtils.hasLocalStorage && ReactiveUtils.hasSessionStorage) {
  enableAllFeatures();
}
```

### **Pattern 4: Either Available**
```javascript
if (ReactiveUtils.hasLocalStorage || ReactiveUtils.hasSessionStorage) {
  enableCaching();
}
```

### **Pattern 5: Status Object**
```javascript
const storage = {
  local: ReactiveUtils.hasLocalStorage,
  session: ReactiveUtils.hasSessionStorage
};
```

---

## **When to Use**

| Scenario | Use Properties |
|----------|---------------|
| Quick availability check | ✓ Yes |
| Feature detection | ✓ Yes |
| Conditional initialization | ✓ Yes |
| Fallback logic | ✓ Yes |
| User warnings | ✓ Yes |
| Configuration | ✓ Yes |
| Runtime checks | ✓ Yes |
| Need detailed info | ✗ No (use isStorageAvailable()) |

---

## **vs. `isStorageAvailable()`**

| Feature | Properties | Function |
|---------|-----------|----------|
| Type | Boolean properties | Function call |
| Syntax | `hasLocalStorage` | `isStorageAvailable('localStorage')` |
| Evaluated | At initialization | On each call |
| Performance | Faster (pre-computed) | Slower (runs test) |
| Use Case | Quick checks | Dynamic checks |

```javascript
// Properties (recommended for most cases)
if (ReactiveUtils.hasLocalStorage) {
  // Fast, pre-computed
}

// Function (use for dynamic/runtime checks)
if (ReactiveUtils.isStorageAvailable('localStorage')) {
  // Tests storage each time
}
```

---

## **Browser Support**

Both properties detect storage support in:
- ✓ Modern browsers (Chrome, Firefox, Safari, Edge)
- ✓ Mobile browsers (iOS Safari, Android Chrome)
- ✓ Older browsers (with storage API)
- ✗ Very old browsers (IE < 8)

**Special Cases:**
- **Private/Incognito Mode**: Properties return `false` in Safari private mode
- **Cookies Disabled**: May affect storage in some browsers
- **Third-Party Context**: May return `false` in restricted iframes
- **Quota Exceeded**: Returns `false` if storage is completely full

---

## **Best Practices**

1. **Check before using storage**
   ```javascript
   if (ReactiveUtils.hasLocalStorage) {
     ReactiveUtils.autoSave(state, 'key');
   }
   ```

2. **Implement fallbacks**
   ```javascript
   const storage = ReactiveUtils.hasLocalStorage ? 'localStorage' :
                   ReactiveUtils.hasSessionStorage ? 'sessionStorage' :
                   null;
   ```

3. **Warn users**
   ```javascript
   if (!ReactiveUtils.hasLocalStorage) {
     showWarning('Storage disabled');
   }
   ```

4. **Log capabilities**
   ```javascript
   console.log('Storage:', {
     local: ReactiveUtils.hasLocalStorage,
     session: ReactiveUtils.hasSessionStorage
   });
   ```

5. **Enable graceful degradation**
   ```javascript
   if (!ReactiveUtils.hasLocalStorage && !ReactiveUtils.hasSessionStorage) {
     useMemoryMode();
   }
   ```

6. **Use for feature flags**
   ```javascript
   const features = {
     offline: ReactiveUtils.hasLocalStorage
   };
   ```

---

## **Key Takeaways**

1. **Boolean Properties**: Not functions, no parentheses
2. **Pre-Computed**: Evaluated once at initialization
3. **Convenience**: Easier than calling function
4. **Reliable**: Uses `isStorageAvailable()` internally
5. **Fast**: No runtime overhead
6. **Read-Only**: Cannot be modified
7. **Browser Support**: Detects all browser scenarios
8. **Private Mode**: Detects private/incognito mode
9. **Fallback**: Enable fallback strategies
10. **Best Practice**: Check before storage operations

---

## **Summary**

`hasLocalStorage` and `hasSessionStorage` are boolean properties that indicate whether localStorage and sessionStorage are available and functional in the current environment. These properties are pre-computed at library initialization using `isStorageAvailable()`, making them fast and convenient for repeated checks throughout your application. They return `true` if the respective storage type is available and fully functional, or `false` if storage is unavailable, disabled (private mode), blocked (cookies disabled), or non-functional (quota exceeded). Use these properties for quick availability checks before calling `autoSave()` or `reactiveStorage()`, implementing fallback strategies when storage is unavailable, detecting private/incognito browsing mode, conditionally enabling features based on storage support, and warning users about storage limitations. These properties are preferred over calling `isStorageAvailable()` repeatedly since they're pre-computed and have no runtime overhead, though the function remains useful for dynamic runtime checks when needed.
