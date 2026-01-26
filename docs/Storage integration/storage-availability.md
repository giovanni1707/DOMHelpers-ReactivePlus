# Storage Availability Properties - `hasLocalStorage` & `hasSessionStorage`

**Quick Start (30 seconds)**
```javascript
// Check if localStorage is available
if (ReactiveUtils.hasLocalStorage) {
  console.log('localStorage is available');
  localStorage.setItem('key', 'value');
} else {
  console.log('localStorage is NOT available');
  // Use fallback storage mechanism
}

// Check if sessionStorage is available
if (ReactiveUtils.hasSessionStorage) {
  console.log('sessionStorage is available');
  sessionStorage.setItem('key', 'value');
} else {
  console.log('sessionStorage is NOT available');
  // Use fallback storage mechanism
}

// Use with autoSave
const state = ReactiveUtils.reactive({ data: 'test' });

if (ReactiveUtils.hasLocalStorage) {
  ReactiveUtils.autoSave(state, 'data', { storage: 'localStorage' });
} else if (ReactiveUtils.hasSessionStorage) {
  ReactiveUtils.autoSave(state, 'data', { storage: 'sessionStorage' });
} else {
  console.warn('No storage available, data will not persist');
}
```

---

## **What are `hasLocalStorage` and `hasSessionStorage`?**

`hasLocalStorage` and `hasSessionStorage` are **read-only boolean properties** that indicate whether localStorage and sessionStorage are available and functional in the current browser environment.

**Key characteristics:**
- **Boolean Flags**: `true` if available, `false` if not
- **Read-Only**: Cannot be modified
- **Pre-Checked**: Checked once on module load
- **Safe Detection**: Handles quota errors and security restrictions
- **Environment Aware**: Works in browsers, Node.js, SSR
- **Error-Free**: Never throws errors

---

## **Properties**

### **`hasLocalStorage`**

**Type**: `boolean` (read-only)

**Description**:
Indicates if `localStorage` is available and functional.

**Returns**:
- `true` - localStorage is available and working
- `false` - localStorage is not available (private browsing, disabled, quota exceeded, etc.)

**Syntax**:
```javascript
if (ReactiveUtils.hasLocalStorage) {
  // Use localStorage
}
```

---

### **`hasSessionStorage`**

**Type**: `boolean` (read-only)

**Description**:
Indicates if `sessionStorage` is available and functional.

**Returns**:
- `true` - sessionStorage is available and working
- `false` - sessionStorage is not available (private browsing, disabled, quota exceeded, etc.)

**Syntax**:
```javascript
if (ReactiveUtils.hasSessionStorage) {
  // Use sessionStorage
}
```

---

## **How it works**

```javascript
// Detection implementation
function isStorageAvailable(type) {
  try {
    const storage = window[type];
    const test = '__storage_test__';
    storage.setItem(test, test);
    storage.removeItem(test);
    return true;
  } catch (e) {
    return false;
  }
}

// Pre-checked on module load
const hasLocalStorage = isStorageAvailable('localStorage');
const hasSessionStorage = isStorageAvailable('sessionStorage');

// Exported for use
ReactiveUtils.hasLocalStorage = hasLocalStorage;
ReactiveUtils.hasSessionStorage = hasSessionStorage;
```

**What happens:**
1. Module loads and runs detection tests
2. Attempts to write and remove a test key
3. Returns `true` if successful
4. Returns `false` if any error occurs
5. Values stored as constants
6. Exported as read-only properties
7. Never re-checked (cached result)

**Detects:**
- ✓ localStorage/sessionStorage disabled
- ✓ Private browsing mode restrictions
- ✓ Quota exceeded errors
- ✓ Security policy restrictions
- ✓ Browser support
- ✓ Server-side rendering (always false)

---

## **Examples**

### **Example 1: Basic Storage Detection**
```javascript
// Check localStorage
if (ReactiveUtils.hasLocalStorage) {
  console.log('✓ localStorage available');
  localStorage.setItem('app', 'data');
} else {
  console.log('✗ localStorage not available');
}

// Check sessionStorage
if (ReactiveUtils.hasSessionStorage) {
  console.log('✓ sessionStorage available');
  sessionStorage.setItem('temp', 'data');
} else {
  console.log('✗ sessionStorage not available');
}
```

### **Example 2: Conditional autoSave**
```javascript
const userPrefs = ReactiveUtils.reactive({
  theme: 'light',
  fontSize: 14
});

// Use localStorage if available
if (ReactiveUtils.hasLocalStorage) {
  ReactiveUtils.autoSave(userPrefs, 'preferences', {
    storage: 'localStorage'
  });
  console.log('Preferences will persist across sessions');
} else {
  console.warn('Preferences will be lost on page reload');
}
```

### **Example 3: Storage Fallback Chain**
```javascript
const data = ReactiveUtils.reactive({ items: [] });

// Try localStorage first, then sessionStorage, then in-memory
if (ReactiveUtils.hasLocalStorage) {
  ReactiveUtils.autoSave(data, 'appData', {
    storage: 'localStorage'
  });
  console.log('Using localStorage (persistent)');
} else if (ReactiveUtils.hasSessionStorage) {
  ReactiveUtils.autoSave(data, 'appData', {
    storage: 'sessionStorage'
  });
  console.log('Using sessionStorage (session only)');
} else {
  console.warn('No storage available - data will be lost on reload');
  // Implement custom in-memory storage or server-side sync
}
```

### **Example 4: Feature Detection**
```javascript
function initializeApp() {
  const features = {
    persistentStorage: ReactiveUtils.hasLocalStorage,
    sessionStorage: ReactiveUtils.hasSessionStorage,
    offline: ReactiveUtils.hasLocalStorage || ReactiveUtils.hasSessionStorage
  };

  console.log('App features:', features);

  if (!features.offline) {
    showWarning('Offline features not available');
  }

  return features;
}

const appFeatures = initializeApp();
```

### **Example 5: User Warning**
```javascript
if (!ReactiveUtils.hasLocalStorage) {
  showNotification({
    type: 'warning',
    message: 'localStorage is disabled. Settings will not be saved.',
    duration: 5000
  });
}

if (!ReactiveUtils.hasSessionStorage) {
  console.warn('sessionStorage disabled - some features may not work');
}
```

### **Example 6: Private Browsing Detection**
```javascript
function checkPrivateBrowsing() {
  if (!ReactiveUtils.hasLocalStorage && !ReactiveUtils.hasSessionStorage) {
    return {
      isPrivate: true,
      message: 'You appear to be in private browsing mode'
    };
  }

  if (!ReactiveUtils.hasLocalStorage && ReactiveUtils.hasSessionStorage) {
    return {
      isPrivate: true,
      message: 'Limited storage available (private browsing?)'
    };
  }

  return {
    isPrivate: false,
    message: 'Normal browsing mode'
  };
}

const privacy = checkPrivateBrowsing();
console.log(privacy.message);
```

### **Example 7: Adaptive Storage Strategy**
```javascript
class StorageManager {
  constructor() {
    this.storageType = this.detectBestStorage();
  }

  detectBestStorage() {
    if (ReactiveUtils.hasLocalStorage) {
      return 'localStorage';
    } else if (ReactiveUtils.hasSessionStorage) {
      return 'sessionStorage';
    } else {
      return 'memory';
    }
  }

  save(key, value) {
    if (this.storageType === 'memory') {
      this.memoryStore = this.memoryStore || {};
      this.memoryStore[key] = value;
    } else {
      const storage = window[this.storageType];
      storage.setItem(key, JSON.stringify(value));
    }
  }

  load(key) {
    if (this.storageType === 'memory') {
      return this.memoryStore?.[key] || null;
    } else {
      const storage = window[this.storageType];
      const item = storage.getItem(key);
      return item ? JSON.parse(item) : null;
    }
  }
}

const storage = new StorageManager();
console.log(`Using: ${storage.storageType}`);
```

### **Example 8: Server-Side Rendering (SSR)**
```javascript
// Safe for SSR environments
function initializeStorage() {
  if (typeof window === 'undefined') {
    console.log('Server-side: No storage available');
    return null;
  }

  if (ReactiveUtils.hasLocalStorage) {
    return ReactiveUtils.reactiveStorage('localStorage');
  } else if (ReactiveUtils.hasSessionStorage) {
    return ReactiveUtils.reactiveStorage('sessionStorage');
  } else {
    console.warn('Client-side: No storage available');
    return null;
  }
}

const storage = initializeStorage();

if (storage) {
  // Use storage
  storage.set('key', 'value');
}
```

### **Example 9: Graceful Degradation**
```javascript
const appState = ReactiveUtils.reactive({
  user: null,
  preferences: { theme: 'light' }
});

// Try to persist, but work without it
if (ReactiveUtils.hasLocalStorage) {
  ReactiveUtils.autoSave(appState, 'appState');
  console.log('✓ App state will persist');
} else {
  console.warn('⚠ App state will not persist - consider server-side storage');

  // Alternative: sync to server periodically
  setInterval(() => {
    fetch('/api/save-state', {
      method: 'POST',
      body: JSON.stringify(appState)
    });
  }, 30000); // Save to server every 30s
}
```

### **Example 10: Feature Flag System**
```javascript
function initializeFeatureFlags() {
  const flags = {
    offlineMode: ReactiveUtils.hasLocalStorage,
    autoSave: ReactiveUtils.hasLocalStorage || ReactiveUtils.hasSessionStorage,
    crossTabSync: ReactiveUtils.hasLocalStorage,
    sessionPersistence: ReactiveUtils.hasSessionStorage
  };

  console.log('Feature flags:', flags);

  return flags;
}

const features = initializeFeatureFlags();

// Enable features based on availability
if (features.offlineMode) {
  enableOfflineMode();
}

if (features.autoSave) {
  enableAutoSave();
}

if (features.crossTabSync) {
  enableCrossTabSync();
}
```

### **Example 11: Development vs Production**
```javascript
function setupStorage() {
  const isDev = process.env.NODE_ENV === 'development';

  if (!ReactiveUtils.hasLocalStorage) {
    const warning = 'localStorage not available';

    if (isDev) {
      console.error(warning);
      throw new Error(warning);
    } else {
      console.warn(warning);
      // Send to error tracking
      trackError({ type: 'storage', message: warning });
    }
  }
}
```

### **Example 12: Cache Strategy**
```javascript
const CacheStrategy = {
  PERSISTENT: 'localStorage',
  SESSION: 'sessionStorage',
  MEMORY: 'memory',
  NONE: 'none'
};

function getCacheStrategy() {
  if (ReactiveUtils.hasLocalStorage) {
    return CacheStrategy.PERSISTENT;
  } else if (ReactiveUtils.hasSessionStorage) {
    return CacheStrategy.SESSION;
  } else {
    return CacheStrategy.MEMORY;
  }
}

const strategy = getCacheStrategy();
console.log(`Cache strategy: ${strategy}`);

function cacheData(key, data, options = {}) {
  const { ttl = 3600, strategy = getCacheStrategy() } = options;

  switch (strategy) {
    case CacheStrategy.PERSISTENT:
      ReactiveUtils.reactiveStorage('localStorage')
        .set(key, data, { expires: ttl });
      break;

    case CacheStrategy.SESSION:
      ReactiveUtils.reactiveStorage('sessionStorage')
        .set(key, data);
      break;

    case CacheStrategy.MEMORY:
      // Use Map or object for in-memory cache
      window.__memoryCache = window.__memoryCache || new Map();
      window.__memoryCache.set(key, data);
      break;

    case CacheStrategy.NONE:
      console.warn('No caching available');
      break;
  }
}
```

### **Example 13: Progressive Enhancement**
```javascript
class ProgressiveStorage {
  constructor() {
    this.capabilities = this.detectCapabilities();
    this.strategy = this.selectStrategy();
  }

  detectCapabilities() {
    return {
      localStorage: ReactiveUtils.hasLocalStorage,
      sessionStorage: ReactiveUtils.hasSessionStorage,
      indexedDB: typeof indexedDB !== 'undefined',
      cookies: typeof document !== 'undefined'
    };
  }

  selectStrategy() {
    if (this.capabilities.localStorage) {
      return 'localStorage'; // Best option
    } else if (this.capabilities.sessionStorage) {
      return 'sessionStorage'; // OK for session data
    } else if (this.capabilities.indexedDB) {
      return 'indexedDB'; // Fallback to IndexedDB
    } else if (this.capabilities.cookies) {
      return 'cookies'; // Last resort
    } else {
      return 'none'; // No storage available
    }
  }

  save(key, value) {
    console.log(`Saving with ${this.strategy}:`, key);
    // Implement save logic for each strategy
  }
}

const storage = new ProgressiveStorage();
console.log('Storage capabilities:', storage.capabilities);
console.log('Selected strategy:', storage.strategy);
```

### **Example 14: Diagnostic Tool**
```javascript
function diagnoseStorage() {
  const report = {
    timestamp: new Date().toISOString(),
    environment: typeof window !== 'undefined' ? 'browser' : 'server',
    localStorage: {
      available: ReactiveUtils.hasLocalStorage,
      reason: ReactiveUtils.hasLocalStorage ? 'Working' : 'Not available'
    },
    sessionStorage: {
      available: ReactiveUtils.hasSessionStorage,
      reason: ReactiveUtils.hasSessionStorage ? 'Working' : 'Not available'
    },
    userAgent: typeof navigator !== 'undefined' ? navigator.userAgent : 'N/A',
    privateMode: (!ReactiveUtils.hasLocalStorage && !ReactiveUtils.hasSessionStorage) ? 'Likely' : 'Unlikely'
  };

  // Additional tests
  if (typeof window !== 'undefined') {
    try {
      const quota = navigator.storage?.estimate?.() || Promise.resolve(null);
      quota.then(estimate => {
        if (estimate) {
          report.quota = {
            usage: estimate.usage,
            quota: estimate.quota,
            percentUsed: ((estimate.usage / estimate.quota) * 100).toFixed(2) + '%'
          };
        }
        console.table(report);
      });
    } catch (e) {
      console.log('Storage diagnostic report:', report);
    }
  }

  return report;
}

// Run diagnostics
const diagnosis = diagnoseStorage();
```

### **Example 15: Conditional Component Loading**
```javascript
// React example
function StorageRequiredFeature() {
  React.useEffect(() => {
    if (!ReactiveUtils.hasLocalStorage) {
      console.error('This feature requires localStorage');
    }
  }, []);

  if (!ReactiveUtils.hasLocalStorage) {
    return (
      <div className="error">
        <h3>Storage Required</h3>
        <p>This feature requires localStorage to be enabled.</p>
        <p>Please disable private browsing or enable storage.</p>
      </div>
    );
  }

  return <ActualFeature />;
}
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
  // Use localStorage
} else if (ReactiveUtils.hasSessionStorage) {
  // Use sessionStorage
} else {
  // Use in-memory or server-side
}
```

### **Pattern 3: Warning on Missing**
```javascript
if (!ReactiveUtils.hasLocalStorage) {
  console.warn('localStorage not available');
}
```

### **Pattern 4: Both Required**
```javascript
if (ReactiveUtils.hasLocalStorage && ReactiveUtils.hasSessionStorage) {
  // Use both
}
```

### **Pattern 5: Either Required**
```javascript
if (ReactiveUtils.hasLocalStorage || ReactiveUtils.hasSessionStorage) {
  // At least one available
}
```

---

## **When Storage is Unavailable**

Storage can be unavailable in several scenarios:

### **Private Browsing Mode**
- Safari: localStorage throws on access
- Firefox: localStorage quota set to 0
- Chrome: localStorage works but cleared on exit

### **Disabled by User**
- Browser settings
- Privacy extensions
- Corporate policies

### **Quota Exceeded**
- Storage full
- Quota limits reached
- Too much data stored

### **Security Restrictions**
- Cross-origin restrictions
- Iframe restrictions
- Content Security Policy

### **Server-Side Rendering**
- No `window` object
- No browser storage APIs
- Always returns `false`

### **Unsupported Browsers**
- Very old browsers
- Feature phones
- Custom browsers

---

## **Comparison with Native Detection**

| Feature | `hasLocalStorage` | Manual try/catch |
|---------|------------------|------------------|
| Pre-checked | ✓ Yes | ✗ No |
| Error-free | ✓ Yes | ⚠ Can throw |
| Write test | ✓ Yes | ⚠ Optional |
| Cached result | ✓ Yes | ✗ Re-runs each time |
| Convenience | ✓ One property | Multiple lines |

```javascript
// hasLocalStorage - simple, cached
if (ReactiveUtils.hasLocalStorage) {
  // Use localStorage
}

// Manual detection - verbose, not cached
let hasStorage = false;
try {
  const test = '__test__';
  localStorage.setItem(test, test);
  localStorage.removeItem(test);
  hasStorage = true;
} catch (e) {
  hasStorage = false;
}

if (hasStorage) {
  // Use localStorage
}
```

---

## **When to Use**

| Scenario | Use hasLocalStorage/hasSessionStorage |
|----------|--------------------------------------|
| Check before using storage | ✓ Yes |
| Conditional features | ✓ Yes |
| Fallback logic | ✓ Yes |
| SSR compatibility | ✓ Yes |
| Feature detection | ✓ Yes |
| User warnings | ✓ Yes |
| Always assume available | ✗ No |

---

## **Best Practices**

1. **Always check before using storage**
   ```javascript
   if (ReactiveUtils.hasLocalStorage) {
     localStorage.setItem('key', 'value');
   }
   ```

2. **Provide fallbacks**
   ```javascript
   if (!ReactiveUtils.hasLocalStorage) {
     // Use alternative storage or warn user
   }
   ```

3. **Handle SSR gracefully**
   ```javascript
   if (typeof window !== 'undefined' && ReactiveUtils.hasLocalStorage) {
     // Use storage
   }
   ```

4. **Warn users when unavailable**
   ```javascript
   if (!ReactiveUtils.hasLocalStorage) {
     showWarning('Storage disabled - data will not persist');
   }
   ```

5. **Use with autoSave**
   ```javascript
   if (ReactiveUtils.hasLocalStorage) {
     ReactiveUtils.autoSave(state, 'key');
   }
   ```

6. **Don't assume both are available**
   ```javascript
   // Good: check each separately
   const useLocal = ReactiveUtils.hasLocalStorage;
   const useSession = ReactiveUtils.hasSessionStorage;

   // Bad: assume both
   // May fail in some environments
   ```

---

## **Alternative Detection Method**

If you need to check dynamically at runtime (rare):

```javascript
// Available as a function too
const isAvailable = ReactiveUtils.isStorageAvailable('localStorage');

if (isAvailable) {
  console.log('localStorage is available');
}
```

**Note**: Using the pre-checked properties is preferred for performance.

---

## **Access Locations**

These properties are available in multiple locations:

```javascript
// ReactiveUtils namespace (recommended)
ReactiveUtils.hasLocalStorage
ReactiveUtils.hasSessionStorage

// ReactiveStorage namespace
ReactiveStorage.hasLocalStorage
ReactiveStorage.hasSessionStorage

// Global (if exposed)
hasLocalStorage
hasSessionStorage
```

---

## **Key Takeaways**

1. **Boolean Flags**: Simple true/false indicating storage availability
2. **Pre-Checked**: Tested once on module load for performance
3. **Safe Detection**: Handles all error cases gracefully
4. **Read-Only**: Cannot be modified at runtime
5. **SSR Safe**: Works in server-side environments (returns false)
6. **No Errors**: Never throws exceptions
7. **Cached**: Result stored, not re-checked
8. **Write Test**: Actually tests writing, not just existence
9. **Fallback Ready**: Easy to implement fallback logic
10. **Universal**: Works across all browsers and environments

---

## **Summary**

`hasLocalStorage` and `hasSessionStorage` are read-only boolean properties that indicate whether localStorage and sessionStorage are available and functional in the current browser environment. These properties are checked once when the module loads by attempting to write and remove a test key, returning `true` if successful or `false` if any error occurs (including quota exceeded, private browsing restrictions, disabled storage, or server-side environments). Use these properties to safely detect storage availability before using storage APIs, implement graceful fallbacks when storage is unavailable, show appropriate user warnings, enable or disable features based on storage capabilities, and ensure server-side rendering compatibility. The pre-checked nature of these properties makes them performant for repeated checks, and their error-free design makes them safe to use anywhere in your code. Always check these properties before using browser storage to ensure your application works correctly across all environments and user configurations.
