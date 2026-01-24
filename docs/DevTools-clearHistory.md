# `DevTools.clearHistory()` - Clear State Change History

**Quick Start (30 seconds)**
```javascript
ReactiveUtils.DevTools.enable();

// Create and track state
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

---

## **What is `DevTools.clearHistory()`?**

`DevTools.clearHistory()` is a **method** that clears all logged state changes from DevTools history, including both the global history and per-state update arrays.

**Key characteristics:**
- **Clears All History**: Removes all logged changes
- **Per-State Updates**: Also clears each state's updates array
- **Immediate Effect**: History cleared instantly
- **No Return Value**: Returns `undefined`
- **Non-Reversible**: Cannot undo clear operation
- **Safe to Call**: Idempotent, safe when disabled

---

## **Syntax**

```javascript
ReactiveUtils.DevTools.clearHistory()
```

### **Parameters**
- None

### **Returns**
- **Type**: `void`

---

## **How it works**

```javascript
clearHistory() {
  this.history.length = 0;
  this.states.forEach(info => {
    info.updates.length = 0;
  });
}
```

**What happens:**
1. Clears main history array
2. Iterates through all tracked states
3. Clears each state's updates array
4. All change history removed

---

## **Examples**

### **Example 1: Basic Clear**
```javascript
ReactiveUtils.DevTools.enable();

const state = ReactiveUtils.state({ count: 0 });
ReactiveUtils.DevTools.trackState(state, 'Counter');

// Make changes
state.count = 1;
state.count = 2;
state.count = 3;

console.log(ReactiveUtils.DevTools.getHistory().length); // 3

// Clear history
ReactiveUtils.DevTools.clearHistory();

console.log(ReactiveUtils.DevTools.getHistory().length); // 0
```

### **Example 2: Clear After Debug Session**
```javascript
ReactiveUtils.DevTools.enable();

const state = ReactiveUtils.state({ value: 0 });
ReactiveUtils.DevTools.trackState(state, 'MyState');

// Debug session
for (let i = 0; i < 50; i++) {
  state.value = i;
}

// View history
console.log('Debug complete:', ReactiveUtils.DevTools.getHistory().length);

// Clear for next session
ReactiveUtils.DevTools.clearHistory();
console.log('Ready for next debug session');
```

### **Example 3: Clear Between Tests**
```javascript
ReactiveUtils.DevTools.enable();

function setupTest() {
  ReactiveUtils.DevTools.clearHistory();
}

function test1() {
  setupTest();

  const state = ReactiveUtils.state({ count: 0 });
  ReactiveUtils.DevTools.trackState(state, 'Test1State');

  state.count = 1;
  console.assert(ReactiveUtils.DevTools.getHistory().length === 1);
}

function test2() {
  setupTest();

  const state = ReactiveUtils.state({ count: 0 });
  ReactiveUtils.DevTools.trackState(state, 'Test2State');

  state.count = 2;
  console.assert(ReactiveUtils.DevTools.getHistory().length === 1);
}

test1();
test2();
```

### **Example 4: Memory Management**
```javascript
ReactiveUtils.DevTools.enable();

const state = ReactiveUtils.state({ data: [] });
ReactiveUtils.DevTools.trackState(state, 'DataState');

// Make many changes
for (let i = 0; i < 1000; i++) {
  state.data = new Array(100).fill(i);
}

// Free memory by clearing history
ReactiveUtils.DevTools.clearHistory();
console.log('History cleared, memory freed');
```

### **Example 5: Clear and Restart**
```javascript
ReactiveUtils.DevTools.enable();

function restartDebugSession() {
  ReactiveUtils.DevTools.clearHistory();
  console.log('🔄 Debug session restarted');
}

const state = ReactiveUtils.state({ count: 0 });
ReactiveUtils.DevTools.trackState(state, 'Counter');

// First session
state.count = 1;
state.count = 2;
console.log('History:', ReactiveUtils.DevTools.getHistory().length); // 2

// Restart
restartDebugSession();

// Second session
state.count = 3;
console.log('History:', ReactiveUtils.DevTools.getHistory().length); // 1
```

### **Example 6: Clear Per-State Updates**
```javascript
ReactiveUtils.DevTools.enable();

const state = ReactiveUtils.state({ count: 0 });
ReactiveUtils.DevTools.trackState(state, 'Counter');

// Make changes
state.count = 1;
state.count = 2;
state.count = 3;

// Check state-specific updates
let states = ReactiveUtils.DevTools.getStates();
console.log(states[0].updates.length); // 3

// Clear history
ReactiveUtils.DevTools.clearHistory();

// Per-state updates also cleared
states = ReactiveUtils.DevTools.getStates();
console.log(states[0].updates.length); // 0
```

### **Example 7: Periodic Clear**
```javascript
ReactiveUtils.DevTools.enable();

const state = ReactiveUtils.state({ value: 0 });
ReactiveUtils.DevTools.trackState(state, 'MyState');

// Clear history every 5 seconds
setInterval(() => {
  const historySize = ReactiveUtils.DevTools.getHistory().length;
  if (historySize > 0) {
    ReactiveUtils.DevTools.clearHistory();
    console.log(`Cleared ${historySize} history entries`);
  }
}, 5000);

// State continues to work normally
setInterval(() => {
  state.value++;
}, 100);
```

### **Example 8: Clear Before Disable**
```javascript
ReactiveUtils.DevTools.enable();

const state = ReactiveUtils.state({ count: 0 });
ReactiveUtils.DevTools.trackState(state, 'Counter');

for (let i = 0; i < 50; i++) {
  state.count = i;
}

// Clear before disabling
ReactiveUtils.DevTools.clearHistory();
ReactiveUtils.DevTools.disable();

console.log('DevTools disabled with clean state');
```

### **Example 9: Conditional Clear**
```javascript
ReactiveUtils.DevTools.enable();

function clearIfNeeded(threshold = 100) {
  const history = ReactiveUtils.DevTools.getHistory();

  if (history.length >= threshold) {
    ReactiveUtils.DevTools.clearHistory();
    console.log(`History cleared (was ${history.length} entries)`);
    return true;
  }

  return false;
}

const state = ReactiveUtils.state({ value: 0 });
ReactiveUtils.DevTools.trackState(state, 'MyState');

// Make many changes
for (let i = 0; i < 150; i++) {
  state.value = i;
  clearIfNeeded(100);
}
```

### **Example 10: Test Helper**
```javascript
ReactiveUtils.DevTools.enable();

class TestHelper {
  static beforeEach() {
    ReactiveUtils.DevTools.clearHistory();
  }

  static afterEach() {
    const history = ReactiveUtils.DevTools.getHistory();
    console.log(`Test completed with ${history.length} state changes`);
    ReactiveUtils.DevTools.clearHistory();
  }
}

describe('State Tests', () => {
  beforeEach(() => TestHelper.beforeEach());
  afterEach(() => TestHelper.afterEach());

  it('should update state', () => {
    const state = ReactiveUtils.state({ count: 0 });
    ReactiveUtils.DevTools.trackState(state, 'TestState');

    state.count = 1;

    const history = ReactiveUtils.DevTools.getHistory();
    console.assert(history.length === 1);
  });
});
```

---

## **Common Patterns**

### **Pattern 1: Simple Clear**
```javascript
ReactiveUtils.DevTools.clearHistory();
```

### **Pattern 2: Clear Before Disable**
```javascript
ReactiveUtils.DevTools.clearHistory();
ReactiveUtils.DevTools.disable();
```

### **Pattern 3: Clear Between Tests**
```javascript
beforeEach(() => {
  ReactiveUtils.DevTools.clearHistory();
});
```

---

## **What Gets Cleared**

| Location | Description | Cleared? |
|----------|-------------|----------|
| Main history array | Global change history | ✓ Yes |
| Per-state updates | Each state's update array | ✓ Yes |
| Tracked states | State tracking continues | ✗ No |
| Tracked effects | Effect tracking continues | ✗ No |

---

## **State After Clear**

```javascript
// Before clear
{
  history: [...50 entries...],
  states: Map([
    [state1, { id: 1, name: 'State1', updates: [...10 entries...] }],
    [state2, { id: 2, name: 'State2', updates: [...20 entries...] }]
  ])
}

// After clear
{
  history: [],
  states: Map([
    [state1, { id: 1, name: 'State1', updates: [] }],
    [state2, { id: 2, name: 'State2', updates: [] }]
  ])
}
```

---

## **Console Access**

Available via global reference:

```javascript
// After enabling DevTools
window.__REACTIVE_DEVTOOLS__.clearHistory()
```

---

## **Key Takeaways**

1. **Clears All History**: Removes all logged changes
2. **Per-State Updates**: Also clears each state's updates
3. **Non-Reversible**: Cannot undo clear operation
4. **Tracking Continues**: States remain tracked
5. **Immediate Effect**: History cleared instantly
6. **Memory Management**: Frees memory from old entries
7. **Safe to Call**: Works even when disabled

---

## **Summary**

`DevTools.clearHistory()` clears all logged state changes from DevTools history, including both the global history array and each tracked state's updates array. The method is useful for memory management, clearing history between debug sessions, resetting state for tests, and starting fresh debugging sessions. After clearing, state and effect tracking continues normally - only the change history is removed. The operation is immediate and non-reversible, but safe to call multiple times. Use it to manage memory in long-running applications or to keep debug output clean during development.
