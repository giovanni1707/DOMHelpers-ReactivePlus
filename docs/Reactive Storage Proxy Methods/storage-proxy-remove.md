# `proxy.remove(key)` - Remove Value from Reactive Storage

**Quick Start (30 seconds)**
```javascript
// Create reactive storage
const storage = ReactiveUtils.reactiveStorage('localStorage', 'myApp');

// Set some values
storage.set('temp1', 'value1');
storage.set('temp2', 'value2');
storage.set('temp3', 'value3');

// Remove a value (triggers reactivity)
storage.remove('temp1');

// Reactive effect responds to removal
ReactiveUtils.effect(() => {
  if (storage.has('temp1')) {
    console.log('temp1 exists');
  } else {
    console.log('temp1 was removed');
  }
});
// Logs: "temp1 was removed"
```

---

## **What is `proxy.remove(key)`?**

`proxy.remove()` is a **method on the reactive storage proxy** returned by `reactiveStorage()` that removes a key-value pair from browser storage and triggers reactive updates in any effects or computed properties tracking that storage.

**Key characteristics:**
- **Removes Key**: Deletes key from localStorage or sessionStorage
- **Triggers Reactivity**: Notifies all tracking effects
- **Returns Boolean**: true on success, false on failure
- **Permanent**: Cannot be undone
- **Cross-Tab**: Triggers storage events
- **Safe**: Never throws errors
- **Namespaced**: Handles namespace automatically

---

## **Syntax**

```javascript
proxy.remove(key)
```

### **Parameters**
- **`key`** (string): Storage key to remove (without namespace prefix)

### **Returns**
- **Type**: `boolean`
- **`true`**: Successfully removed
- **`false`**: Removal failed (key not found, etc.)

---

## **How it works**

```javascript
// When you call proxy.remove()
const storage = ReactiveUtils.reactiveStorage('localStorage', 'app');
storage.remove('temp');

// Internally:
// 1. Constructs full key
const fullKey = namespace ? `${namespace}:${key}` : key;

// 2. Removes from storage
localStorage.removeItem(fullKey);

// 3. Increments reactive version
reactiveState._version++;
reactiveState._keys = new Set(store.keys());

// 4. All tracking effects re-run

// 5. Fires storage event (cross-tab)
```

**What happens:**
1. Constructs namespaced storage key
2. Removes item from browser storage
3. Increments internal reactive version number
4. Updates internal keys Set
5. Triggers all effects tracking this storage
6. Fires storage event for cross-tab sync
7. Returns success status

---

## **Examples**

### **Example 1: Basic Remove**
```javascript
const storage = ReactiveUtils.reactiveStorage('localStorage');

// Set values
storage.set('temp', 'temporary data');
storage.set('cache', 'cached data');

// Remove values
storage.remove('temp');
storage.remove('cache');

console.log(storage.get('temp'));  // null
console.log(storage.get('cache')); // null
```

### **Example 2: Reactive Removal**
```javascript
const storage = ReactiveUtils.reactiveStorage('localStorage', 'app');

storage.set('notification', 'You have a new message');

// Effect tracks storage
ReactiveUtils.effect(() => {
  const notification = storage.get('notification');
  if (notification) {
    querySelector('#notification').textContent = notification;
    querySelector('#notification').style.display = 'block';
  } else {
    querySelector('#notification').style.display = 'none';
  }
});

// Remove notification after 5 seconds
setTimeout(() => {
  storage.remove('notification');
  // Effect runs, notification hidden
}, 5000);
```

### **Example 3: Clear Temporary Data**
```javascript
const cache = ReactiveUtils.reactiveStorage('localStorage', 'cache');

// Set cache with expiration
cache.set('apiData', data, { expires: 3600 });
cache.set('tempResult', result, { expires: 300 });

// Manually clear cache
function clearCache() {
  cache.remove('apiData');
  cache.remove('tempResult');
  console.log('Cache cleared');
}

// Clear button
querySelector('#clearCacheBtn').addEventListener('click', clearCache);
```

### **Example 4: Remove After Use**
```javascript
const temp = ReactiveUtils.reactiveStorage('sessionStorage', 'temp');

// Store temporary form data
temp.set('formSubmission', formData);

// Submit form
submitForm(formData).then(() => {
  // Remove after successful submission
  temp.remove('formSubmission');
  console.log('Temporary data removed');
});
```

### **Example 5: Logout Flow**
```javascript
const auth = ReactiveUtils.reactiveStorage('localStorage', 'auth');

// Login
function login(credentials) {
  const token = authenticateUser(credentials);
  auth.set('token', token);
  auth.set('userId', credentials.userId);
  auth.set('loginTime', Date.now());
}

// Logout - remove all auth data
function logout() {
  auth.remove('token');
  auth.remove('userId');
  auth.remove('loginTime');
  redirectToLogin();
}

// Reactive auth state
ReactiveUtils.effect(() => {
  const token = auth.get('token');
  if (token) {
    console.log('User authenticated');
  } else {
    console.log('User logged out');
  }
});
```

### **Example 6: Shopping Cart**
```javascript
const cart = ReactiveUtils.reactiveStorage('localStorage', 'cart');

// Initialize cart
cart.set('items', []);

// Reactive cart display
ReactiveUtils.effect(() => {
  const items = cart.get('items') || [];
  querySelector('#cartCount').textContent = items.length;

  const html = items.map(item => `
    <div class="cart-item">
      <span>${item.name}</span>
      <button onclick="removeFromCart('${item.id}')">Remove</button>
    </div>
  `).join('');
  querySelector('#cartItems').innerHTML = html;
});

// Remove specific item
function removeFromCart(itemId) {
  const items = cart.get('items') || [];
  const updated = items.filter(item => item.id !== itemId);
  cart.set('items', updated);
}

// Clear entire cart
function clearCart() {
  cart.remove('items');
  // Or reset to empty array
  // cart.set('items', []);
}
```

### **Example 7: Dismiss Notifications**
```javascript
const notifications = ReactiveUtils.reactiveStorage('localStorage', 'notifications');

// Set notification
notifications.set('update', {
  message: 'New version available',
  timestamp: Date.now()
});

// Reactive notification display
ReactiveUtils.effect(() => {
  const update = notifications.get('update');
  if (update) {
    showNotificationBanner(update.message);
  } else {
    hideNotificationBanner();
  }
});

// Dismiss notification
function dismissNotification() {
  notifications.remove('update');
  // Effect runs, banner hidden
}
```

### **Example 8: Remove Old Cache Entries**
```javascript
const cache = ReactiveUtils.reactiveStorage('localStorage', 'cache');

// Cache multiple items
cache.set('data1', { timestamp: Date.now() - 7200000 }); // 2 hours old
cache.set('data2', { timestamp: Date.now() - 3600000 });  // 1 hour old
cache.set('data3', { timestamp: Date.now() - 300000 });   // 5 minutes old

// Remove entries older than 1 hour
function cleanOldCache() {
  const oneHourAgo = Date.now() - 3600000;
  const keys = cache.keys();

  keys.forEach(key => {
    const item = cache.get(key);
    if (item && item.timestamp < oneHourAgo) {
      cache.remove(key);
      console.log(`Removed old cache: ${key}`);
    }
  });
}
```

### **Example 9: Cross-Tab Removal**
```javascript
const shared = ReactiveUtils.reactiveStorage('localStorage', 'shared');

// Tab 1: Listen for task removal
ReactiveUtils.effect(() => {
  const tasks = shared.get('tasks') || [];
  console.log('Current tasks:', tasks);
  displayTasks(tasks);
});

// Tab 2: Remove completed task
function completeTask(taskId) {
  const tasks = shared.get('tasks') || [];
  const updated = tasks.filter(task => task.id !== taskId);
  shared.set('tasks', updated);
  // Tab 1 effect runs, task list updates
}
```

### **Example 10: Privacy Mode**
```javascript
const history = ReactiveUtils.reactiveStorage('localStorage', 'history');

// Track visited pages
history.set('visited', [
  { url: '/page1', timestamp: Date.now() },
  { url: '/page2', timestamp: Date.now() }
]);

// Enable privacy mode - remove history
function enablePrivacyMode() {
  history.remove('visited');
  history.remove('searches');
  history.remove('recentItems');
  console.log('Privacy mode enabled - history cleared');
}

// Reactive privacy indicator
ReactiveUtils.effect(() => {
  const hasHistory = history.has('visited');
  querySelector('#privacyMode').textContent =
    hasHistory ? 'Normal Mode' : 'Privacy Mode';
});
```

---

## **Common Patterns**

### **Pattern 1: Simple Remove**
```javascript
storage.remove('key');
```

### **Pattern 2: Remove and Check**
```javascript
const removed = storage.remove('key');
if (removed) {
  console.log('Successfully removed');
}
```

### **Pattern 3: Remove Multiple Keys**
```javascript
['key1', 'key2', 'key3'].forEach(key => {
  storage.remove(key);
});
```

### **Pattern 4: Conditional Remove**
```javascript
if (shouldRemove) {
  storage.remove('key');
}
```

### **Pattern 5: Remove After Action**
```javascript
performAction().then(() => {
  storage.remove('tempData');
});
```

---

## **When to Use**

| Scenario | Use proxy.remove() |
|----------|-------------------|
| Clear temporary data | ✓ Yes |
| Logout flow | ✓ Yes |
| Dismiss notifications | ✓ Yes |
| Clear cache | ✓ Yes |
| Privacy actions | ✓ Yes |
| Remove old entries | ✓ Yes |
| Reset specific key | ✓ Yes |
| Clear all keys | ✗ No (use clear()) |

---

## **vs. set(key, null)**

Both remove a value, but with different semantics:

```javascript
// Remove - deletes the key
storage.remove('key');
console.log(storage.has('key')); // false
console.log(storage.get('key')); // null

// Set to null - key exists with null value
storage.set('key', null);
console.log(storage.has('key')); // true
console.log(storage.get('key')); // null
```

**Best Practice**: Use `remove()` to completely delete keys.

---

## **Return Value**

### **Successful Removal**
```javascript
storage.set('temp', 'value');
const result = storage.remove('temp');
console.log(result); // true
```

### **Key Not Found**
```javascript
const result = storage.remove('nonexistent');
console.log(result); // true (still succeeds)
```

### **Error Handling**
```javascript
try {
  storage.remove('key');
} catch (e) {
  // Never throws - always safe to call
}
```

---

## **Best Practices**

1. **Remove after use**
   ```javascript
   const data = storage.get('temp');
   processData(data);
   storage.remove('temp');
   ```

2. **Clear sensitive data on logout**
   ```javascript
   function logout() {
     storage.remove('authToken');
     storage.remove('userId');
     storage.remove('sessionId');
   }
   ```

3. **Remove expired cache**
   ```javascript
   function cleanCache() {
     const keys = storage.keys();
     keys.forEach(key => {
       const data = storage.get(key);
       if (isExpired(data)) {
         storage.remove(key);
       }
     });
   }
   ```

4. **Provide feedback**
   ```javascript
   storage.remove('notification');
   showToast('Notification dismissed');
   ```

5. **Check before removing**
   ```javascript
   if (storage.has('temp')) {
     storage.remove('temp');
   }
   ```

6. **Batch removals**
   ```javascript
   const tempKeys = storage.keys().filter(k => k.startsWith('temp_'));
   tempKeys.forEach(key => storage.remove(key));
   ```

---

## **vs. Native localStorage.removeItem()**

| Feature | `proxy.remove()` | `localStorage.removeItem()` |
|---------|-----------------|----------------------------|
| Triggers reactivity | ✓ Yes | ✗ No |
| Returns boolean | ✓ Yes | ✗ No (void) |
| Namespace | ✓ Automatic | ✗ Manual |
| Cross-tab sync | ✓ Yes | ⚠ Via events |
| Safe | ✓ Never throws | ✓ Never throws |

```javascript
// proxy.remove() - reactive
const storage = ReactiveUtils.reactiveStorage('localStorage');
storage.remove('key');
// Triggers effects, handles namespace

// localStorage.removeItem() - manual
localStorage.removeItem('key');
// No reactivity, manual namespace
```

---

## **Key Takeaways**

1. **Removes Key**: Deletes from browser storage
2. **Triggers Reactivity**: Notifies tracking effects
3. **Returns Boolean**: Indicates success
4. **Permanent**: Cannot be undone
5. **Safe**: Never throws errors
6. **Cross-Tab**: Fires storage events
7. **Namespaced**: Handles namespace automatically
8. **Updates Keys**: Updates internal keys Set
9. **Reactive**: Effects re-run on removal
10. **Complete**: Removes key entirely (not sets to null)

---

## **Summary**

`proxy.remove(key)` is a method on the reactive storage proxy returned by `reactiveStorage()` that removes a key-value pair from browser storage and triggers reactive updates in any effects or computed properties tracking that storage. When called, it constructs the namespaced storage key, removes the item from localStorage or sessionStorage, increments an internal reactive version number, and triggers all effects that depend on this storage to re-run. The method returns true if the removal succeeded (or if the key didn't exist), and false only in exceptional error cases. Use `proxy.remove()` when you need to delete stored data and want reactive components to automatically respond to the removal, such as clearing temporary data after use, implementing logout flows that clear authentication data, dismissing notifications, removing old cache entries, privacy actions that clear history, or any scenario where multiple parts of your application need to react when data is deleted. Unlike native localStorage.removeItem(), proxy.remove() provides automatic namespace handling and most importantly, reactive updates that keep your UI synchronized when data is removed from storage.
