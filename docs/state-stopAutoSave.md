# `$stopAutoSave()` - Stop Automatic Saving

**Quick Start (30 seconds)**
```javascript
const state = ReactiveUtils.reactive({
  count: 0,
  data: []
});

ReactiveUtils.autoSave(state, 'myState', {
  debounce: 500
});

// Changes auto-save normally
state.count = 1; // Saves after 500ms

// Stop auto-saving
state.$stopAutoSave();

// Changes no longer auto-save
state.count = 2; // Not saved
state.data = [1, 2, 3]; // Not saved

// Can still save manually
state.$save();
```

---

## **What is `$stopAutoSave()`?**

`$stopAutoSave()` is a **method added to reactive state objects** by `autoSave()` that temporarily disables automatic saving while keeping all other functionality intact.

**Key characteristics:**
- **Stops Auto-Save**: Disables automatic saving on changes
- **Keeps State Reactive**: State remains reactive
- **Manual Save Works**: $save() still functions
- **Reversible**: Can restart with $startAutoSave()
- **No Parameters**: Takes no arguments
- **Returns Void**: No return value

---

## **Syntax**

```javascript
state.$stopAutoSave()
```

### **Parameters**
- None

### **Returns**
- **Type**: `void`

---

## **How it works**

```javascript
// When autoSave() is called
ReactiveUtils.autoSave(state, 'key', options);

// Internal state
let autoSaveEnabled = options.autoSave !== false;
let effectCleanup;

// This method is added to state:
state.$stopAutoSave = function() {
  autoSaveEnabled = false;

  // Stop the reactive effect that watches for changes
  if (effectCleanup) {
    effectCleanup();
    effectCleanup = null;
  }
};
```

**What happens:**
1. Sets internal flag to disable auto-save
2. Removes the reactive effect watching for changes
3. State remains reactive but changes don't trigger saves
4. Manual $save() still works

---

## **Examples**

### **Example 1: Batch Updates**
```javascript
const data = ReactiveUtils.reactive({
  items: [],
  count: 0,
  total: 0
});

ReactiveUtils.autoSave(data, 'data', { debounce: 500 });

// Stop auto-save for batch updates
data.$stopAutoSave();

data.items = [1, 2, 3, 4, 5];
data.count = 5;
data.total = 15;

// Save once after all updates
data.$save();

// Resume auto-save
data.$startAutoSave();
```

### **Example 2: Editing Mode**
```javascript
const document = ReactiveUtils.reactive({
  title: '',
  content: '',
  isEditing: false
});

ReactiveUtils.autoSave(document, 'doc');

function startEditing() {
  document.isEditing = true;
  document.$stopAutoSave(); // Don't save while editing
}

function saveAndExit() {
  document.$save(); // Save manually
  document.$startAutoSave(); // Resume auto-save
  document.isEditing = false;
}
```

### **Example 3: Form Validation**
```javascript
const form = ReactiveUtils.reactive({
  email: '',
  password: '',
  isValid: false
});

ReactiveUtils.autoSave(form, 'form');

// Stop auto-save during form fill
form.$stopAutoSave();

form.email = 'user@example.com';
form.password = 'password123';

// Validate
form.isValid = validateForm(form);

if (form.isValid) {
  form.$save();
  form.$startAutoSave();
}
```

### **Example 4: Offline Mode**
```javascript
const state = ReactiveUtils.reactive({
  data: [],
  isOnline: navigator.onLine
});

ReactiveUtils.autoSave(state, 'state');

window.addEventListener('offline', () => {
  state.isOnline = false;
  state.$stopAutoSave();
  console.log('Auto-save disabled (offline)');
});

window.addEventListener('online', () => {
  state.isOnline = true;
  state.$save(); // Sync when back online
  state.$startAutoSave();
  console.log('Auto-save enabled (online)');
});
```

### **Example 5: Temporary Suspension**
```javascript
const state = ReactiveUtils.reactive({
  value: 0
});

ReactiveUtils.autoSave(state, 'state');

function performBulkOperation() {
  state.$stopAutoSave();

  for (let i = 0; i < 1000; i++) {
    state.value = i; // Would save 1000 times without stop
  }

  state.$save(); // Save once
  state.$startAutoSave();
}
```

### **Example 6: Privacy Mode**
```javascript
const browsing = ReactiveUtils.reactive({
  history: [],
  privateMode: false
});

ReactiveUtils.autoSave(browsing, 'history');

function togglePrivateMode(enabled) {
  browsing.privateMode = enabled;

  if (enabled) {
    browsing.$stopAutoSave();
    console.log('Private mode: Auto-save disabled');
  } else {
    browsing.$startAutoSave();
    console.log('Private mode: Auto-save enabled');
  }
}
```

### **Example 7: Performance Optimization**
```javascript
const canvas = ReactiveUtils.reactive({
  pixels: [],
  width: 800,
  height: 600
});

ReactiveUtils.autoSave(canvas, 'canvas');

// Stop during intensive drawing
function startDrawing() {
  canvas.$stopAutoSave();
}

function finishDrawing() {
  canvas.$save();
  canvas.$startAutoSave();
}
```

### **Example 8: Conditional Auto-Save**
```javascript
const editor = ReactiveUtils.reactive({
  content: '',
  autoSaveEnabled: true
});

ReactiveUtils.autoSave(editor, 'editor');

function setAutoSave(enabled) {
  editor.autoSaveEnabled = enabled;

  if (enabled) {
    editor.$startAutoSave();
  } else {
    editor.$stopAutoSave();
  }
}

// Usage
setAutoSave(false); // User disabled auto-save
setAutoSave(true);  // User enabled auto-save
```

### **Example 9: Import Data**
```javascript
const state = ReactiveUtils.reactive({
  records: []
});

ReactiveUtils.autoSave(state, 'records');

async function importRecords(file) {
  state.$stopAutoSave();

  const data = await parseFile(file);
  state.records = data; // Large data import

  state.$save();
  state.$startAutoSave();

  console.log(`Imported ${data.length} records`);
}
```

### **Example 10: Testing Mode**
```javascript
const app = ReactiveUtils.reactive({
  config: {}
});

ReactiveUtils.autoSave(app, 'config');

if (process.env.NODE_ENV === 'test') {
  // Don't save during tests
  app.$stopAutoSave();
}
```

---

## **Common Patterns**

### **Pattern 1: Stop-Update-Resume**
```javascript
state.$stopAutoSave();
// Make changes
state.$startAutoSave();
```

### **Pattern 2: Stop-Update-Save-Resume**
```javascript
state.$stopAutoSave();
// Make changes
state.$save();
state.$startAutoSave();
```

### **Pattern 3: Conditional Stop**
```javascript
if (shouldStopAutoSave) {
  state.$stopAutoSave();
}
```

### **Pattern 4: Batch Updates**
```javascript
state.$stopAutoSave();
batchUpdate(state);
state.$save();
state.$startAutoSave();
```

### **Pattern 5: Toggle Auto-Save**
```javascript
function toggleAutoSave(enabled) {
  if (enabled) {
    state.$startAutoSave();
  } else {
    state.$stopAutoSave();
  }
}
```

---

## **When to Use**

| Scenario | Use $stopAutoSave() |
|----------|---------------------|
| Batch updates | ✓ Yes |
| Large imports | ✓ Yes |
| Performance optimization | ✓ Yes |
| Offline mode | ✓ Yes |
| Privacy mode | ✓ Yes |
| Testing | ✓ Yes |
| User preference | ✓ Yes |
| Single change | ✗ No |
| Normal operations | ✗ No |

---

## **Important Notes**

### **State Remains Reactive**
```javascript
state.$stopAutoSave();
// State still reactive, just doesn't save
ReactiveUtils.effect(() => {
  console.log(state.value); // Still runs on changes
});
```

### **Manual Save Still Works**
```javascript
state.$stopAutoSave();
state.value = 'new';
state.$save(); // This still works
```

### **Can Be Resumed**
```javascript
state.$stopAutoSave();
// Later
state.$startAutoSave(); // Resumes auto-save
```

---

## **vs. autoSave: false**

| Feature | `$stopAutoSave()` | `autoSave: false` option |
|---------|------------------|-------------------------|
| When set | Runtime | Initialization |
| Reversible | ✓ Yes ($startAutoSave) | ✗ No |
| Dynamic | ✓ Yes | ✗ No |

```javascript
// Option: Set at initialization, permanent
ReactiveUtils.autoSave(state, 'key', { autoSave: false });

// Method: Can toggle at runtime
ReactiveUtils.autoSave(state, 'key');
state.$stopAutoSave(); // Disable
state.$startAutoSave(); // Enable
```

---

## **Best Practices**

1. **Always resume after batch updates**
   ```javascript
   state.$stopAutoSave();
   batchUpdate();
   state.$save();
   state.$startAutoSave(); // Don't forget!
   ```

2. **Use for performance**
   ```javascript
   // Stop during intensive operations
   state.$stopAutoSave();
   performIntensiveOperation();
   state.$save();
   state.$startAutoSave();
   ```

3. **Combine with manual save**
   ```javascript
   state.$stopAutoSave();
   makeChanges();
   state.$save(); // Save when done
   state.$startAutoSave();
   ```

4. **Use for user preferences**
   ```javascript
   if (userPrefs.disableAutoSave) {
     state.$stopAutoSave();
   }
   ```

5. **Handle offline scenarios**
   ```javascript
   if (!navigator.onLine) {
     state.$stopAutoSave();
   }
   ```

6. **Clean up properly**
   ```javascript
   try {
     state.$stopAutoSave();
     doWork();
   } finally {
     state.$startAutoSave();
   }
   ```

---

## **Key Takeaways**

1. **Stops Auto-Save**: Disables automatic saving
2. **Reversible**: Can restart with $startAutoSave()
3. **State Reactive**: State remains reactive
4. **Manual Save**: $save() still works
5. **Performance**: Good for batch operations
6. **User Control**: Allows user to disable auto-save
7. **Offline**: Useful for offline mode
8. **Testing**: Good for test environments
9. **No Parameters**: Takes no arguments
10. **Returns Void**: No return value

---

## **Summary**

`$stopAutoSave()` is a method added to reactive state objects by `autoSave()` that temporarily disables automatic saving while keeping all other functionality intact. When called, it removes the reactive effect that watches for state changes and triggers saves, but the state itself remains reactive and all other methods ($save, $load, etc.) continue to work normally. Use `$stopAutoSave()` when making batch updates to avoid multiple save operations, during large data imports that would trigger many saves, for performance optimization in intensive operations, when implementing offline mode, for privacy modes where saving should be disabled, in test environments, or when giving users control over auto-save behavior. Always pair `$stopAutoSave()` with `$startAutoSave()` to resume automatic saving when done, and consider using manual `$save()` calls before resuming to ensure changes are persisted. The method is reversible and can be toggled at runtime, unlike the autoSave option which is set at initialization.
