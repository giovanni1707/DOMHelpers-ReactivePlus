# `$save()` - Force Save State to Storage

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
state.$save();
console.log('Saved immediately!');
```

---

## **What is `$save()`?**

`$save()` is a **method added to reactive state objects** by `autoSave()` that forces an immediate save to storage, bypassing debounce delays and auto-save settings.

**Key characteristics:**
- **Immediate**: Saves immediately, no debounce delay
- **Manual Control**: Explicit save operation
- **Bypasses Auto-Save**: Works even if autoSave is disabled
- **Respects Hooks**: Still runs onSave callback if configured
- **Returns Void**: No return value
- **Synchronous**: Completes before continuing

---

## **Syntax**

```javascript
state.$save()
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

// This method is added to state:
state.$save = function() {
  let valueToSave = toRaw(state);

  // Apply onSave transform if configured
  if (options.onSave) {
    valueToSave = options.onSave(valueToSave);
  }

  // Save to storage immediately
  store.set(key, valueToSave, { expires: options.expires });

  // No debounce, no delay
};
```

**What happens:**
1. Gets raw (non-reactive) copy of state
2. Applies onSave transformation if configured
3. Saves to storage immediately
4. Bypasses any debounce delay
5. Returns immediately

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
settings.$save();
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
  formData.$save();
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
  editor.$save();
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
  gameState.$save();
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
  state.$save();
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
    config.$save();
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
    data.$save();
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
  form.$save();

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
  state.$save();
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
    editor.$save();
    console.log('Saved on tab switch');
  }
});
```

---

## **Common Patterns**

### **Pattern 1: Immediate Save**
```javascript
state.$save();
```

### **Pattern 2: Save on Event**
```javascript
button.addEventListener('click', () => {
  state.$save();
});
```

### **Pattern 3: Save Before Action**
```javascript
function doSomething() {
  state.$save();
  performAction();
}
```

### **Pattern 4: Conditional Save**
```javascript
if (state.isDirty) {
  state.$save();
}
```

### **Pattern 5: Save with Feedback**
```javascript
state.$save();
showNotification('Saved!');
```

---

## **When to Use**

| Scenario | Use $save() |
|----------|-------------|
| Manual save control | ✓ Yes |
| Save before navigation | ✓ Yes |
| Save on button click | ✓ Yes |
| Batch updates | ✓ Yes |
| Auto-save disabled | ✓ Yes |
| Bypass debounce | ✓ Yes |
| Automatic saving | ✗ No (use autoSave: true) |

---

## **vs. Auto-Save**

| Feature | `$save()` | Auto-Save |
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
state.$save(); // Saves immediately
```

---

## **Best Practices**

1. **Use before critical operations**
   ```javascript
   state.$save();
   navigateAway();
   ```

2. **Save before page unload**
   ```javascript
   window.addEventListener('beforeunload', () => {
     state.$save();
   });
   ```

3. **Save after batch updates**
   ```javascript
   state.a = 1;
   state.b = 2;
   state.c = 3;
   state.$save(); // One save for all
   ```

4. **Provide user feedback**
   ```javascript
   state.$save();
   showNotification('Saved successfully');
   ```

5. **Combine with validation**
   ```javascript
   if (isValid(state)) {
     state.$save();
   }
   ```

6. **Use for important saves**
   ```javascript
   // Auto-save for drafts
   ReactiveUtils.autoSave(draft, 'draft', { debounce: 2000 });

   // Manual save for final submit
   finalData.$save();
   ```

---

## **Key Takeaways**

1. **Manual**: Explicitly saves state to storage
2. **Immediate**: No debounce delay
3. **Bypass Auto-Save**: Works even if auto-save disabled
4. **Synchronous**: Completes immediately
5. **Respects onSave**: Still runs onSave transformation
6. **No Return**: Returns void
7. **Use for Control**: When you need explicit save timing
8. **Critical Saves**: Use before important operations
9. **User Triggered**: Perfect for save buttons
10. **No Dependencies**: Doesn't require auto-save enabled

---

## **Summary**

`$save()` is a method added to reactive state objects by `autoSave()` that forces an immediate save to storage, bypassing debounce delays and auto-save settings. When called, it immediately saves the current state to the configured storage (localStorage or sessionStorage), applying any onSave transformation if configured. The method is synchronous and returns immediately after saving. Use `$save()` when you need explicit control over save timing, such as before page navigation, on button clicks, after batch updates, or when auto-save is disabled. It's perfect for critical save operations where you can't wait for debounced auto-saves, form submissions where you want to ensure data is saved, or when implementing manual save buttons. The method works whether auto-save is enabled or disabled, making it suitable for hybrid approaches where most saves are automatic but some require immediate persistence.
