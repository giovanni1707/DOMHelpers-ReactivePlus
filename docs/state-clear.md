# `clear(state)` - Clear State from Storage

**Quick Start (30 seconds)**
```javascript
const state = ReactiveUtils.reactive({
  session: 'abc123',
  user: { name: 'John' }
});

ReactiveUtils.autoSave(state, 'sessionData');

// Clear from storage
ReactiveUtils.clear(state);
console.log('Storage cleared');

// State object remains unchanged
console.log(state.session); // Still 'abc123'
console.log(state.user);    // Still { name: 'John' }
```

---

## **What is `clear(state)`?**

`clear(state)` is a **method added to reactive state objects** by `autoSave()` that removes the state data from storage without modifying the current state object.

**Key characteristics:**
- **Removes from Storage**: Deletes stored data
- **Keeps State Intact**: Doesn't modify current state object
- **No Return Value**: Returns void
- **Permanent**: Cannot be undone
- **Stops Persistence**: Prevents future auto-loads
- **Manual Operation**: Explicit clear action

---

## **Syntax**

```javascript
ReactiveUtils.clear(state)
```

### **Parameters**
- **`state`** (Object): Storage-enabled reactive state object

### **Returns**
- **Type**: `void`

---

## **How it works**

```javascript
// When autoSave() is called
ReactiveUtils.autoSave(state, 'key', options);

// This method is added to state:
state.$clear = function() {
  // Remove from storage
  store.remove(key);

  // State object remains unchanged
  // Only storage is affected
};
```

**What happens:**
1. Removes the key from storage (localStorage or sessionStorage)
2. State object values remain unchanged
3. Future load(state) calls will return false
4. Auto-load on page refresh will not load this data

---

## **Examples**

### **Example 1: Clear Session Data**
```javascript
const session = ReactiveUtils.reactive({
  token: 'abc123',
  userId: '456'
});

ReactiveUtils.autoSave(session, 'session');

function logout() {
  ReactiveUtils.clear(session);
  session.token = null;
  session.userId = null;
  redirectToLogin();
}
```

### **Example 2: Clear Cache**
```javascript
const cache = ReactiveUtils.reactive({
  data: [],
  timestamp: Date.now()
});

ReactiveUtils.autoSave(cache, 'cache');

function clearCache() {
  cache.clear(state);
  cache.data = [];
  cache.timestamp = Date.now();
  console.log('Cache cleared');
}
```

### **Example 3: Reset to Defaults**
```javascript
const settings = ReactiveUtils.reactive({
  theme: 'dark',
  fontSize: 16,
  notifications: true
});

ReactiveUtils.autoSave(settings, 'settings');

function resetSettings() {
  // Clear stored settings
  settings.clear(state);

  // Reset to defaults
  settings.theme = 'light';
  settings.fontSize = 14;
  settings.notifications = false;

  // Save new defaults
  settings.save(state);
}
```

### **Example 4: Clear on Error**
```javascript
const data = ReactiveUtils.reactive({
  value: null
});

ReactiveUtils.autoSave(data, 'data', {
  onLoad: (value) => {
    if (!isValid(value)) {
      // Clear corrupted data
      data.clear(state);
      return getDefaults();
    }
    return value;
  }
});
```

### **Example 5: Clear Temporary Data**
```javascript
const temp = ReactiveUtils.reactive({
  formData: {}
});

ReactiveUtils.autoSave(temp, 'tempData');

document.getElementById('submitBtn').addEventListener('click', () => {
  submitForm(temp.formData);

  // Clear after successful submit
  ReactiveUtils.clear(temp);
  temp.formData = {};
});
```

### **Example 6: Clear Draft**
```javascript
const draft = ReactiveUtils.reactive({
  title: '',
  content: ''
});

ReactiveUtils.autoSave(draft, 'draft');

function publishAndClear() {
  publishPost(draft);

  // Clear draft after publishing
  draft.clear(state);
  draft.title = '';
  draft.content = '';

  console.log('Draft cleared');
}
```

### **Example 7: Clear Multiple Storages**
```javascript
const user = ReactiveUtils.reactive({ id: null });
const prefs = ReactiveUtils.reactive({ theme: 'light' });
const cache = ReactiveUtils.reactive({ data: [] });

ReactiveUtils.autoSave(user, 'user');
ReactiveUtils.autoSave(prefs, 'prefs');
ReactiveUtils.autoSave(cache, 'cache');

function clearAll() {
  user.clear(state);
  prefs.clear(state);
  cache.clear(state);
  console.log('All storage cleared');
}
```

### **Example 8: Conditional Clear**
```javascript
const state = ReactiveUtils.reactive({
  data: [],
  shouldPersist: true
});

ReactiveUtils.autoSave(state, 'state');

window.addEventListener('beforeunload', () => {
  if (!state.shouldPersist) {
    ReactiveUtils.clear(state);
  }
});
```

### **Example 9: Clear on Privacy Mode**
```javascript
const browsing = ReactiveUtils.reactive({
  history: [],
  bookmarks: []
});

ReactiveUtils.autoSave(browsing, 'browsing');

function enablePrivacyMode() {
  browsing.clear(state);
  browsing.history = [];
  browsing.bookmarks = [];
  console.log('Privacy mode enabled');
}
```

### **Example 10: Clear Expired Data**
```javascript
const cache = ReactiveUtils.reactive({
  value: null,
  expiresAt: null
});

ReactiveUtils.autoSave(cache, 'cache');

function checkExpiration() {
  if (cache.expiresAt && Date.now() > cache.expiresAt) {
    console.log('Data expired, clearing');
    cache.clear(state);
    cache.value = null;
    cache.expiresAt = null;
  }
}
```

---

## **Common Patterns**

### **Pattern 1: Clear and Reset**
```javascript
ReactiveUtils.clear(state);
state.value = defaultValue;
```

### **Pattern 2: Clear on Logout**
```javascript
function logout() {
  ReactiveUtils.clear(session);
  redirectToLogin();
}
```

### **Pattern 3: Clear and Save New**
```javascript
ReactiveUtils.clear(state);
state.data = newData;
ReactiveUtils.save(state);
```

### **Pattern 4: Clear Temporary**
```javascript
if (isTemporary) {
  ReactiveUtils.clear(state);
}
```

### **Pattern 5: Clear on Complete**
```javascript
onComplete(() => {
  ReactiveUtils.clear(state);
});
```

---

## **When to Use**

| Scenario | Use clear() |
|----------|--------------|
| Logout | ✓ Yes |
| Clear cache | ✓ Yes |
| Reset to defaults | ✓ Yes |
| Privacy mode | ✓ Yes |
| Complete action | ✓ Yes |
| Clear temporary data | ✓ Yes |
| Modify state values | ✗ No (use state.prop = value) |

---

## **Important Notes**

### **Storage vs State**
```javascript
// clear(state) only affects storage
ReactiveUtils.clear(state);
console.log(state.value); // Still has value

// To clear both:
ReactiveUtils.clear(state);
state.value = null;
```

### **Cannot Undo**
```javascript
ReactiveUtils.clear(state);
// Data is permanently removed from storage
// Cannot be recovered with load(state)
```

### **Future Loads**
```javascript
ReactiveUtils.clear(state);

// Later...
const loaded = ReactiveUtils.load(state); // Returns false
```

---

## **vs. State Reset**

| Operation | Storage | State Object |
|-----------|---------|--------------|
| `clear(state)` | Cleared | Unchanged |
| `state.prop = null` | Auto-saved (if enabled) | Changed |
| `clear(state) + reset` | Cleared then saved | Changed then saved |

---

## **Best Practices**

1. **Clear and reset together**
   ```javascript
   ReactiveUtils.clear(state);
   state.value = defaultValue;
   ```

2. **Clear sensitive data on logout**
   ```javascript
   function logout() {
     ReactiveUtils.clear(session);
     ReactiveUtils.clear(authToken);
   }
   ```

3. **Confirm before clearing**
   ```javascript
   if (confirm('Clear all data?')) {
     ReactiveUtils.clear(state);
   }
   ```

4. **Clear temporary data after use**
   ```javascript
   processData(temp.value);
   ReactiveUtils.clear(temp);
   ```

5. **Clear on privacy actions**
   ```javascript
   function clearHistory() {
     history.clear(state);
     history.items = [];
   }
   ```

6. **Document side effects**
   ```javascript
   // Clears saved preferences
   // User will see defaults on next visit
   prefs.clear(state);
   ```

---

## **Key Takeaways**

1. **Removes Storage**: Deletes data from storage
2. **Keeps State**: State object remains unchanged
3. **No Return**: Returns void
4. **Permanent**: Cannot be undone
5. **Affects Load**: Future load(state) will return false
6. **Manual**: Requires explicit call
7. **Privacy**: Good for sensitive data
8. **Cleanup**: Use for temporary data
9. **Reset Pattern**: Often combined with state reset
10. **No Auto-Load**: Prevents auto-load on refresh

---

## **Summary**

`clear(state)` is a method added to reactive state objects by `autoSave()` that removes the state data from storage without modifying the current state object. When called, it permanently deletes the stored data from localStorage or sessionStorage, but leaves the in-memory state object unchanged. This means the current state values remain available, but future page loads will not auto-load this data and load(state) calls will return false. Use `clear(state)` when you need to remove persisted data from storage, such as during logout to clear sensitive session data, when implementing privacy modes, when resetting to default settings, after completing temporary operations, or when clearing cache. It's perfect for cleanup operations where you want to prevent data from persisting across page reloads. Typically used in combination with resetting state values to defaults, though the operations are independent - clearing storage doesn't automatically reset the state object.
