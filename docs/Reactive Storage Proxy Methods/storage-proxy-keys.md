# `proxy.keys()` - Get All Keys from Reactive Storage

**Quick Start (30 seconds)**
```javascript
// Create reactive storage
const storage = ReactiveUtils.reactiveStorage('localStorage', 'myApp');

// Set multiple values
storage.set('theme', 'dark');
storage.set('fontSize', 16);
storage.set('language', 'en');

// Get all keys (tracks dependency)
const keys = storage.keys();
console.log(keys); // ['theme', 'fontSize', 'language']

// Reactive effect tracks keys()
ReactiveUtils.effect(() => {
  const allKeys = storage.keys();
  console.log(`Storage has ${allKeys.length} keys:`, allKeys);
  // Effect re-runs when keys are added/removed
});

// Add new key - effect runs automatically
storage.set('notifications', true);
// Logs: "Storage has 4 keys: ['theme', 'fontSize', 'language', 'notifications']"
```

---

## **What is `proxy.keys()`?**

`proxy.keys()` is a **method on the reactive storage proxy** returned by `reactiveStorage()` that returns an array of all storage keys in the namespace and **tracks the access as a reactive dependency** so that effects re-run when keys are added or removed.

**Key characteristics:**
- **Returns Array**: Array of key names (strings)
- **Tracks Dependency**: Effects re-run when keys change
- **Namespace Filtered**: Only returns keys in the namespace
- **Reactive**: Creates dependency in tracking contexts
- **No Prefixes**: Returns keys without namespace prefix
- **Safe**: Never throws errors
- **Dynamic**: Reflects current state of storage

---

## **Syntax**

```javascript
proxy.keys()
```

### **Parameters**
None

### **Returns**
- **Type**: `string[]` (array of strings)
- Returns array of all key names in the namespace
- Returns empty array `[]` if no keys exist

---

## **How it works**

```javascript
// When you call proxy.keys() inside an effect
const storage = ReactiveUtils.reactiveStorage('localStorage', 'app');

ReactiveUtils.effect(() => {
  const keys = storage.keys();
  console.log(keys);
});

// Internally:
// 1. Proxy intercepts keys() call
// 2. Tracks reactive dependency (_keys)
const _ = reactiveState._keys;

// 3. Iterates through storage
const keys = [];
const prefix = namespace ? `${namespace}:` : '';

for (let i = 0; i < localStorage.length; i++) {
  const key = localStorage.key(i);
  if (key && (!namespace || key.startsWith(prefix))) {
    // Strip namespace prefix
    const strippedKey = namespace
      ? key.slice(prefix.length)
      : key;
    keys.push(strippedKey);
  }
}

// 4. Returns key array
return keys;
```

**What happens:**
1. Proxy intercepts the keys() call
2. Accesses reactive state _keys (creates dependency)
3. Iterates through browser storage
4. Filters keys by namespace prefix
5. Strips namespace prefix from keys
6. Returns array of key names
7. Effect now "subscribes" to key list changes

---

## **Examples**

### **Example 1: List All Keys**
```javascript
const storage = ReactiveUtils.reactiveStorage('localStorage', 'app');

storage.set('user', { name: 'John' });
storage.set('theme', 'dark');
storage.set('language', 'en');

const keys = storage.keys();
console.log(keys); // ['user', 'theme', 'language']
```

### **Example 2: Reactive Key Count**
```javascript
const storage = ReactiveUtils.reactiveStorage('localStorage', 'app');

// Reactive counter
ReactiveUtils.effect(() => {
  const keys = storage.keys();
  document.getElementById('keyCount').textContent =
    `Storage has ${keys.length} keys`;
});

// Adding/removing keys updates count automatically
storage.set('key1', 'value1'); // Count: 1
storage.set('key2', 'value2'); // Count: 2
storage.remove('key1');        // Count: 1
```

### **Example 3: Settings Panel**
```javascript
const prefs = ReactiveUtils.reactiveStorage('localStorage', 'prefs');

// Reactive settings display
ReactiveUtils.effect(() => {
  const keys = prefs.keys();

  const html = keys.map(key => {
    const value = prefs.get(key);
    return `
      <div class="setting">
        <label>${key}:</label>
        <span>${JSON.stringify(value)}</span>
      </div>
    `;
  }).join('');

  document.getElementById('settingsPanel').innerHTML = html;
});

// Add/remove settings - panel updates automatically
prefs.set('theme', 'dark');
prefs.set('fontSize', 16);
```

### **Example 4: Clear Old Keys**
```javascript
const cache = ReactiveUtils.reactiveStorage('localStorage', 'cache');

function clearOldCache() {
  const keys = cache.keys();
  const oneHourAgo = Date.now() - 3600000;

  keys.forEach(key => {
    const data = cache.get(key);
    if (data && data.timestamp < oneHourAgo) {
      cache.remove(key);
      console.log(`Removed old cache: ${key}`);
    }
  });
}

setInterval(clearOldCache, 60000); // Clean every minute
```

### **Example 5: Export All Data**
```javascript
const storage = ReactiveUtils.reactiveStorage('localStorage', 'app');

function exportData() {
  const keys = storage.keys();
  const data = {};

  keys.forEach(key => {
    data[key] = storage.get(key);
  });

  const json = JSON.stringify(data, null, 2);
  downloadFile('export.json', json);
}

document.getElementById('exportBtn').addEventListener('click', exportData);
```

### **Example 6: Search Keys**
```javascript
const storage = ReactiveUtils.reactiveStorage('localStorage', 'data');

storage.set('user_john', { name: 'John' });
storage.set('user_jane', { name: 'Jane' });
storage.set('config_theme', 'dark');
storage.set('config_lang', 'en');

// Find user keys
const userKeys = storage.keys().filter(key => key.startsWith('user_'));
console.log(userKeys); // ['user_john', 'user_jane']

// Find config keys
const configKeys = storage.keys().filter(key => key.startsWith('config_'));
console.log(configKeys); // ['config_theme', 'config_lang']
```

### **Example 7: Bulk Operations**
```javascript
const storage = ReactiveUtils.reactiveStorage('localStorage', 'temp');

// Clear all temp data
function clearAllTemp() {
  const keys = storage.keys();
  keys.forEach(key => storage.remove(key));
  console.log(`Cleared ${keys.length} temporary items`);
}

// Copy all data to another namespace
function backupData() {
  const backup = ReactiveUtils.reactiveStorage('localStorage', 'backup');
  const keys = storage.keys();

  keys.forEach(key => {
    const value = storage.get(key);
    backup.set(key, value);
  });

  console.log(`Backed up ${keys.length} items`);
}
```

### **Example 8: Storage Browser**
```javascript
const storage = ReactiveUtils.reactiveStorage('localStorage', 'app');

// Reactive storage browser UI
ReactiveUtils.effect(() => {
  const keys = storage.keys();

  const rows = keys.map(key => {
    const value = storage.get(key);
    const size = JSON.stringify(value).length;

    return `
      <tr>
        <td>${key}</td>
        <td>${typeof value}</td>
        <td>${size} bytes</td>
        <td><button onclick="removeKey('${key}')">Delete</button></td>
      </tr>
    `;
  }).join('');

  document.getElementById('storageTable').innerHTML = `
    <table>
      <thead>
        <tr>
          <th>Key</th>
          <th>Type</th>
          <th>Size</th>
          <th>Actions</th>
        </tr>
      </thead>
      <tbody>${rows}</tbody>
    </table>
  `;
});
```

### **Example 9: Namespace Isolation**
```javascript
const userStorage = ReactiveUtils.reactiveStorage('localStorage', 'user');
const appStorage = ReactiveUtils.reactiveStorage('localStorage', 'app');

userStorage.set('name', 'John');
userStorage.set('email', 'john@example.com');

appStorage.set('theme', 'dark');
appStorage.set('language', 'en');

// Each namespace has its own keys
console.log(userStorage.keys()); // ['name', 'email']
console.log(appStorage.keys());  // ['theme', 'language']
```

### **Example 10: Reactive Key List**
```javascript
const tasks = ReactiveUtils.reactiveStorage('localStorage', 'tasks');

// Reactive task list
ReactiveUtils.effect(() => {
  const taskKeys = tasks.keys();

  if (taskKeys.length === 0) {
    document.getElementById('tasks').innerHTML =
      '<p>No tasks yet. Add one!</p>';
    return;
  }

  const html = taskKeys.map(key => {
    const task = tasks.get(key);
    return `
      <div class="task">
        <h3>${task.title}</h3>
        <p>${task.description}</p>
        <button onclick="completeTask('${key}')">Complete</button>
      </div>
    `;
  }).join('');

  document.getElementById('tasks').innerHTML = html;
});

function addTask(title, description) {
  const id = `task_${Date.now()}`;
  tasks.set(id, { title, description, created: Date.now() });
}

function completeTask(id) {
  tasks.remove(id);
}
```

---

## **Common Patterns**

### **Pattern 1: Get All Keys**
```javascript
const keys = storage.keys();
```

### **Pattern 2: Count Keys**
```javascript
const count = storage.keys().length;
```

### **Pattern 3: Iterate Keys**
```javascript
storage.keys().forEach(key => {
  const value = storage.get(key);
  console.log(key, value);
});
```

### **Pattern 4: Filter Keys**
```javascript
const userKeys = storage.keys().filter(k => k.startsWith('user_'));
```

### **Pattern 5: Check Empty**
```javascript
if (storage.keys().length === 0) {
  console.log('Storage is empty');
}
```

---

## **When to Use**

| Scenario | Use proxy.keys() |
|----------|------------------|
| List all stored data | ✓ Yes |
| Count stored items | ✓ Yes |
| Export all data | ✓ Yes |
| Search for keys | ✓ Yes |
| Bulk operations | ✓ Yes |
| Storage browser UI | ✓ Yes |
| Clear all data | ✓ Yes (with forEach) |
| Get single key | ✗ No (use get()) |

---

## **Reactivity Details**

### **Creates Dependency**
```javascript
ReactiveUtils.effect(() => {
  const keys = storage.keys();
  // Effect depends on key list
  // Re-runs when keys added/removed
});
```

### **Responds to set()**
```javascript
ReactiveUtils.effect(() => {
  const count = storage.keys().length;
  console.log(`${count} keys`);
});

storage.set('newKey', 'value');
// Effect runs, logs new count
```

### **Responds to remove()**
```javascript
ReactiveUtils.effect(() => {
  const keys = storage.keys();
  console.log('Keys:', keys);
});

storage.remove('someKey');
// Effect runs, shows updated list
```

---

## **Return Value**

### **With Keys**
```javascript
storage.set('a', 1);
storage.set('b', 2);
console.log(storage.keys()); // ['a', 'b']
```

### **Empty Storage**
```javascript
const empty = ReactiveUtils.reactiveStorage('localStorage', 'empty');
console.log(empty.keys()); // []
```

### **Namespace Filtering**
```javascript
// In localStorage:
// "app:theme" => "dark"
// "app:lang" => "en"
// "other:data" => "value"

const app = ReactiveUtils.reactiveStorage('localStorage', 'app');
console.log(app.keys()); // ['theme', 'lang'] (not 'other:data')
```

---

## **Best Practices**

1. **Use for iteration**
   ```javascript
   storage.keys().forEach(key => {
     const value = storage.get(key);
     process(key, value);
   });
   ```

2. **Filter before processing**
   ```javascript
   const tempKeys = storage.keys().filter(k => k.startsWith('temp_'));
   tempKeys.forEach(key => storage.remove(key));
   ```

3. **Check empty state**
   ```javascript
   if (storage.keys().length === 0) {
     initializeDefaults();
   }
   ```

4. **Export/backup**
   ```javascript
   const backup = {};
   storage.keys().forEach(key => {
     backup[key] = storage.get(key);
   });
   ```

5. **Use reactive counts**
   ```javascript
   ReactiveUtils.effect(() => {
     const count = storage.keys().length;
     updateUI(count);
   });
   ```

6. **Namespace organization**
   ```javascript
   // Good: separate namespaces
   const users = ReactiveUtils.reactiveStorage('localStorage', 'users');
   const prefs = ReactiveUtils.reactiveStorage('localStorage', 'prefs');

   // Each has independent keys
   users.keys();  // Only user keys
   prefs.keys();  // Only pref keys
   ```

---

## **vs. Native Storage**

| Feature | `proxy.keys()` | Manual iteration |
|---------|---------------|------------------|
| Tracks reactivity | ✓ Yes | ✗ No |
| Returns array | ✓ Yes | ✗ Manual |
| Namespace filter | ✓ Automatic | ✗ Manual |
| Safe | ✓ Never throws | ✗ Can throw |
| Convenient | ✓ One call | ✗ Multi-step |

```javascript
// proxy.keys() - reactive, convenient
const storage = ReactiveUtils.reactiveStorage('localStorage', 'app');
const keys = storage.keys();
// Automatic filtering, tracking, array return

// Manual - complex
const keys = [];
for (let i = 0; i < localStorage.length; i++) {
  const key = localStorage.key(i);
  if (key && key.startsWith('app:')) {
    keys.push(key.slice(4));
  }
}
// No reactivity, manual filtering
```

---

## **Key Takeaways**

1. **Returns Array**: Array of all key names
2. **Tracks Dependency**: Creates reactive dependency
3. **Namespace Filtered**: Only keys in namespace
4. **No Prefixes**: Returns clean key names
5. **Reactive**: Effects re-run on key changes
6. **Dynamic**: Reflects current storage state
7. **Safe**: Never throws errors
8. **Empty Safe**: Returns [] if no keys
9. **Iteration Ready**: Perfect for forEach/map
10. **Export Helper**: Great for data export

---

## **Summary**

`proxy.keys()` is a method on the reactive storage proxy returned by `reactiveStorage()` that returns an array of all storage keys in the namespace and tracks the access as a reactive dependency, causing effects and computed properties to automatically re-run when keys are added or removed from storage. When called, it iterates through the browser storage, filters keys by namespace prefix, strips the namespace prefix from each key, and returns an array of clean key names. The key feature is that when used inside an effect or computed property, keys() creates a reactive dependency on the key list, so the effect automatically re-runs whenever keys are added via set() or removed via remove(). This makes it perfect for displaying lists of stored data, counting storage items, implementing storage browser UIs, export/backup functionality, bulk operations on all keys, clearing old cache entries, and any scenario where UI or logic needs to react to changes in what keys exist in storage. Use proxy.keys() when you need to iterate over all stored data, display storage contents, perform bulk operations, or track how many items are stored with automatic reactive updates.
