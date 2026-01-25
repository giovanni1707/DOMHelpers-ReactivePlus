# `$storageInfo()` - Get Storage Information

**Quick Start (30 seconds)**
```javascript
const state = ReactiveUtils.reactive({
  data: 'some content'
});

ReactiveUtils.autoSave(state, 'myData', {
  storage: 'localStorage',
  namespace: 'app',
  debounce: 1000
});

// Get storage information
const info = state.$storageInfo();
console.log(info);
// {
//   key: 'myData',
//   fullKey: 'app:myData',
//   storage: 'localStorage',
//   namespace: 'app',
//   exists: true,
//   size: 156,
//   autoSaveEnabled: true,
//   debounce: 1000
// }
```

---

## **What is `$storageInfo()`?**

`$storageInfo()` is a **method added to reactive state objects** by `autoSave()` that returns detailed information about the storage configuration and current state.

**Key characteristics:**
- **Returns Object**: Detailed configuration information
- **Non-Destructive**: Read-only operation
- **Current State**: Reflects current auto-save status
- **Size Information**: Approximate storage size in bytes
- **Existence Check**: Indicates if data exists in storage
- **Debug Helper**: Useful for debugging and monitoring

---

## **Syntax**

```javascript
state.$storageInfo()
```

### **Parameters**
- None

### **Returns**
- **Type**: `Object` with the following properties:

```javascript
{
  key: string,              // Storage key
  fullKey: string,          // Full key with namespace
  storage: string,          // 'localStorage' or 'sessionStorage'
  namespace: string,        // Namespace prefix
  exists: boolean,          // Whether data exists in storage
  size: number,             // Approximate size in bytes
  autoSaveEnabled: boolean, // Whether auto-save is currently enabled
  debounce: number          // Debounce delay in milliseconds
}
```

---

## **How it works**

```javascript
// When autoSave() is called
ReactiveUtils.autoSave(state, 'key', options);

// This method is added to state:
state.$storageInfo = function() {
  const fullKey = namespace ? `${namespace}:${key}` : key;
  const exists = store.has(key);
  let size = 0;

  if (exists) {
    const item = storage.getItem(fullKey);
    size = item ? item.length : 0;
  }

  return {
    key: key,
    fullKey: fullKey,
    storage: options.storage || 'localStorage',
    namespace: namespace || '',
    exists: exists,
    size: size,
    autoSaveEnabled: autoSaveEnabled,
    debounce: options.debounce || 0
  };
};
```

**What happens:**
1. Constructs full storage key with namespace
2. Checks if data exists in storage
3. Calculates approximate size if exists
4. Returns object with all configuration details
5. Includes current auto-save enabled state

---

## **Examples**

### **Example 1: Basic Info**
```javascript
const state = ReactiveUtils.reactive({ value: 123 });
ReactiveUtils.autoSave(state, 'data');

const info = state.$storageInfo();
console.log(`Stored as: ${info.fullKey}`);
console.log(`Storage type: ${info.storage}`);
console.log(`Size: ${info.size} bytes`);
```

### **Example 2: Debug Panel**
```javascript
const state = ReactiveUtils.reactive({ data: [] });
ReactiveUtils.autoSave(state, 'appData', {
  namespace: 'myApp',
  debounce: 1000
});

function showDebugInfo() {
  const info = state.$storageInfo();

  console.group('Storage Debug Info');
  console.log('Key:', info.key);
  console.log('Full Key:', info.fullKey);
  console.log('Storage:', info.storage);
  console.log('Exists:', info.exists);
  console.log('Size:', info.size, 'bytes');
  console.log('Auto-Save:', info.autoSaveEnabled ? 'Enabled' : 'Disabled');
  console.log('Debounce:', info.debounce, 'ms');
  console.groupEnd();
}
```

### **Example 3: Check Storage Usage**
```javascript
const states = {
  user: ReactiveUtils.reactive({}),
  prefs: ReactiveUtils.reactive({}),
  cache: ReactiveUtils.reactive({})
};

ReactiveUtils.autoSave(states.user, 'user');
ReactiveUtils.autoSave(states.prefs, 'prefs');
ReactiveUtils.autoSave(states.cache, 'cache');

function getTotalStorageUsage() {
  const userInfo = states.user.$storageInfo();
  const prefsInfo = states.prefs.$storageInfo();
  const cacheInfo = states.cache.$storageInfo();

  const total = userInfo.size + prefsInfo.size + cacheInfo.size;

  console.log(`Total storage: ${total} bytes`);
  return total;
}
```

### **Example 4: Monitor Auto-Save Status**
```javascript
const editor = ReactiveUtils.reactive({ content: '' });
ReactiveUtils.autoSave(editor, 'document');

function updateStatusIndicator() {
  const info = editor.$storageInfo();

  const indicator = document.getElementById('autoSaveStatus');
  indicator.textContent = info.autoSaveEnabled
    ? `✓ Auto-save (${info.debounce}ms delay)`
    : '✗ Auto-save disabled';
}

setInterval(updateStatusIndicator, 1000);
```

### **Example 5: Validate Configuration**
```javascript
const state = ReactiveUtils.reactive({ data: null });
ReactiveUtils.autoSave(state, 'data', {
  namespace: 'app',
  storage: 'localStorage'
});

function validateStorageConfig() {
  const info = state.$storageInfo();

  if (info.storage !== 'localStorage') {
    console.warn('Expected localStorage');
  }

  if (info.namespace !== 'app') {
    console.warn('Expected namespace "app"');
  }

  console.log('Configuration valid');
}
```

### **Example 6: Storage Report**
```javascript
const state = ReactiveUtils.reactive({ items: [] });
ReactiveUtils.autoSave(state, 'items', { namespace: 'shop' });

function generateStorageReport() {
  const info = state.$storageInfo();

  return {
    location: `${info.storage} -> ${info.fullKey}`,
    status: info.exists ? 'Saved' : 'Not saved',
    size: `${(info.size / 1024).toFixed(2)} KB`,
    autoSave: info.autoSaveEnabled ? 'On' : 'Off',
    saveDelay: `${info.debounce}ms`
  };
}

console.table(generateStorageReport());
```

### **Example 7: Check Before Clear**
```javascript
const cache = ReactiveUtils.reactive({ data: [] });
ReactiveUtils.autoSave(cache, 'cache');

function clearIfLarge() {
  const info = cache.$storageInfo();

  if (info.size > 100000) { // Over 100KB
    console.log(`Cache is ${info.size} bytes, clearing...`);
    cache.$clear();
  }
}
```

### **Example 8: Compare Storage Keys**
```javascript
function compareStorage(state1, state2) {
  const info1 = state1.$storageInfo();
  const info2 = state2.$storageInfo();

  console.log('State 1:', info1.fullKey, `(${info1.size} bytes)`);
  console.log('State 2:', info2.fullKey, `(${info2.size} bytes)`);

  if (info1.fullKey === info2.fullKey) {
    console.warn('WARNING: States share the same storage key!');
  }
}
```

### **Example 9: Settings Panel**
```javascript
const settings = ReactiveUtils.reactive({
  theme: 'light',
  autoSave: true
});

ReactiveUtils.autoSave(settings, 'settings');

function renderSettingsInfo() {
  const info = settings.$storageInfo();

  const html = `
    <div class="storage-info">
      <h3>Storage Information</h3>
      <dl>
        <dt>Location:</dt>
        <dd>${info.storage}</dd>

        <dt>Key:</dt>
        <dd>${info.fullKey}</dd>

        <dt>Status:</dt>
        <dd>${info.exists ? '✓ Saved' : '✗ Not saved'}</dd>

        <dt>Size:</dt>
        <dd>${info.size} bytes</dd>

        <dt>Auto-Save:</dt>
        <dd>${info.autoSaveEnabled ? 'Enabled' : 'Disabled'}</dd>
      </dl>
    </div>
  `;

  document.getElementById('settingsPanel').innerHTML = html;
}
```

### **Example 10: Debugging Helper**
```javascript
const state = ReactiveUtils.reactive({ value: 0 });
ReactiveUtils.autoSave(state, 'debug', { debounce: 2000 });

// Add to window for console access
window.debugStorage = () => {
  const info = state.$storageInfo();

  console.table({
    'Storage Key': info.fullKey,
    'Storage Type': info.storage,
    'Exists': info.exists,
    'Size (bytes)': info.size,
    'Auto-Save': info.autoSaveEnabled,
    'Debounce (ms)': info.debounce
  });

  if (info.exists) {
    console.log('Stored value:', localStorage.getItem(info.fullKey));
  }
};

// Usage in console: debugStorage()
```

---

## **Common Patterns**

### **Pattern 1: Get Basic Info**
```javascript
const info = state.$storageInfo();
console.log(info.key, info.size);
```

### **Pattern 2: Check Existence**
```javascript
if (state.$storageInfo().exists) {
  // Data exists
}
```

### **Pattern 3: Monitor Size**
```javascript
const size = state.$storageInfo().size;
console.log(`${size} bytes`);
```

### **Pattern 4: Check Auto-Save Status**
```javascript
const enabled = state.$storageInfo().autoSaveEnabled;
```

### **Pattern 5: Debug Output**
```javascript
console.table(state.$storageInfo());
```

---

## **Return Value Properties**

### **`key`** (string)
The storage key without namespace
```javascript
const info = state.$storageInfo();
console.log(info.key); // 'myData'
```

### **`fullKey`** (string)
The complete storage key with namespace
```javascript
const info = state.$storageInfo();
console.log(info.fullKey); // 'app:myData'
```

### **`storage`** (string)
Storage type: 'localStorage' or 'sessionStorage'
```javascript
const info = state.$storageInfo();
console.log(info.storage); // 'localStorage'
```

### **`namespace`** (string)
Namespace prefix (empty string if none)
```javascript
const info = state.$storageInfo();
console.log(info.namespace); // 'app' or ''
```

### **`exists`** (boolean)
Whether data exists in storage
```javascript
const info = state.$storageInfo();
if (info.exists) {
  console.log('Data is stored');
}
```

### **`size`** (number)
Approximate size in bytes (0 if not exists)
```javascript
const info = state.$storageInfo();
console.log(`${info.size} bytes`);
console.log(`${(info.size / 1024).toFixed(2)} KB`);
```

### **`autoSaveEnabled`** (boolean)
Whether auto-save is currently enabled
```javascript
const info = state.$storageInfo();
if (!info.autoSaveEnabled) {
  console.log('Auto-save is disabled');
}
```

### **`debounce`** (number)
Debounce delay in milliseconds
```javascript
const info = state.$storageInfo();
console.log(`Save delay: ${info.debounce}ms`);
```

---

## **When to Use**

| Scenario | Use $storageInfo() |
|----------|-------------------|
| Debugging | ✓ Yes |
| Monitoring | ✓ Yes |
| Size checking | ✓ Yes |
| Status display | ✓ Yes |
| Configuration validation | ✓ Yes |
| Admin panels | ✓ Yes |
| Modifying data | ✗ No (use other methods) |

---

## **Best Practices**

1. **Use for debugging**
   ```javascript
   console.table(state.$storageInfo());
   ```

2. **Monitor storage usage**
   ```javascript
   const info = state.$storageInfo();
   if (info.size > threshold) {
     warnUser();
   }
   ```

3. **Display status to users**
   ```javascript
   const info = state.$storageInfo();
   showStatus(`Saved (${info.size} bytes)`);
   ```

4. **Validate configuration**
   ```javascript
   const info = state.$storageInfo();
   assert(info.storage === 'localStorage');
   ```

5. **Create admin panels**
   ```javascript
   const allInfo = states.map(s => s.$storageInfo());
   renderAdminPanel(allInfo);
   ```

6. **Check before operations**
   ```javascript
   if (!state.$storageInfo().exists) {
     initializeDefaults();
   }
   ```

---

## **Key Takeaways**

1. **Returns Object**: Detailed configuration information
2. **Read-Only**: Non-destructive operation
3. **Current State**: Reflects live auto-save status
4. **Size Info**: Approximate storage size in bytes
5. **Existence**: Indicates if data exists
6. **Debug Tool**: Perfect for debugging
7. **Monitoring**: Good for storage monitoring
8. **No Parameters**: Takes no arguments
9. **Full Details**: Complete configuration snapshot
10. **Admin Helper**: Useful for admin interfaces

---

## **Summary**

`$storageInfo()` is a method added to reactive state objects by `autoSave()` that returns detailed information about the storage configuration and current state. When called, it returns an object containing the storage key, full key with namespace, storage type (localStorage/sessionStorage), namespace, whether data exists in storage, approximate size in bytes, current auto-save enabled status, and debounce delay. This is a read-only, non-destructive operation that provides a complete snapshot of the storage configuration and state. Use `$storageInfo()` for debugging storage issues, monitoring storage usage, displaying status information to users, validating configuration, building admin panels, or checking storage state before operations. The returned information is particularly useful during development for understanding how data is stored, tracking storage consumption across multiple states, and verifying that auto-save is configured and operating as expected.
