# `DevTools.getHistory()` - Get State Change History

**Quick Start (30 seconds)**
```javascript
ReactiveUtils.DevTools.enable();

// Create and track state
const state = ReactiveUtils.state({ count: 0, name: 'John' });
ReactiveUtils.DevTools.trackState(state, 'MyState');

// Make changes
state.count = 1;
state.count = 2;
state.name = 'Jane';

// Get history
const history = ReactiveUtils.DevTools.getHistory();
console.log(history);
// [
//   { stateId: 1, stateName: 'MyState', key: 'count', oldValue: 0, newValue: 1, timestamp: ... },
//   { stateId: 1, stateName: 'MyState', key: 'count', oldValue: 1, newValue: 2, timestamp: ... },
//   { stateId: 1, stateName: 'MyState', key: 'name', oldValue: 'John', newValue: 'Jane', timestamp: ... }
// ]
```

---

## **What is `DevTools.getHistory()`?**

`DevTools.getHistory()` is a **method** that returns an array of all logged state changes with complete details about what changed, when, and in which state.

**Key characteristics:**
- **Returns Array**: Array of change entries
- **Complete Details**: State ID, name, key, old/new values, timestamp
- **Chronological**: Ordered by time of change
- **Limited Size**: Max 50 entries (configurable)
- **Empty When Disabled**: Returns empty array if disabled
- **Non-Destructive**: Does not modify history

---

## **Syntax**

```javascript
ReactiveUtils.DevTools.getHistory()
```

### **Parameters**
- None

### **Returns**
- **Type**: `Array<Object>`
- **Format**: Array of change entry objects

---

## **How it works**

```javascript
getHistory() {
  return [...this.history];
}
```

**What happens:**
1. Creates shallow copy of history array
2. Returns the copy
3. Original history unchanged

---

## **History Entry Format**

Each history entry contains:

```javascript
{
  stateId: 1,                  // State ID
  stateName: 'Counter',        // State name
  key: 'count',                // Changed property
  oldValue: 0,                 // Previous value
  newValue: 1,                 // New value
  timestamp: 1234567890        // When changed (Date.now())
}
```

---

## **Examples**

### **Example 1: Basic History**
```javascript
ReactiveUtils.DevTools.enable();

const state = ReactiveUtils.state({ count: 0 });
ReactiveUtils.DevTools.trackState(state, 'Counter');

state.count = 1;
state.count = 2;
state.count = 3;

const history = ReactiveUtils.DevTools.getHistory();
console.log(`${history.length} changes logged`); // 3
```

### **Example 2: View All Changes**
```javascript
ReactiveUtils.DevTools.enable();

const state = ReactiveUtils.state({ count: 0, name: 'John' });
ReactiveUtils.DevTools.trackState(state, 'MyState');

state.count = 10;
state.name = 'Jane';
state.count = 20;

const history = ReactiveUtils.DevTools.getHistory();
history.forEach(change => {
  console.log(`[${change.stateName}] ${change.key}: ${change.oldValue} → ${change.newValue}`);
});
// [MyState] count: 0 → 10
// [MyState] name: John → Jane
// [MyState] count: 10 → 20
```

### **Example 3: Filter by State**
```javascript
ReactiveUtils.DevTools.enable();

const state1 = ReactiveUtils.state({ value: 0 });
const state2 = ReactiveUtils.state({ value: 0 });

ReactiveUtils.DevTools.trackState(state1, 'State1');
ReactiveUtils.DevTools.trackState(state2, 'State2');

state1.value = 1;
state2.value = 2;
state1.value = 3;

const history = ReactiveUtils.DevTools.getHistory();
const state1Changes = history.filter(h => h.stateName === 'State1');
console.log(`State1 changes: ${state1Changes.length}`); // 2
```

### **Example 4: View Timestamps**
```javascript
ReactiveUtils.DevTools.enable();

const state = ReactiveUtils.state({ count: 0 });
ReactiveUtils.DevTools.trackState(state, 'Counter');

state.count = 1;
state.count = 2;
state.count = 3;

const history = ReactiveUtils.DevTools.getHistory();
history.forEach(change => {
  const time = new Date(change.timestamp).toLocaleTimeString();
  console.log(`${time}: ${change.key} = ${change.newValue}`);
});
```

### **Example 5: Nested State Changes**
```javascript
ReactiveUtils.DevTools.enable();

const state = ReactiveUtils.state({
  user: { name: 'John', age: 30 },
  settings: { theme: 'dark', lang: 'en' }
});

ReactiveUtils.DevTools.trackState(state, 'AppState');

state.user.name = 'Jane';
state.user.age = 31;
state.settings.theme = 'light';

const history = ReactiveUtils.DevTools.getHistory();
console.log(history);
// Shows changes to nested properties
```

### **Example 6: Multiple States**
```javascript
ReactiveUtils.DevTools.enable();

const user = ReactiveUtils.state({ name: 'John' });
const app = ReactiveUtils.state({ theme: 'dark' });

ReactiveUtils.DevTools.trackState(user, 'User');
ReactiveUtils.DevTools.trackState(app, 'App');

user.name = 'Jane';
app.theme = 'light';
user.name = 'Bob';

const history = ReactiveUtils.DevTools.getHistory();
console.log(`Total changes: ${history.length}`); // 3

const byState = {};
history.forEach(h => {
  byState[h.stateName] = (byState[h.stateName] || 0) + 1;
});
console.log(byState); // { User: 2, App: 1 }
```

### **Example 7: Recent Changes**
```javascript
ReactiveUtils.DevTools.enable();

const state = ReactiveUtils.state({ value: 0 });
ReactiveUtils.DevTools.trackState(state, 'MyState');

for (let i = 1; i <= 10; i++) {
  state.value = i;
}

// Get last 5 changes
const history = ReactiveUtils.DevTools.getHistory();
const recent = history.slice(-5);
console.log('Recent changes:', recent);
```

### **Example 8: Empty When Disabled**
```javascript
ReactiveUtils.DevTools.disable();

const state = ReactiveUtils.state({ count: 0 });
ReactiveUtils.DevTools.trackState(state, 'Counter');

state.count = 1;
state.count = 2;

const history = ReactiveUtils.DevTools.getHistory();
console.log(history.length); // 0 (DevTools disabled)
```

### **Example 9: Change Timeline**
```javascript
ReactiveUtils.DevTools.enable();

const state = ReactiveUtils.state({
  status: 'idle',
  data: null,
  error: null
});

ReactiveUtils.DevTools.trackState(state, 'AsyncState');

// Simulate async flow
state.status = 'loading';
setTimeout(() => {
  state.data = { id: 1, name: 'Item' };
  state.status = 'success';
}, 100);

// Later, view timeline
setTimeout(() => {
  const history = ReactiveUtils.DevTools.getHistory();
  console.log('Timeline:');
  history.forEach(change => {
    const time = new Date(change.timestamp).toLocaleTimeString();
    console.log(`  ${time}: ${change.key} = ${change.newValue}`);
  });
}, 200);
```

### **Example 10: Debug State Flow**
```javascript
ReactiveUtils.DevTools.enable();

function debugStateFlow() {
  const history = ReactiveUtils.DevTools.getHistory();

  console.log('=== State Change Flow ===');
  console.log(`Total Changes: ${history.length}\n`);

  history.forEach((change, index) => {
    console.log(`[${index + 1}] ${change.stateName}.${change.key}`);
    console.log(`    ${change.oldValue} → ${change.newValue}`);
    console.log(`    @ ${new Date(change.timestamp).toLocaleTimeString()}\n`);
  });
}

// Create state
const state = ReactiveUtils.state({
  step: 1,
  message: 'Starting'
});

ReactiveUtils.DevTools.trackState(state, 'Wizard');

// Simulate wizard flow
state.step = 2;
state.message = 'Step 2';
state.step = 3;
state.message = 'Complete';

// Debug
debugStateFlow();
```

---

## **Common Patterns**

### **Pattern 1: Basic History**
```javascript
const history = ReactiveUtils.DevTools.getHistory();
console.log(history);
```

### **Pattern 2: Filter by State**
```javascript
const history = ReactiveUtils.DevTools.getHistory();
const filtered = history.filter(h => h.stateName === 'MyState');
```

### **Pattern 3: Recent Changes**
```javascript
const history = ReactiveUtils.DevTools.getHistory();
const recent = history.slice(-10);
```

---

## **History Entry Properties**

| Property | Type | Description |
|----------|------|-------------|
| `stateId` | `number` | ID of the state |
| `stateName` | `string` | Name of the state |
| `key` | `string` | Property that changed |
| `oldValue` | `any` | Previous value |
| `newValue` | `any` | New value |
| `timestamp` | `number` | When changed (Date.now()) |

---

## **History Limit**

- **Default**: 50 entries
- **Configurable**: `ReactiveUtils.DevTools.maxHistory = 100`
- **Behavior**: Old entries removed when limit reached

```javascript
ReactiveUtils.DevTools.maxHistory = 100; // Increase limit
```

---

## **Console Access**

Available via global reference:

```javascript
// After enabling DevTools
window.__REACTIVE_DEVTOOLS__.getHistory()
```

---

## **Key Takeaways**

1. **Returns Array**: Array of all change entries
2. **Complete Details**: State info, key, old/new values, timestamp
3. **Chronological**: Ordered by time
4. **Limited Size**: Max 50 entries (configurable)
5. **Empty When Disabled**: Returns empty array if disabled
6. **Non-Destructive**: Does not modify history
7. **Filterable**: Can filter by state, key, or time

---

## **Summary**

`DevTools.getHistory()` returns an array of all logged state changes, providing complete details about what changed, when, and in which state. Each entry includes the state ID and name, the property key that changed, old and new values, and a timestamp. The history is limited to 50 entries by default (configurable via `maxHistory`) to prevent memory growth. Use it to debug state changes, understand state flow, view change timelines, and track down unexpected updates. The method is available globally via `window.__REACTIVE_DEVTOOLS__` for console debugging.
