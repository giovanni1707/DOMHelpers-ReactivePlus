# `DevTools` - Development Tools for Reactive State

**Quick Start (30 seconds)**
```javascript
// Enable DevTools
ReactiveUtils.DevTools.enable();

// Create and track states
const state1 = ReactiveUtils.state({ count: 0 });
ReactiveUtils.DevTools.trackState(state1, 'Counter');

const state2 = ReactiveUtils.state({ name: '' });
ReactiveUtils.DevTools.trackState(state2, 'User');

// Track effects
const dispose = effect(() => {
  console.log('Count:', state1.count);
});
ReactiveUtils.DevTools.trackEffect(dispose, 'CountLogger');

// Make changes - automatically logged
state1.count++; // Logged to DevTools

// Inspect via console
console.log(ReactiveUtils.DevTools.getStates());
console.log(ReactiveUtils.DevTools.getHistory());

// Access via global
window.__REACTIVE_DEVTOOLS__.getHistory();
```

---

## **What is `DevTools`?**

`DevTools` is a **development tools object** for debugging reactive state, tracking effects, and inspecting state changes. It provides visibility into your reactive system during development.

**Key characteristics:**
- **Development Only**: For debugging and development
- **State Tracking**: Monitor state objects and their changes
- **Effect Tracking**: Track effect creation and execution
- **History Logging**: Record all state changes with timestamps
- **Global Access**: Available via `window.__REACTIVE_DEVTOOLS__`
- **Auto-Enable**: Enabled on localhost by default

---

## **DevTools API**

| Method | Description |
|--------|-------------|
| `enable()` | Enable DevTools and expose globally |
| `disable()` | Disable DevTools and remove global reference |
| `trackState(state, name)` | Register state for tracking |
| `trackEffect(effect, name)` | Register effect for tracking |
| `getStates()` | Get array of all tracked states |
| `getHistory()` | Get array of all logged state changes |
| `clearHistory()` | Clear DevTools history |

---

## **Properties**

| Property | Type | Description |
|----------|------|-------------|
| `enabled` | `boolean` | Whether DevTools is enabled |
| `states` | `Map` | Map of tracked states |
| `effects` | `Map` | Map of tracked effects |
| `history` | `Array` | Array of logged changes |
| `maxHistory` | `number` | Max history entries (default: 50) |

---

## **How it works**

```javascript
const DevTools = {
  enabled: false,
  states: new Map(),
  effects: new Map(),
  history: [],
  maxHistory: 50,
  
  enable() {
    this.enabled = true;
    window.__REACTIVE_DEVTOOLS__ = this;
  },
  
  disable() {
    this.enabled = false;
    delete window.__REACTIVE_DEVTOOLS__;
  },
  
  trackState(state, name) {
    if (!this.enabled) return;
    
    const id = this.states.size + 1;
    this.states.set(state, {
      id,
      name: name || `State ${id}`,
      created: Date.now(),
      updates: []
    });
  },
  
  // ... more methods
};
```

---

## **Auto-Enable on Localhost**

DevTools automatically enables on localhost:

```javascript
if (typeof window !== 'undefined' && 
    (window.location.hostname === 'localhost' || 
     window.location.hostname === '127.0.0.1')) {
  DevTools.enable();
}
```

---

## **Examples**

### **Example 1: Basic Setup**
```javascript
// Enable DevTools
ReactiveUtils.DevTools.enable();

// Create and track state
const state = ReactiveUtils.state({ count: 0 });
ReactiveUtils.DevTools.trackState(state, 'Counter');

// Make changes
state.count = 1;
state.count = 2;
state.count = 3;

// View history
console.log(ReactiveUtils.DevTools.getHistory());
```

### **Example 2: Track Multiple States**
```javascript
ReactiveUtils.DevTools.enable();

const userState = ReactiveUtils.state({ name: '', email: '' });
const appState = ReactiveUtils.state({ theme: 'light', lang: 'en' });

ReactiveUtils.DevTools.trackState(userState, 'User');
ReactiveUtils.DevTools.trackState(appState, 'App');

// View tracked states
console.log(ReactiveUtils.DevTools.getStates());
```

### **Example 3: Track Effects**
```javascript
ReactiveUtils.DevTools.enable();

const state = ReactiveUtils.state({ count: 0 });

const dispose1 = effect(() => {
  console.log('Effect 1:', state.count);
});

const dispose2 = effect(() => {
  console.log('Effect 2:', state.count * 2);
});

ReactiveUtils.DevTools.trackEffect(dispose1, 'Logger');
ReactiveUtils.DevTools.trackEffect(dispose2, 'Doubler');
```

### **Example 4: Debug State Changes**
```javascript
ReactiveUtils.DevTools.enable();

const state = ReactiveUtils.state({ 
  user: { name: 'John', age: 30 },
  settings: { theme: 'dark' }
});

ReactiveUtils.DevTools.trackState(state, 'AppState');

// Make changes
state.user.name = 'Jane';
state.user.age = 31;
state.settings.theme = 'light';

// Inspect history
const history = ReactiveUtils.DevTools.getHistory();
history.forEach(change => {
  console.log(`[${change.stateName}] ${change.key}:`, 
              change.oldValue, '→', change.newValue);
});
```

### **Example 5: Monitor Form State**
```javascript
ReactiveUtils.DevTools.enable();

const form = Forms.createForm({ 
  email: '', 
  password: '' 
});

ReactiveUtils.DevTools.trackState(form, 'LoginForm');

// Watch form changes
form.setValue('email', 'user@example.com');
form.setValue('password', 'secret123');

// View changes
console.log(ReactiveUtils.DevTools.getHistory());
```

### **Example 6: Clear History**
```javascript
ReactiveUtils.DevTools.enable();

const state = ReactiveUtils.state({ count: 0 });
ReactiveUtils.DevTools.trackState(state, 'Counter');

// Make many changes
for (let i = 0; i < 100; i++) {
  state.count = i;
}

console.log(ReactiveUtils.DevTools.getHistory().length); // 50 (max)

// Clear history
ReactiveUtils.DevTools.clearHistory();

console.log(ReactiveUtils.DevTools.getHistory().length); // 0
```

### **Example 7: Global Access**
```javascript
ReactiveUtils.DevTools.enable();

// Access via global
window.__REACTIVE_DEVTOOLS__.getStates();
window.__REACTIVE_DEVTOOLS__.getHistory();
window.__REACTIVE_DEVTOOLS__.clearHistory();

// Useful for browser console debugging
```

### **Example 8: Production Disable**
```javascript
if (process.env.NODE_ENV === 'production') {
  ReactiveUtils.DevTools.disable();
} else {
  ReactiveUtils.DevTools.enable();
}
```

### **Example 9: Custom History Limit**
```javascript
ReactiveUtils.DevTools.enable();
ReactiveUtils.DevTools.maxHistory = 100; // Increase limit

const state = ReactiveUtils.state({ count: 0 });
ReactiveUtils.DevTools.trackState(state, 'Counter');

// Make many changes
for (let i = 0; i < 150; i++) {
  state.count = i;
}

console.log(ReactiveUtils.DevTools.getHistory().length); // 100 (custom max)
```

### **Example 10: Debug Effect Performance**
```javascript
ReactiveUtils.DevTools.enable();

const state = ReactiveUtils.state({ data: [] });

let runCount = 0;
const dispose = effect(() => {
  runCount++;
  console.log('Effect run #', runCount);
  processData(state.data);
});

ReactiveUtils.DevTools.trackEffect(dispose, 'DataProcessor');

// Track how many times effect runs
state.data = [1, 2, 3];
state.data = [4, 5, 6];
state.data = [7, 8, 9];

console.log(`Effect ran ${runCount} times`);
```

---

## **Common Patterns**

### **Pattern 1: Enable in Development**
```javascript
if (process.env.NODE_ENV === 'development') {
  ReactiveUtils.DevTools.enable();
}
```

### **Pattern 2: Track All States**
```javascript
function createTrackedState(initialValue, name) {
  const state = ReactiveUtils.state(initialValue);
  ReactiveUtils.DevTools.trackState(state, name);
  return state;
}
```

### **Pattern 3: Debug Changes**
```javascript
ReactiveUtils.DevTools.enable();
// ... make changes
console.log(ReactiveUtils.DevTools.getHistory());
```

---

## **Console Usage**

When DevTools is enabled, you can use it from the browser console:

```javascript
// Get all tracked states
__REACTIVE_DEVTOOLS__.getStates()

// Get change history
__REACTIVE_DEVTOOLS__.getHistory()

// Clear history
__REACTIVE_DEVTOOLS__.clearHistory()

// Check if enabled
__REACTIVE_DEVTOOLS__.enabled
```

---

## **History Entry Format**

Each history entry contains:

```javascript
{
  stateId: 1,              // State ID
  stateName: 'Counter',    // State name
  key: 'count',            // Changed property
  oldValue: 0,             // Previous value
  newValue: 1,             // New value
  timestamp: 1234567890    // When changed
}
```

---

## **State Entry Format**

Each tracked state entry contains:

```javascript
{
  id: 1,                   // Unique ID
  name: 'Counter',         // State name
  created: 1234567890,     // Creation timestamp
  updates: [],             // Array of updates
  state: { /* ... */ }     // The state object
}
```

---

## **Performance Considerations**

- **Development Only**: Disable in production for performance
- **History Limit**: Prevents memory growth (default: 50 entries)
- **Overhead**: Tracking adds small overhead to state changes
- **WeakMaps**: Uses WeakMaps to avoid memory leaks

---

## **Key Takeaways**

1. **Development Tool**: For debugging reactive state
2. **Auto-Enable**: Enabled on localhost automatically
3. **State Tracking**: Monitor state objects and changes
4. **Effect Tracking**: Track effect creation and execution
5. **History Logging**: Records all changes with timestamps
6. **Global Access**: Available via `window.__REACTIVE_DEVTOOLS__`
7. **History Limit**: Prevents memory growth
8. **Production**: Disable in production builds

---

## **Summary**

`DevTools` is a development tools object for debugging reactive state, providing visibility into state changes, effect execution, and reactive behavior. It automatically enables on localhost, tracks state objects and effects, logs all changes with timestamps, and exposes a global API for console access. Use it during development to understand reactive behavior, debug state changes, monitor effect execution, and inspect your reactive system. Always disable in production for optimal performance.

---

## **See Also**

- `DevTools.enable()` - Enable DevTools
- `DevTools.disable()` - Disable DevTools
- `DevTools.trackState()` - Track state object
- `DevTools.trackEffect()` - Track effect function
- `DevTools.getStates()` - Get tracked states
- `DevTools.getHistory()` - Get change history
- `DevTools.clearHistory()` - Clear history
