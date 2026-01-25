# `save(state)` - Force Save State to Storage

**Quick Start (30 seconds)**
```javascript
const state = ReactiveUtils.reactive({
  count: 0,
  name: 'App'
});

ReactiveUtils.autoSave(state, 'myState', {
  autoSave: false, // Disable auto-save
  debounce: 1000
});

// Make changes
state.count = 42;
state.name = 'My App';

// Force immediate save
ReactiveUtils.save(state);
console.log('Saved immediately!');
```

---

## **What is `save(state)`?**

`save()` is a **namespace method** that forces an immediate save of reactive state to storage, bypassing debounce delays and auto-save settings.

**Key characteristics:**
- **Immediate**: Saves immediately, no debounce delay
- **Manual Control**: Explicit save operation
- **Bypasses Auto-Save**: Works even if autoSave is disabled
- **Respects Hooks**: Still runs onSave callback if configured
- **Returns Boolean**: Returns true on success
- **Synchronous**: Completes before continuing

---

## **Syntax**

```javascript
ReactiveUtils.save(state)
```

### **Parameters**
- **`state`** (Object): Storage-enabled reactive state object

### **Returns**
- **Type**: `boolean`
- **`true`**: Successfully saved
- **`false`**: Save failed or state doesn't have storage enabled

---

## **How it works**

```javascript
// Namespace method implementation
ReactiveUtils.save = function(state) {
  if (!state || !state.$save) {
    console.error('Invalid state or storage not enabled');
    return false;
  }
  return ReactiveUtils.save(state); // Calls internal implementation
};

// Internal implementation
// Gets raw copy of state
// Applies onSave transformation if configured
// Saves to storage immediately
// Bypasses any debounce delay
```

**What happens:**
1. Validates state has storage enabled
2. Gets raw (non-reactive) copy of state
3. Applies onSave transformation if configured
4. Saves to storage immediately
5. Bypasses any debounce delay
6. Returns success status

---

## **Examples**

### **Example 1: Manual Save**
```javascript
const settings = ReactiveUtils.reactive({
  theme: 'light',
  language: 'en'
});

ReactiveUtils.autoSave(settings, 'settings', {
  autoSave: false // Manual save only
});

settings.theme = 'dark';
settings.language = 'es';

// Save when ready
ReactiveUtils.save(settings);
```

### **Example 2: Save on Button Click**
```javascript
const formData = ReactiveUtils.reactive({
  title: '',
  content: ''
});

ReactiveUtils.autoSave(formData, 'draft', {
  autoSave: false
});

document.getElementById('saveBtn').addEventListener('click', () => {
  ReactiveUtils.save(formData);
  showNotification('Draft saved!');
});
```

### **Example 3: Save Before Navigation**
```javascript
const editor = ReactiveUtils.reactive({
  content: '',
  lastSaved: null
});

ReactiveUtils.autoSave(editor, 'document', {
  debounce: 2000
});

window.addEventListener('beforeunload', (e) => {
  // Force save before leaving
  ReactiveUtils.save(editor);
});
```

### **Example 4: Periodic Manual Save**
```javascript
const gameState = ReactiveUtils.reactive({
  level: 1,
  score: 0,
  position: { x: 0, y: 0 }
});

ReactiveUtils.autoSave(gameState, 'savegame', {
  autoSave: false
});

// Save every 30 seconds
setInterval(() => {
  ReactiveUtils.save(gameState);
  console.log('Game auto-saved');
}, 30000);
```

### **Example 5: Save After Batch Updates**
```javascript
const state = ReactiveUtils.reactive({
  users: [],
  posts: []
});

ReactiveUtils.autoSave(state, 'appData', {
  autoSave: false
});

function loadData(data) {
  // Make multiple updates
  state.users = data.users;
  state.posts = data.posts;

  // Save once after all updates
  ReactiveUtils.save(state);
}
```

### **Example 6: Conditional Save**
```javascript
const config = ReactiveUtils.reactive({
  apiUrl: '',
  timeout: 5000,
  isDirty: false
});

ReactiveUtils.autoSave(config, 'config', {
  autoSave: false
});

function updateConfig(updates) {
  Object.assign(config, updates);
  config.isDirty = true;
}

function saveIfNeeded() {
  if (config.isDirty) {
    ReactiveUtils.save(config);
    config.isDirty = false;
    console.log('Config saved');
  }
}
```

### **Example 7: Save with Confirmation**
```javascript
const data = ReactiveUtils.reactive({
  importantData: null
});

ReactiveUtils.autoSave(data, 'critical', {
  autoSave: false
});

async function saveWithConfirm() {
  if (confirm('Save changes?')) {
    ReactiveUtils.save(data);
    alert('Saved successfully!');
  }
}
```

### **Example 8: Save on Form Submit**
```javascript
const form = ReactiveUtils.reactive({
  name: '',
  email: '',
  message: ''
});

ReactiveUtils.autoSave(form, 'contactForm', {
  autoSave: false
});

document.getElementById('form').addEventListener('submit', (e) => {
  e.preventDefault();

  // Validate
  if (!form.name || !form.email) {
    alert('Please fill all fields');
    return;
  }

  // Save before submitting
  ReactiveUtils.save(form);

  // Submit
  submitForm(form);
});
```

### **Example 9: Manual Save with Feedback**
```javascript
const state = ReactiveUtils.reactive({
  data: []
});

ReactiveUtils.autoSave(state, 'data', {
  autoSave: false,
  onSave: (value) => {
    document.querySelector('.save-indicator').textContent = '💾 Saving...';
    return value;
  }
});

function saveNow() {
  ReactiveUtils.save(state);
  setTimeout(() => {
    document.querySelector('.save-indicator').textContent = '✓ Saved';
  }, 100);
}
```

### **Example 10: Save on Visibility Change**
```javascript
const editor = ReactiveUtils.reactive({
  content: '',
  selection: null
});

ReactiveUtils.autoSave(editor, 'editor', {
  debounce: 1000
});

document.addEventListener('visibilitychange', () => {
  if (document.hidden) {
    // Force save when tab becomes hidden
    ReactiveUtils.save(editor);
    console.log('Saved on tab switch');
  }
});
```

---

## **Common Patterns**

### **Pattern 1: Immediate Save**
```javascript
ReactiveUtils.save(state);
```

### **Pattern 2: Save on Event**
```javascript
button.addEventListener('click', () => {
  ReactiveUtils.save(state);
});
```

### **Pattern 3: Save Before Action**
```javascript
function doSomething() {
  ReactiveUtils.save(state);
  performAction();
}
```

### **Pattern 4: Conditional Save**
```javascript
if (state.isDirty) {
  ReactiveUtils.save(state);
}
```

### **Pattern 5: Save with Feedback**
```javascript
ReactiveUtils.save(state);
showNotification('Saved!');
```

---

## **When to Use**

| Scenario | Use save() |
|----------|------------|
| Manual save control | ✓ Yes |
| Save before navigation | ✓ Yes |
| Save on button click | ✓ Yes |
| Batch updates | ✓ Yes |
| Auto-save disabled | ✓ Yes |
| Bypass debounce | ✓ Yes |
| Automatic saving | ✗ No (use autoSave: true) |

---

## **vs. Auto-Save**

| Feature | `save()` | Auto-Save |
|---------|----------|-----------|
| Trigger | Manual | Automatic |
| Timing | Immediate | Debounced |
| Control | Explicit | Implicit |
| Use Case | Critical saves | Background saves |

```javascript
// Auto-save (automatic)
ReactiveUtils.autoSave(state, 'key', { autoSave: true, debounce: 500 });
state.value = 'new'; // Saves after 500ms

// Manual save (explicit)
ReactiveUtils.autoSave(state, 'key', { autoSave: false });
state.value = 'new'; // Not saved
ReactiveUtils.save(state); // Saves immediately
```

---

## **Best Practices**

1. **Use before critical operations**
   ```javascript
   ReactiveUtils.save(state);
   navigateAway();
   ```

2. **Save before page unload**
   ```javascript
   window.addEventListener('beforeunload', () => {
     ReactiveUtils.save(state);
   });
   ```

3. **Save after batch updates**
   ```javascript
   state.a = 1;
   state.b = 2;
   state.c = 3;
   ReactiveUtils.save(state); // One save for all
   ```

4. **Provide user feedback**
   ```javascript
   ReactiveUtils.save(state);
   showNotification('Saved successfully');
   ```

5. **Combine with validation**
   ```javascript
   if (isValid(state)) {
     ReactiveUtils.save(state);
   }
   ```

6. **Use for important saves**
   ```javascript
   // Auto-save for drafts
   ReactiveUtils.autoSave(draft, 'draft', { debounce: 2000 });

   // Manual save for final submit
   ReactiveUtils.save(finalData);
   ```

---

## **Key Takeaways**

1. **Manual**: Explicitly saves state to storage
2. **Immediate**: No debounce delay
3. **Bypass Auto-Save**: Works even if auto-save disabled
4. **Synchronous**: Completes immediately
5. **Respects onSave**: Still runs onSave transformation
6. **Returns Boolean**: Returns success status
7. **Use for Control**: When you need explicit save timing
8. **Critical Saves**: Use before important operations
9. **User Triggered**: Perfect for save buttons
10. **No Dependencies**: Doesn't require auto-save enabled

---

## **Summary**

`save(state)` is a namespace method that forces an immediate save of reactive state to storage, bypassing debounce delays and auto-save settings. When called, it immediately saves the current state to the configured storage (localStorage or sessionStorage), applying any onSave transformation if configured. The method is synchronous and returns a boolean indicating success or failure. Use `save()` when you need explicit control over save timing, such as before page navigation, on button clicks, after batch updates, or when auto-save is disabled. It's perfect for critical save operations where you can't wait for debounced auto-saves, form submissions where you want to ensure data is saved, or when implementing manual save buttons. The method works whether auto-save is enabled or disabled, making it suitable for hybrid approaches where most saves are automatic but some require immediate persistence.
