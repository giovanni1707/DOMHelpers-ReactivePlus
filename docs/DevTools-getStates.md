# `DevTools.getStates()` - Get All Tracked States

**Quick Start (30 seconds)**
```javascript
ReactiveUtils.DevTools.enable();

// Create and track states
const state1 = ReactiveUtils.state({ count: 0 });
const state2 = ReactiveUtils.state({ name: 'John' });

ReactiveUtils.DevTools.trackState(state1, 'Counter');
ReactiveUtils.DevTools.trackState(state2, 'User');

// Get all tracked states
const states = ReactiveUtils.DevTools.getStates();
console.log(states);
// [
//   { id: 1, name: 'Counter', created: 1234567890, updates: [], state: {...} },
//   { id: 2, name: 'User', created: 1234567891, updates: [], state: {...} }
// ]
```

---

## **What is `DevTools.getStates()`?**

`DevTools.getStates()` is a **method** that returns an array of all tracked state objects with their metadata, including IDs, names, creation timestamps, and update history.

**Key characteristics:**
- **Returns Array**: Array of state entries
- **Includes Metadata**: ID, name, created, updates
- **Includes State**: Reference to actual state object
- **Empty When Disabled**: Returns empty array if disabled
- **Non-Destructive**: Does not modify tracked states
- **Console Friendly**: Perfect for debugging

---

## **Syntax**

```javascript
ReactiveUtils.DevTools.getStates()
```

### **Parameters**
- None

### **Returns**
- **Type**: `Array<Object>`
- **Format**: Array of state entry objects

---

## **How it works**

```javascript
getStates() {
  return Array.from(this.states.entries()).map(([state, info]) => ({
    ...info,
    state
  }));
}
```

**What happens:**
1. Converts states Map to array of entries
2. Maps each entry to object
3. Spreads info object (id, name, created, updates)
4. Adds state reference
5. Returns array

---

## **Return Format**

Each state entry contains:

```javascript
{
  id: 1,                      // Unique ID
  name: 'Counter',            // State name
  created: 1234567890,        // Creation timestamp
  updates: [],                // Array of updates to this state
  state: { count: 0 }         // The actual state object
}
```

---

## **Examples**

### **Example 1: Basic Usage**
```javascript
ReactiveUtils.DevTools.enable();

const state = ReactiveUtils.state({ count: 0 });
ReactiveUtils.DevTools.trackState(state, 'Counter');

const states = ReactiveUtils.DevTools.getStates();
console.log(states);
// [{ id: 1, name: 'Counter', created: ..., updates: [], state: {...} }]
```

### **Example 2: Multiple States**
```javascript
ReactiveUtils.DevTools.enable();

const user = ReactiveUtils.state({ name: 'John', age: 30 });
const app = ReactiveUtils.state({ theme: 'dark', lang: 'en' });
const cart = ReactiveUtils.state({ items: [], total: 0 });

ReactiveUtils.DevTools.trackState(user, 'User');
ReactiveUtils.DevTools.trackState(app, 'App');
ReactiveUtils.DevTools.trackState(cart, 'Cart');

const states = ReactiveUtils.DevTools.getStates();
console.log(`Tracking ${states.length} states`); // 3
```

### **Example 3: Find State by Name**
```javascript
ReactiveUtils.DevTools.enable();

const state1 = ReactiveUtils.state({ a: 1 });
const state2 = ReactiveUtils.state({ b: 2 });
const state3 = ReactiveUtils.state({ c: 3 });

ReactiveUtils.DevTools.trackState(state1, 'State1');
ReactiveUtils.DevTools.trackState(state2, 'State2');
ReactiveUtils.DevTools.trackState(state3, 'State3');

const states = ReactiveUtils.DevTools.getStates();
const state2Info = states.find(s => s.name === 'State2');
console.log(state2Info);
```

### **Example 4: View State Updates**
```javascript
ReactiveUtils.DevTools.enable();

const state = ReactiveUtils.state({ count: 0 });
ReactiveUtils.DevTools.trackState(state, 'Counter');

// Make changes
state.count = 1;
state.count = 2;
state.count = 3;

// View updates for this state
const states = ReactiveUtils.DevTools.getStates();
const counterState = states.find(s => s.name === 'Counter');
console.log(counterState.updates); // Array of 3 updates
```

### **Example 5: Inspect All States**
```javascript
ReactiveUtils.DevTools.enable();

const state1 = ReactiveUtils.state({ value: 0 });
const state2 = ReactiveUtils.state({ value: 0 });

ReactiveUtils.DevTools.trackState(state1, 'State1');
ReactiveUtils.DevTools.trackState(state2, 'State2');

state1.value = 10;
state2.value = 20;

const states = ReactiveUtils.DevTools.getStates();
states.forEach(s => {
  console.log(`[${s.name}] Value: ${s.state.value}, Updates: ${s.updates.length}`);
});
// [State1] Value: 10, Updates: 1
// [State2] Value: 20, Updates: 1
```

### **Example 6: Debug State Creation Time**
```javascript
ReactiveUtils.DevTools.enable();

const state1 = ReactiveUtils.state({ a: 1 });
ReactiveUtils.DevTools.trackState(state1, 'First');

// Wait a moment
setTimeout(() => {
  const state2 = ReactiveUtils.state({ b: 2 });
  ReactiveUtils.DevTools.trackState(state2, 'Second');

  const states = ReactiveUtils.DevTools.getStates();
  states.forEach(s => {
    console.log(`${s.name} created at: ${new Date(s.created).toISOString()}`);
  });
}, 1000);
```

### **Example 7: Empty When Disabled**
```javascript
ReactiveUtils.DevTools.disable();

const state = ReactiveUtils.state({ count: 0 });
ReactiveUtils.DevTools.trackState(state, 'Counter');

const states = ReactiveUtils.DevTools.getStates();
console.log(states.length); // 0 (DevTools disabled)
```

### **Example 8: Filter States**
```javascript
ReactiveUtils.DevTools.enable();

const state1 = ReactiveUtils.state({ value: 0 });
const state2 = ReactiveUtils.state({ value: 0 });
const state3 = ReactiveUtils.state({ value: 0 });

ReactiveUtils.DevTools.trackState(state1, 'UserState');
ReactiveUtils.DevTools.trackState(state2, 'AppState');
ReactiveUtils.DevTools.trackState(state3, 'CartState');

state1.value = 1;
state2.value = 2;
// state3 unchanged

// Find states with updates
const states = ReactiveUtils.DevTools.getStates();
const statesWithUpdates = states.filter(s => s.updates.length > 0);
console.log(statesWithUpdates.map(s => s.name)); // ['UserState', 'AppState']
```

### **Example 9: Console Debugging**
```javascript
ReactiveUtils.DevTools.enable();

const state = ReactiveUtils.state({
  user: { name: 'John', age: 30 },
  settings: { theme: 'dark' }
});

ReactiveUtils.DevTools.trackState(state, 'AppState');

// Make changes
state.user.name = 'Jane';
state.settings.theme = 'light';

// Debug from console
const states = window.__REACTIVE_DEVTOOLS__.getStates();
console.table(states.map(s => ({
  ID: s.id,
  Name: s.name,
  Updates: s.updates.length,
  Created: new Date(s.created).toLocaleTimeString()
})));
```

### **Example 10: State Dashboard**
```javascript
ReactiveUtils.DevTools.enable();

function showStateDashboard() {
  const states = ReactiveUtils.DevTools.getStates();

  console.log('=== State Dashboard ===');
  console.log(`Total States: ${states.length}`);

  states.forEach(s => {
    console.log(`\n[${s.id}] ${s.name}`);
    console.log(`  Created: ${new Date(s.created).toLocaleString()}`);
    console.log(`  Updates: ${s.updates.length}`);
    console.log(`  Current Value:`, s.state);
  });
}

// Create states
const user = ReactiveUtils.state({ name: 'John' });
const app = ReactiveUtils.state({ theme: 'dark' });

ReactiveUtils.DevTools.trackState(user, 'User');
ReactiveUtils.DevTools.trackState(app, 'App');

// Show dashboard
showStateDashboard();
```

---

## **Common Patterns**

### **Pattern 1: Basic Inspection**
```javascript
const states = ReactiveUtils.DevTools.getStates();
console.log(states);
```

### **Pattern 2: Find Specific State**
```javascript
const states = ReactiveUtils.DevTools.getStates();
const myState = states.find(s => s.name === 'MyState');
```

### **Pattern 3: Filter States**
```javascript
const states = ReactiveUtils.DevTools.getStates();
const activeStates = states.filter(s => s.updates.length > 0);
```

---

## **State Entry Properties**

| Property | Type | Description |
|----------|------|-------------|
| `id` | `number` | Unique state ID |
| `name` | `string` | State name |
| `created` | `number` | Creation timestamp |
| `updates` | `Array` | Array of updates to this state |
| `state` | `Object` | The actual state object |

---

## **Console Access**

Available via global reference:

```javascript
// After enabling DevTools
window.__REACTIVE_DEVTOOLS__.getStates()
```

---

## **Key Takeaways**

1. **Returns Array**: Array of all tracked states
2. **Includes Metadata**: ID, name, timestamps, updates
3. **State Reference**: Includes actual state object
4. **Empty When Disabled**: Returns empty array if disabled
5. **Non-Destructive**: Does not modify states
6. **Console Friendly**: Perfect for debugging
7. **Filterable**: Can filter/find specific states

---

## **Summary**

`DevTools.getStates()` returns an array of all tracked state objects with their complete metadata, including unique IDs, names, creation timestamps, update history, and references to the actual state objects. Use it to inspect all tracked states during debugging, find specific states by name, view update history, and understand the current state of your reactive system. The method returns an empty array when DevTools is disabled and is available globally via `window.__REACTIVE_DEVTOOLS__` for console debugging.
