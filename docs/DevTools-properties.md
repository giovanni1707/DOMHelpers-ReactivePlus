# DevTools Properties - Complete Reference

**Quick Start (30 seconds)**
```javascript
ReactiveUtils.DevTools.enable();

const state = ReactiveUtils.state({ count: 0 });
ReactiveUtils.DevTools.trackState(state, 'Counter');

// Check if enabled
console.log(ReactiveUtils.DevTools.enabled); // true

// View tracked states
console.log(ReactiveUtils.DevTools.states.size); // 1

// View tracked effects
console.log(ReactiveUtils.DevTools.effects.size); // 0

// Make changes and view history
state.count = 1;
state.count = 2;
console.log(ReactiveUtils.DevTools.history.length); // 2

// Check maximum history size
console.log(ReactiveUtils.DevTools.maxHistory); // 50
```

---

## **What are DevTools Properties?**

DevTools properties are **read-only (or configurable) data properties** on the DevTools object that expose internal state for debugging and inspection.

**The 5 core properties:**
1. **`enabled`** - Boolean indicating if DevTools is active
2. **`states`** - Map storing tracked state objects with metadata
3. **`effects`** - Map storing tracked effect functions with metadata
4. **`history`** - Array of logged state changes
5. **`maxHistory`** - Maximum number of history entries to retain

---

## **Property Reference**

### **1. `DevTools.enabled`**

**Type**: `boolean`
**Default**: `false` (true on localhost)
**Writable**: Yes (via enable/disable methods)

Indicates whether DevTools is currently active and tracking changes.

```javascript
console.log(ReactiveUtils.DevTools.enabled); // false

ReactiveUtils.DevTools.enable();
console.log(ReactiveUtils.DevTools.enabled); // true

ReactiveUtils.DevTools.disable();
console.log(ReactiveUtils.DevTools.enabled); // false
```

**When enabled:**
- State changes are logged to history
- Tracking methods work normally
- Console access via `window.__REACTIVE_DEVTOOLS__`

**When disabled:**
- Tracking methods return early
- No changes logged
- No global window reference

---

### **2. `DevTools.states`**

**Type**: `Map<State, StateInfo>`
**Default**: Empty Map
**Writable**: No (internal use only)

Map storing all tracked state objects with their metadata.

**StateInfo structure:**
```javascript
{
  id: 1,                    // Unique numeric ID
  name: 'Counter',          // Custom or auto-generated name
  created: 1706198400000,   // Timestamp when tracked
  updates: []               // Array of changes for this state
}
```

**Usage:**
```javascript
ReactiveUtils.DevTools.enable();

const state = ReactiveUtils.state({ count: 0 });
ReactiveUtils.DevTools.trackState(state, 'Counter');

// Access states Map
console.log(ReactiveUtils.DevTools.states.size); // 1

// Get info for specific state
const info = ReactiveUtils.DevTools.states.get(state);
console.log(info.name); // 'Counter'
console.log(info.id); // 1
```

---

### **3. `DevTools.effects`**

**Type**: `Map<Function, EffectInfo>`
**Default**: Empty Map
**Writable**: No (internal use only)

Map storing all tracked effect functions with their metadata.

**EffectInfo structure:**
```javascript
{
  id: 1,                    // Unique numeric ID
  name: 'Logger',           // Custom or auto-generated name
  created: 1706198400000,   // Timestamp when tracked
  runs: 5                   // Number of times effect executed
}
```

**Usage:**
```javascript
ReactiveUtils.DevTools.enable();

const state = ReactiveUtils.state({ count: 0 });

const effect = ReactiveUtils.effect(() => {
  console.log(state.count);
});

ReactiveUtils.DevTools.trackEffect(effect, 'Logger');

// Access effects Map
console.log(ReactiveUtils.DevTools.effects.size); // 1

// Get info for specific effect
const info = ReactiveUtils.DevTools.effects.get(effect);
console.log(info.name); // 'Logger'
console.log(info.runs); // Number of executions
```

---

### **4. `DevTools.history`**

**Type**: `Array<ChangeRecord>`
**Default**: Empty array
**Writable**: Yes (but use clearHistory method)

Array storing logged state changes in chronological order.

**ChangeRecord structure:**
```javascript
{
  stateId: 1,                 // ID of the state that changed
  stateName: 'Counter',       // Name of the state
  key: 'count',               // Property that changed
  oldValue: 0,                // Previous value
  newValue: 1,                // New value
  timestamp: 1706198400000    // When change occurred
}
```

**Usage:**
```javascript
ReactiveUtils.DevTools.enable();

const state = ReactiveUtils.state({ count: 0 });
ReactiveUtils.DevTools.trackState(state, 'Counter');

state.count = 1;
state.count = 2;

// Access history
console.log(ReactiveUtils.DevTools.history.length); // 2
console.log(ReactiveUtils.DevTools.history[0].oldValue); // 0
console.log(ReactiveUtils.DevTools.history[0].newValue); // 1
console.log(ReactiveUtils.DevTools.history[1].oldValue); // 1
console.log(ReactiveUtils.DevTools.history[1].newValue); // 2
```

**Auto-pruning:**
When history exceeds `maxHistory`, oldest entries are removed automatically.

---

### **5. `DevTools.maxHistory`**

**Type**: `number`
**Default**: `50`
**Writable**: Yes (can be modified)

Maximum number of history entries to retain before auto-pruning.

**Usage:**
```javascript
ReactiveUtils.DevTools.enable();

// Check default
console.log(ReactiveUtils.DevTools.maxHistory); // 50

// Increase limit for detailed debugging
ReactiveUtils.DevTools.maxHistory = 200;

// Decrease for memory efficiency
ReactiveUtils.DevTools.maxHistory = 20;
```

**How auto-pruning works:**
```javascript
// When history grows beyond maxHistory
if (this.history.length > this.maxHistory) {
  this.history.shift(); // Remove oldest entry
}
```

---

## **Examples**

### **Example 1: Inspecting Enabled State**
```javascript
ReactiveUtils.DevTools.enable();

function isDebugging() {
  return ReactiveUtils.DevTools.enabled;
}

console.log(isDebugging()); // true

ReactiveUtils.DevTools.disable();
console.log(isDebugging()); // false
```

### **Example 2: Counting Tracked States**
```javascript
ReactiveUtils.DevTools.enable();

const state1 = ReactiveUtils.state({ a: 1 });
const state2 = ReactiveUtils.state({ b: 2 });
const state3 = ReactiveUtils.state({ c: 3 });

ReactiveUtils.DevTools.trackState(state1, 'State1');
ReactiveUtils.DevTools.trackState(state2, 'State2');
ReactiveUtils.DevTools.trackState(state3, 'State3');

console.log(`Tracking ${ReactiveUtils.DevTools.states.size} states`); // 3
```

### **Example 3: Iterating Over Tracked States**
```javascript
ReactiveUtils.DevTools.enable();

const user = ReactiveUtils.state({ name: 'Alice' });
const cart = ReactiveUtils.state({ items: [] });

ReactiveUtils.DevTools.trackState(user, 'User');
ReactiveUtils.DevTools.trackState(cart, 'Cart');

// Iterate over all tracked states
ReactiveUtils.DevTools.states.forEach((info, state) => {
  console.log(`${info.name} (ID: ${info.id}) - ${info.updates.length} updates`);
});
```

### **Example 4: Monitoring Effect Runs**
```javascript
ReactiveUtils.DevTools.enable();

const state = ReactiveUtils.state({ count: 0 });

const effect = ReactiveUtils.effect(() => {
  console.log('Count:', state.count);
});

ReactiveUtils.DevTools.trackEffect(effect, 'Counter Logger');

// Trigger effect multiple times
state.count = 1;
state.count = 2;
state.count = 3;

// Check run count
const info = ReactiveUtils.DevTools.effects.get(effect);
console.log(`Effect ran ${info.runs} times`); // 4 (initial + 3 updates)
```

### **Example 5: Analyzing History**
```javascript
ReactiveUtils.DevTools.enable();

const state = ReactiveUtils.state({ count: 0 });
ReactiveUtils.DevTools.trackState(state, 'Counter');

// Make changes
for (let i = 1; i <= 10; i++) {
  state.count = i;
}

// Analyze history
const history = ReactiveUtils.DevTools.history;

console.log(`Total changes: ${history.length}`); // 10

// Find largest change
const changes = history.map(h => h.newValue - h.oldValue);
const maxChange = Math.max(...changes);
console.log(`Largest increment: ${maxChange}`); // 1

// Count changes per property
const changesByKey = {};
history.forEach(h => {
  changesByKey[h.key] = (changesByKey[h.key] || 0) + 1;
});
console.log(changesByKey); // { count: 10 }
```

### **Example 6: Custom History Limit**
```javascript
ReactiveUtils.DevTools.enable();

// Set small history for memory efficiency
ReactiveUtils.DevTools.maxHistory = 5;

const state = ReactiveUtils.state({ value: 0 });
ReactiveUtils.DevTools.trackState(state, 'MyState');

// Make many changes
for (let i = 1; i <= 20; i++) {
  state.value = i;
}

// Only last 5 retained
console.log(ReactiveUtils.DevTools.history.length); // 5
console.log(ReactiveUtils.DevTools.history[0].newValue); // 16
console.log(ReactiveUtils.DevTools.history[4].newValue); // 20
```

### **Example 7: Per-State Updates**
```javascript
ReactiveUtils.DevTools.enable();

const state = ReactiveUtils.state({ count: 0, name: 'Test' });
ReactiveUtils.DevTools.trackState(state, 'MyState');

state.count = 1;
state.count = 2;
state.name = 'Updated';

// Get state info
const info = ReactiveUtils.DevTools.states.get(state);

// Per-state updates array
console.log(`${info.name} has ${info.updates.length} updates`); // 3

// Same as global history for this state
const stateChanges = ReactiveUtils.DevTools.history.filter(
  h => h.stateId === info.id
);
console.log(stateChanges.length); // 3
```

### **Example 8: State Creation Time**
```javascript
ReactiveUtils.DevTools.enable();

const state1 = ReactiveUtils.state({ a: 1 });
ReactiveUtils.DevTools.trackState(state1, 'First');

// Wait 1 second
setTimeout(() => {
  const state2 = ReactiveUtils.state({ b: 2 });
  ReactiveUtils.DevTools.trackState(state2, 'Second');

  // Compare creation times
  const info1 = ReactiveUtils.DevTools.states.get(state1);
  const info2 = ReactiveUtils.DevTools.states.get(state2);

  const diff = info2.created - info1.created;
  console.log(`States created ${diff}ms apart`); // ~1000ms
}, 1000);
```

### **Example 9: Finding State by Name**
```javascript
ReactiveUtils.DevTools.enable();

const user = ReactiveUtils.state({ name: 'Alice' });
const cart = ReactiveUtils.state({ items: [] });
const prefs = ReactiveUtils.state({ theme: 'dark' });

ReactiveUtils.DevTools.trackState(user, 'User');
ReactiveUtils.DevTools.trackState(cart, 'ShoppingCart');
ReactiveUtils.DevTools.trackState(prefs, 'Preferences');

function findStateByName(name) {
  for (const [state, info] of ReactiveUtils.DevTools.states) {
    if (info.name === name) {
      return { state, info };
    }
  }
  return null;
}

const result = findStateByName('ShoppingCart');
if (result) {
  console.log(`Found state: ${result.info.name} (ID: ${result.info.id})`);
}
```

### **Example 10: History Timeline Visualization**
```javascript
ReactiveUtils.DevTools.enable();

const state = ReactiveUtils.state({ status: 'idle' });
ReactiveUtils.DevTools.trackState(state, 'Status');

state.status = 'loading';
setTimeout(() => state.status = 'success', 100);
setTimeout(() => state.status = 'idle', 200);

setTimeout(() => {
  console.log('\n📊 Status Timeline:');
  console.log('─'.repeat(50));

  const startTime = ReactiveUtils.DevTools.history[0].timestamp;

  ReactiveUtils.DevTools.history.forEach(change => {
    const elapsed = change.timestamp - startTime;
    const arrow = `${change.oldValue} → ${change.newValue}`;
    console.log(`[+${elapsed}ms] ${arrow}`);
  });

  console.log('─'.repeat(50));
}, 300);
```

### **Example 11: Debugging Memory Usage**
```javascript
ReactiveUtils.DevTools.enable();

function getMemoryStats() {
  return {
    trackedStates: ReactiveUtils.DevTools.states.size,
    trackedEffects: ReactiveUtils.DevTools.effects.size,
    historyEntries: ReactiveUtils.DevTools.history.length,
    maxHistory: ReactiveUtils.DevTools.maxHistory,
    historyPercentage: (
      (ReactiveUtils.DevTools.history.length / ReactiveUtils.DevTools.maxHistory) * 100
    ).toFixed(1) + '%'
  };
}

// Create some states
for (let i = 0; i < 5; i++) {
  const state = ReactiveUtils.state({ id: i });
  ReactiveUtils.DevTools.trackState(state, `State${i}`);
}

console.log('Memory Stats:', getMemoryStats());
// {
//   trackedStates: 5,
//   trackedEffects: 0,
//   historyEntries: 0,
//   maxHistory: 50,
//   historyPercentage: '0.0%'
// }
```

### **Example 12: Auto-Enable Detection**
```javascript
// DevTools auto-enables on localhost
function checkAutoEnabled() {
  const isLocalhost =
    window.location.hostname === 'localhost' ||
    window.location.hostname === '127.0.0.1';

  const shouldBeEnabled = isLocalhost;
  const isEnabled = ReactiveUtils.DevTools.enabled;

  console.log(`Running on localhost: ${isLocalhost}`);
  console.log(`DevTools enabled: ${isEnabled}`);
  console.log(`Auto-enabled: ${shouldBeEnabled && isEnabled}`);
}

checkAutoEnabled();
```

### **Example 13: History Filtering**
```javascript
ReactiveUtils.DevTools.enable();

const state = ReactiveUtils.state({ count: 0, name: 'Test', active: true });
ReactiveUtils.DevTools.trackState(state, 'MyState');

state.count = 1;
state.count = 2;
state.name = 'Updated';
state.active = false;
state.count = 3;

// Filter history by property
function getChangesForKey(key) {
  return ReactiveUtils.DevTools.history.filter(h => h.key === key);
}

const countChanges = getChangesForKey('count');
console.log(`'count' changed ${countChanges.length} times`); // 3

const nameChanges = getChangesForKey('name');
console.log(`'name' changed ${nameChanges.length} times`); // 1
```

### **Example 14: Effect Tracking Stats**
```javascript
ReactiveUtils.DevTools.enable();

const state = ReactiveUtils.state({ count: 0 });

const effect1 = ReactiveUtils.effect(() => console.log('Effect 1:', state.count));
const effect2 = ReactiveUtils.effect(() => console.log('Effect 2:', state.count * 2));
const effect3 = ReactiveUtils.effect(() => console.log('Effect 3:', state.count + 10));

ReactiveUtils.DevTools.trackEffect(effect1, 'Logger');
ReactiveUtils.DevTools.trackEffect(effect2, 'Doubler');
ReactiveUtils.DevTools.trackEffect(effect3, 'Adder');

// Trigger effects
state.count = 5;
state.count = 10;

console.log(`\nTracking ${ReactiveUtils.DevTools.effects.size} effects:`);
ReactiveUtils.DevTools.effects.forEach((info, effect) => {
  console.log(`- ${info.name}: ${info.runs} runs`);
});
```

### **Example 15: Real-Time Monitoring**
```javascript
ReactiveUtils.DevTools.enable();

const state = ReactiveUtils.state({ temperature: 20 });
ReactiveUtils.DevTools.trackState(state, 'Sensor');

// Monitor in real-time
setInterval(() => {
  const stats = {
    enabled: ReactiveUtils.DevTools.enabled,
    states: ReactiveUtils.DevTools.states.size,
    effects: ReactiveUtils.DevTools.effects.size,
    changes: ReactiveUtils.DevTools.history.length,
    capacity: `${ReactiveUtils.DevTools.history.length}/${ReactiveUtils.DevTools.maxHistory}`
  };

  console.clear();
  console.log('📊 DevTools Monitor');
  console.log('─'.repeat(40));
  Object.entries(stats).forEach(([key, value]) => {
    console.log(`${key.padEnd(15)}: ${value}`);
  });
}, 1000);

// Simulate sensor updates
setInterval(() => {
  state.temperature = 20 + Math.random() * 10;
}, 500);
```

---

## **Common Patterns**

### **Pattern 1: Check If Enabled**
```javascript
if (ReactiveUtils.DevTools.enabled) {
  // DevTools-specific code
}
```

### **Pattern 2: Count Tracked Items**
```javascript
const trackedStates = ReactiveUtils.DevTools.states.size;
const trackedEffects = ReactiveUtils.DevTools.effects.size;
```

### **Pattern 3: Get Recent Changes**
```javascript
const recentChanges = ReactiveUtils.DevTools.history.slice(-10);
```

### **Pattern 4: Configure History Size**
```javascript
ReactiveUtils.DevTools.maxHistory = 100;
```

### **Pattern 5: Find State Info**
```javascript
const info = ReactiveUtils.DevTools.states.get(myState);
console.log(info.name, info.updates.length);
```

---

## **Property Summary Table**

| Property | Type | Default | Writable | Purpose |
|----------|------|---------|----------|---------|
| `enabled` | boolean | false | Via methods | Indicates if DevTools is active |
| `states` | Map | new Map() | No | Stores tracked state objects |
| `effects` | Map | new Map() | No | Stores tracked effect functions |
| `history` | Array | [] | Yes* | Logged state changes |
| `maxHistory` | number | 50 | Yes | Maximum history entries |

*Use `clearHistory()` method instead of direct modification

---

## **Data Structures**

### **StateInfo Object**
```javascript
{
  id: number,           // Unique ID (sequential)
  name: string,         // State name
  created: number,      // Timestamp
  updates: Array        // Per-state change history
}
```

### **EffectInfo Object**
```javascript
{
  id: number,           // Unique ID (sequential)
  name: string,         // Effect name
  created: number,      // Timestamp
  runs: number          // Execution count
}
```

### **ChangeRecord Object**
```javascript
{
  stateId: number,      // State ID
  stateName: string,    // State name
  key: string,          // Changed property
  oldValue: any,        // Previous value
  newValue: any,        // New value
  timestamp: number     // When change occurred
}
```

---

## **Console Access**

All properties accessible via global reference:

```javascript
// After enabling DevTools
window.__REACTIVE_DEVTOOLS__.enabled
window.__REACTIVE_DEVTOOLS__.states
window.__REACTIVE_DEVTOOLS__.effects
window.__REACTIVE_DEVTOOLS__.history
window.__REACTIVE_DEVTOOLS__.maxHistory
```

---

## **Key Takeaways**

1. **`enabled`** - Boolean flag controlled by enable/disable methods
2. **`states`** - Map with weak references to tracked states
3. **`effects`** - Map with weak references to tracked effects
4. **`history`** - Array of all changes, auto-pruned at maxHistory
5. **`maxHistory`** - Configurable limit (default 50)
6. **Read-Only** - Most properties shouldn't be modified directly
7. **Console Access** - All properties available via global reference
8. **Auto-Pruning** - History automatically limited to maxHistory
9. **Per-State Updates** - Each state tracks its own updates array
10. **Metadata Rich** - IDs, names, timestamps for debugging

---

## **Summary**

DevTools properties expose the internal state of the debugging system for inspection and analysis. The five core properties are: `enabled` (boolean flag), `states` (Map of tracked states), `effects` (Map of tracked effects), `history` (array of changes), and `maxHistory` (history size limit). These properties allow developers to inspect tracked states and effects, analyze change history, monitor system activity, and configure debugging behavior. The `states` and `effects` Maps use weak references and store rich metadata including IDs, names, creation timestamps, and activity counters. The `history` array logs all state changes chronologically with automatic pruning when it exceeds `maxHistory` (default 50). All properties are accessible via the console when DevTools is enabled through `window.__REACTIVE_DEVTOOLS__`, making them invaluable for debugging reactive applications during development.
