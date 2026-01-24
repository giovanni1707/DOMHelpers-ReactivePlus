# `DevTools.trackState()` - Track Reactive State

**Quick Start (30 seconds)**
```javascript
ReactiveUtils.DevTools.enable();

// Create state
const counter = ReactiveUtils.state({ count: 0 });

// Track it
ReactiveUtils.DevTools.trackState(counter, 'Counter');

// Make changes - automatically logged
counter.count = 1;
counter.count = 2;
counter.count = 3;

// View tracked state
const states = ReactiveUtils.DevTools.getStates();
console.log(states[0]);
// {
//   id: 1,
//   name: 'Counter',
//   created: 1234567890,
//   updates: [...],
//   state: { count: 3 }
// }
```

---

## **What is `DevTools.trackState()`?**

`DevTools.trackState()` is a **method** that registers a reactive state object for tracking, logging all changes made to it.

**Key characteristics:**
- **Registers State**: Adds state to tracking system
- **Auto-Logging**: Changes are automatically logged
- **Named Tracking**: Assign readable names to states
- **Unique ID**: Each state gets unique ID
- **History**: All changes stored in history
- **No-Op When Disabled**: Does nothing if DevTools disabled

---

## **Syntax**

```javascript
ReactiveUtils.DevTools.trackState(state, name?)
```

### **Parameters**
- `state` (required) - Reactive state object
  - **Type**: `Object` (created with `state()`)
- `name` (optional) - Human-readable name
  - **Type**: `string`
  - **Default**: `'State {id}'`

### **Returns**
- **Type**: `void`

---

## **How it works**

```javascript
trackState(state, name) {
  if (!this.enabled) return;
  
  const id = this.states.size + 1;
  this.states.set(state, {
    id,
    name: name || `State ${id}`,
    created: Date.now(),
    updates: []
  });
}
```

---

## **Examples**

### **Example 1: Basic Tracking**
```javascript
ReactiveUtils.DevTools.enable();

const state = ReactiveUtils.state({ count: 0 });
ReactiveUtils.DevTools.trackState(state, 'Counter');

state.count++; // Logged to DevTools
```

### **Example 2: Multiple States**
```javascript
ReactiveUtils.DevTools.enable();

const user = ReactiveUtils.state({ name: '', email: '' });
const app = ReactiveUtils.state({ theme: 'light', lang: 'en' });
const cart = ReactiveUtils.state({ items: [], total: 0 });

ReactiveUtils.DevTools.trackState(user, 'User');
ReactiveUtils.DevTools.trackState(app, 'App');
ReactiveUtils.DevTools.trackState(cart, 'Cart');

// All changes to these states are logged
```

### **Example 3: Without Name**
```javascript
ReactiveUtils.DevTools.enable();

const state = ReactiveUtils.state({ value: 0 });
ReactiveUtils.DevTools.trackState(state); // Name: "State 1"

// View state
const states = ReactiveUtils.DevTools.getStates();
console.log(states[0].name); // "State 1"
```

### **Example 4: Form Tracking**
```javascript
ReactiveUtils.DevTools.enable();

const loginForm = Forms.createForm({ 
  email: '', 
  password: '' 
});

ReactiveUtils.DevTools.trackState(loginForm, 'LoginForm');

// Track form changes
loginForm.setValue('email', 'user@example.com');
loginForm.setValue('password', 'secret');

// View history
console.log(ReactiveUtils.DevTools.getHistory());
```

### **Example 5: Component State**
```javascript
class Component {
  constructor(name) {
    this.state = ReactiveUtils.state({ 
      data: null,
      loading: false,
      error: null
    });
    
    if (ReactiveUtils.DevTools.enabled) {
      ReactiveUtils.DevTools.trackState(this.state, `${name}State`);
    }
  }
}

const header = new Component('Header');
const sidebar = new Component('Sidebar');
const content = new Component('Content');
```

### **Example 6: Nested State**
```javascript
ReactiveUtils.DevTools.enable();

const appState = ReactiveUtils.state({
  user: { name: 'John', age: 30 },
  settings: { theme: 'dark', lang: 'en' },
  cart: { items: [], total: 0 }
});

ReactiveUtils.DevTools.trackState(appState, 'AppState');

// All nested changes are logged
appState.user.name = 'Jane';
appState.settings.theme = 'light';
appState.cart.items.push({ id: 1, name: 'Item' });
```

### **Example 7: View Tracked Updates**
```javascript
ReactiveUtils.DevTools.enable();

const state = ReactiveUtils.state({ count: 0 });
ReactiveUtils.DevTools.trackState(state, 'Counter');

state.count = 1;
state.count = 2;
state.count = 3;

// View state's updates
const states = ReactiveUtils.DevTools.getStates();
const counterState = states.find(s => s.name === 'Counter');
console.log(counterState.updates);
```

### **Example 8: Conditional Tracking**
```javascript
function createState(initialValue, name, track = false) {
  const state = ReactiveUtils.state(initialValue);
  
  if (track && ReactiveUtils.DevTools.enabled) {
    ReactiveUtils.DevTools.trackState(state, name);
  }
  
  return state;
}

const tracked = createState({ a: 1 }, 'Tracked', true);
const untracked = createState({ b: 2 }, 'Untracked', false);
```

### **Example 9: Debug Specific State**
```javascript
ReactiveUtils.DevTools.enable();

const state1 = ReactiveUtils.state({ value: 0 });
const state2 = ReactiveUtils.state({ value: 0 });
const state3 = ReactiveUtils.state({ value: 0 });

// Only track state2 for debugging
ReactiveUtils.DevTools.trackState(state2, 'DebugState');

state1.value++; // Not logged
state2.value++; // Logged
state3.value++; // Not logged
```

### **Example 10: Store Pattern**
```javascript
class Store {
  constructor(name, initialState) {
    this.state = ReactiveUtils.state(initialState);
    
    if (ReactiveUtils.DevTools.enabled) {
      ReactiveUtils.DevTools.trackState(this.state, `${name}Store`);
    }
  }
}

const userStore = new Store('User', { name: '', email: '' });
const cartStore = new Store('Cart', { items: [], total: 0 });
const appStore = new Store('App', { theme: 'light', lang: 'en' });

// All stores are tracked
```

---

## **Common Patterns**

### **Pattern 1: Track All States**
```javascript
ReactiveUtils.DevTools.enable();
ReactiveUtils.DevTools.trackState(state, 'StateName');
```

### **Pattern 2: Conditional Tracking**
```javascript
if (ReactiveUtils.DevTools.enabled) {
  ReactiveUtils.DevTools.trackState(state, 'StateName');
}
```

### **Pattern 3: Factory Function**
```javascript
function createTrackedState(init, name) {
  const state = ReactiveUtils.state(init);
  ReactiveUtils.DevTools.trackState(state, name);
  return state;
}
```

---

## **Tracked State Entry**

Each tracked state creates an entry:

```javascript
{
  id: 1,                      // Unique ID
  name: 'Counter',            // State name
  created: 1234567890,        // Creation timestamp
  updates: [],                // Array of updates
  state: { count: 0 }         // The state object
}
```

---

## **Key Takeaways**

1. **Registers State**: Adds to tracking system
2. **Auto-Logging**: Changes automatically logged
3. **Named**: Provide readable names for debugging
4. **Unique IDs**: Each state gets unique identifier
5. **Optional Name**: Auto-generates if not provided
6. **No-Op When Disabled**: Safe to call when disabled
7. **Common Use**: Debug state changes, monitor updates

---

## **Summary**

`DevTools.trackState()` registers a reactive state object for tracking, assigning it a unique ID and optional name. Once tracked, all changes to the state are automatically logged to DevTools history with timestamps. Use it to debug specific states, monitor state changes during development, and understand reactive behavior. The method is a no-op when DevTools is disabled, making it safe to leave in code during production builds.
