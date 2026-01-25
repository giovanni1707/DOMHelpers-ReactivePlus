# `isStorageAvailable(type)` - Check Storage Availability

**Quick Start (30 seconds)**
```javascript
// Check if localStorage is available
if (ReactiveUtils.isStorageAvailable('localStorage')) {
  console.log('localStorage is available');
  const state = ReactiveUtils.reactive({ data: [] });
  ReactiveUtils.autoSave(state, 'data');
} else {
  console.log('localStorage is NOT available');
  // Use in-memory storage or show warning
}

// Check if sessionStorage is available
if (ReactiveUtils.isStorageAvailable('sessionStorage')) {
  console.log('sessionStorage is available');
} else {
  console.log('sessionStorage is NOT available');
}
```

---

## **What is `isStorageAvailable(type)`?**

`isStorageAvailable()` is a **utility function** that checks whether a specific storage type (localStorage or sessionStorage) is available and functional in the current environment.

**Key characteristics:**
- **Availability Check**: Tests if storage is available
- **Functionality Test**: Verifies storage actually works
- **Privacy Mode Detection**: Detects private/incognito mode
- **Quota Check**: Tests write capability
- **Browser Support**: Detects browser support
- **No Side Effects**: Cleans up test data

---

## **Syntax**

```javascript
ReactiveUtils.isStorageAvailable(type)
```

### **Parameters**
- **`type`** (string): Storage type to check - `'localStorage'` or `'sessionStorage'`

### **Returns**
- **Type**: `boolean`
- **`true`**: Storage is available and functional
- **`false`**: Storage is unavailable, disabled, or non-functional

---

## **How it works**

```javascript
function isStorageAvailable(type) {
  try {
    const storage = window[type];
    const testKey = '__storage_test__';

    // Try to write
    storage.setItem(testKey, testKey);

    // Try to read
    const value = storage.getItem(testKey);

    // Clean up
    storage.removeItem(testKey);

    // Verify write/read worked
    return value === testKey;
  } catch (e) {
    // Storage unavailable, disabled, or quota exceeded
    return false;
  }
}
```

**What happens:**
1. Attempts to access window.localStorage or window.sessionStorage
2. Tries to write a test value
3. Tries to read the test value back
4. Removes the test value
5. Returns true if all operations succeeded
6. Returns false if any operation failed

---

## **Examples**

### **Example 1: Basic Check**
```javascript
if (ReactiveUtils.isStorageAvailable('localStorage')) {
  console.log('✓ localStorage works');
} else {
  console.log('✗ localStorage unavailable');
}
```

### **Example 2: Conditional Storage Setup**
```javascript
const state = ReactiveUtils.reactive({
  data: []
});

if (ReactiveUtils.isStorageAvailable('localStorage')) {
  ReactiveUtils.autoSave(state, 'data');
  console.log('Using persistent storage');
} else {
  console.warn('localStorage unavailable, using memory only');
  // Data won't persist across page reloads
}
```

### **Example 3: Fallback to SessionStorage**
```javascript
function setupStorage(state, key) {
  if (ReactiveUtils.isStorageAvailable('localStorage')) {
    ReactiveUtils.autoSave(state, key, {
      storage: 'localStorage'
    });
    console.log('Using localStorage');
  } else if (ReactiveUtils.isStorageAvailable('sessionStorage')) {
    ReactiveUtils.autoSave(state, key, {
      storage: 'sessionStorage'
    });
    console.log('Using sessionStorage');
  } else {
    console.error('No storage available');
  }
}
```

### **Example 4: User Warning**
```javascript
function checkStorage() {
  if (!ReactiveUtils.isStorageAvailable('localStorage')) {
    showWarning(
      'Storage is disabled. ' +
      'Your preferences will not be saved. ' +
      'Please enable cookies/storage in your browser.'
    );
    return false;
  }
  return true;
}
```

### **Example 5: Private Mode Detection**
```javascript
function detectPrivateMode() {
  const isPrivate = !ReactiveUtils.isStorageAvailable('localStorage');

  if (isPrivate) {
    console.log('Likely in private/incognito mode');
    document.getElementById('privateWarning').style.display = 'block';
  }

  return isPrivate;
}
```

### **Example 6: Storage Feature Detection**
```javascript
const features = {
  localStorage: ReactiveUtils.isStorageAvailable('localStorage'),
  sessionStorage: ReactiveUtils.isStorageAvailable('sessionStorage')
};

console.log('Storage Features:', features);

if (!features.localStorage) {
  disableFeature('offline-mode');
  disableFeature('save-preferences');
}
```

### **Example 7: Initialize App**
```javascript
function initializeApp() {
  console.log('Checking storage availability...');

  const hasLocal = ReactiveUtils.isStorageAvailable('localStorage');
  const hasSession = ReactiveUtils.isStorageAvailable('sessionStorage');

  console.log(`localStorage: ${hasLocal ? '✓' : '✗'}`);
  console.log(`sessionStorage: ${hasSession ? '✓' : '✗'}`);

  if (!hasLocal && !hasSession) {
    showFatalError('Storage not available. App cannot function.');
    return false;
  }

  setupApp(hasLocal ? 'localStorage' : 'sessionStorage');
  return true;
}
```

### **Example 8: Dynamic Storage Selection**
```javascript
function getAvailableStorage() {
  if (ReactiveUtils.isStorageAvailable('localStorage')) {
    return 'localStorage';
  } else if (ReactiveUtils.isStorageAvailable('sessionStorage')) {
    return 'sessionStorage';
  } else {
    return null;
  }
}

const storageType = getAvailableStorage();

if (storageType) {
  ReactiveUtils.autoSave(state, 'data', { storage: storageType });
}
```

### **Example 9: Test Before Large Save**
```javascript
function saveLargeData(data) {
  if (!ReactiveUtils.isStorageAvailable('localStorage')) {
    console.error('Cannot save: localStorage unavailable');
    return false;
  }

  try {
    localStorage.setItem('largeData', JSON.stringify(data));
    return true;
  } catch (e) {
    console.error('Save failed:', e.message);
    return false;
  }
}
```

### **Example 10: Environment Check**
```javascript
function checkEnvironment() {
  const report = {
    browser: navigator.userAgent,
    localStorage: ReactiveUtils.isStorageAvailable('localStorage'),
    sessionStorage: ReactiveUtils.isStorageAvailable('sessionStorage'),
    cookies: navigator.cookieEnabled
  };

  console.table(report);

  if (!report.localStorage) {
    logToServer('Storage unavailable', report);
  }

  return report;
}
```

---

## **Common Patterns**

### **Pattern 1: Check Before Use**
```javascript
if (ReactiveUtils.isStorageAvailable('localStorage')) {
  // Use localStorage
}
```

### **Pattern 2: Fallback Chain**
```javascript
if (ReactiveUtils.isStorageAvailable('localStorage')) {
  useLocalStorage();
} else if (ReactiveUtils.isStorageAvailable('sessionStorage')) {
  useSessionStorage();
} else {
  useMemory();
}
```

### **Pattern 3: User Warning**
```javascript
if (!ReactiveUtils.isStorageAvailable('localStorage')) {
  showWarning('Storage disabled');
}
```

### **Pattern 4: Feature Detection**
```javascript
const canPersist = ReactiveUtils.isStorageAvailable('localStorage');
```

### **Pattern 5: Conditional Setup**
```javascript
const storageType = ReactiveUtils.isStorageAvailable('localStorage')
  ? 'localStorage'
  : 'sessionStorage';
```

---

## **When to Use**

| Scenario | Use isStorageAvailable() |
|----------|-------------------------|
| Before using storage | ✓ Yes |
| Private mode detection | ✓ Yes |
| Feature detection | ✓ Yes |
| Fallback logic | ✓ Yes |
| User warnings | ✓ Yes |
| Environment checks | ✓ Yes |
| After storage is set up | ✗ No (unnecessary) |

---

## **Common Failure Scenarios**

### **1. Private/Incognito Mode**
```javascript
// Safari private mode blocks storage
const available = ReactiveUtils.isStorageAvailable('localStorage');
// Returns false in Safari private mode
```

### **2. Cookies Disabled**
```javascript
// Some browsers tie storage to cookie settings
if (!ReactiveUtils.isStorageAvailable('localStorage')) {
  console.log('Check cookie settings');
}
```

### **3. Quota Exceeded**
```javascript
// Storage full
const available = ReactiveUtils.isStorageAvailable('localStorage');
// May return false if storage is completely full
```

### **4. Old Browsers**
```javascript
// IE < 8, very old browsers
if (!ReactiveUtils.isStorageAvailable('localStorage')) {
  console.log('Browser too old');
}
```

### **5. Third-Party Context**
```javascript
// iframes may have restricted storage
if (!ReactiveUtils.isStorageAvailable('localStorage')) {
  console.log('May be in restricted iframe');
}
```

---

## **vs. Feature Detection**

| Check | isStorageAvailable() | typeof localStorage !== 'undefined' |
|-------|---------------------|-----------------------------------|
| Exists | ✓ | ✓ |
| Functional | ✓ | ✗ |
| Quota Check | ✓ | ✗ |
| Private Mode | ✓ Detects | ✗ May miss |

```javascript
// Basic existence check (insufficient)
if (typeof localStorage !== 'undefined') {
  // Exists but may not work
}

// Proper functional check
if (ReactiveUtils.isStorageAvailable('localStorage')) {
  // Exists AND works
}
```

---

## **Best Practices**

1. **Always check before using storage**
   ```javascript
   if (ReactiveUtils.isStorageAvailable('localStorage')) {
     setupStorage();
   }
   ```

2. **Provide fallbacks**
   ```javascript
   const storage = ReactiveUtils.isStorageAvailable('localStorage')
     ? 'localStorage'
     : null;
   ```

3. **Warn users**
   ```javascript
   if (!ReactiveUtils.isStorageAvailable('localStorage')) {
     showWarning('Please enable storage in browser settings');
   }
   ```

4. **Log issues**
   ```javascript
   if (!ReactiveUtils.isStorageAvailable('localStorage')) {
     logError('Storage unavailable');
   }
   ```

5. **Check both types**
   ```javascript
   const hasLocal = ReactiveUtils.isStorageAvailable('localStorage');
   const hasSession = ReactiveUtils.isStorageAvailable('sessionStorage');
   ```

6. **Use with autoSave**
   ```javascript
   if (ReactiveUtils.isStorageAvailable('localStorage')) {
     ReactiveUtils.autoSave(state, 'key');
   }
   ```

---

## **Convenience Properties**

Instead of calling the function, you can use these properties:

```javascript
// These properties use isStorageAvailable() internally
ReactiveUtils.hasLocalStorage    // boolean
ReactiveUtils.hasSessionStorage  // boolean

// Equivalent to:
ReactiveUtils.isStorageAvailable('localStorage')
ReactiveUtils.isStorageAvailable('sessionStorage')
```

See: `hasLocalStorage` and `hasSessionStorage` documentation.

---

## **Key Takeaways**

1. **Functional Test**: Tests if storage actually works
2. **Returns Boolean**: true if available and functional
3. **Privacy Detection**: Detects private/incognito mode
4. **No Side Effects**: Cleans up test data
5. **Browser Support**: Checks browser compatibility
6. **Quota Check**: Tests write capability
7. **Essential**: Use before storage operations
8. **Fallback**: Enable fallback strategies
9. **User Experience**: Warn users about limitations
10. **Best Practice**: Always check availability first

---

## **Summary**

`isStorageAvailable(type)` is a utility function that checks whether a specific storage type (localStorage or sessionStorage) is available and functional in the current environment. Unlike simple existence checks, it actually tests the storage by writing and reading a test value, ensuring that storage is not only present but actually works. This catches scenarios where storage exists but is disabled (private/incognito mode), where quota is exceeded, where cookies are disabled, or where storage is restricted in third-party contexts. Use `isStorageAvailable()` before using any storage functionality to implement fallback strategies, warn users about storage limitations, detect private browsing mode, or handle browser compatibility issues. The function returns true only if storage is fully functional, making it a reliable way to detect storage capabilities across different browsers and contexts. Always check storage availability before calling autoSave() or using reactiveStorage() to ensure your app handles storage unavailability gracefully.
