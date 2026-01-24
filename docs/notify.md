# `notify()` - Manual Dependency Notifications

## Quick Start (30 seconds)

```javascript
// Create state
const app = state({
  count: 0,
  items: []
});

// Create effect that watches count
effect(() => {
  console.log('Count:', app.count);
});

// Modify state externally (bypassing proxy)
const raw = toRaw(app);
raw.count = 10;  // Effect won't run automatically

// Manually notify dependencies
notify(app, 'count');  // Now effect runs!
// Output: Count: 10
```

**That's it.** Manually trigger reactive updates for specific properties. Use when modifying state outside the reactive system.

---

## What is `notify()`?

`notify()` **manually triggers reactive updates for a specific property or all properties** of a reactive state object. It's used when you modify state in ways that bypass the normal reactivity tracking.

Think of it as **the manual refresh button** — when automatic reactivity can't detect changes, you tell the system "this property changed, update everything that depends on it."

**In practical terms:** Use `notify()` when working with raw state objects, external libraries, or making changes that don't go through the reactive proxy.

---

## Syntax

```javascript
// Notify specific property
notify(stateObject, 'propertyName');

// Notify all properties (no key specified)
notify(stateObject);

// Examples
const user = state({
  name: 'Alice',
  age: 30
});

// Notify that 'name' changed
notify(user, 'name');

// Notify all properties changed
notify(user);
```

**Parameters:**
- `stateObject` - Reactive state object whose dependencies need notification
- `key` (optional) - Property name to notify. If omitted, notifies all properties

**Returns:**
- `undefined` (this is a side-effect function)

**Important:**
- Only **non-computed** effects are triggered
- Notifications are **batched** for performance
- Use sparingly — normal assignments trigger reactivity automatically

---

## Why Does This Exist?

### The Problem Without notify()

When you bypass the reactive proxy, effects don't run:

```javascript
// ❌ Direct modification of raw object - effects won't run
const counter = state({ count: 0 });

effect(() => {
  console.log('Count:', counter.count);
});

// Get raw object (bypasses reactivity)
const raw = toRaw(counter);
raw.count = 10;  // No effect runs!

// Or using external library that modifies object directly
someLibrary.mutate(raw);  // Effects won't know about changes
```

**Problems:**
❌ **Bypassed reactivity** - Direct modifications don't trigger effects
❌ **Stale UI** - DOM doesn't update when it should
❌ **No notification** - Watchers and effects miss changes
❌ **Silent failures** - Hard to debug why effects aren't running

### The Solution with `notify()`

```javascript
// ✅ Manual notification after direct modification
const counter = state({ count: 0 });

effect(() => {
  console.log('Count:', counter.count);
});

// Modify raw object
const raw = toRaw(counter);
raw.count = 10;

// Manually notify
notify(counter, 'count');
// ✨ Effect runs! Output: Count: 10

// Or use with external library
someLibrary.mutate(raw);
notify(counter);  // Notify all properties
```

**Benefits:**
✅ **Manual control** - Trigger updates when needed
✅ **Integration** - Work with external libraries
✅ **Batching** - Notifications are batched automatically
✅ **Selective** - Notify specific properties or all
✅ **Debugging** - Explicit about when updates happen

---

## Mental Model: Manual Refresh

Think of `notify()` like **pressing the refresh button**:

**Normal Reactivity (Automatic):**
```
┌─────────────────────────────┐
│  Automatic Reactivity       │
│                             │
│  state.count = 10           │
│  ↓                          │
│  Proxy detects change       │
│  ↓                          │
│  Automatically notifies     │
│  ↓                          │
│  Effects run                │
│                             │
│  No manual action needed!   │
└─────────────────────────────┘
```

**With notify() (Manual):**
```
┌─────────────────────────────┐
│  Manual Notification        │
│                             │
│  raw.count = 10             │
│  ↓                          │
│  Bypasses proxy             │
│  ↓                          │
│  No automatic detection     │
│  ↓                          │
│  notify(state, 'count')     │
│  ↓                          │
│  Manually notifies          │
│  ↓                          │
│  Effects run                │
└─────────────────────────────┘
```

`notify()` is **the manual refresh** — you tell the system when to update.

---

## How Does It Work?

`notify()` triggers all effects watching a property:

```
Call notify(state, key)
    ↓
Get reactive metadata
    ↓
If key specified:
  ┌────────────────────┐
  │ Get effects for    │
  │ specific property  │
  └────────────────────┘
        ↓
    Queue all effects
        ↓
If no key:
  ┌────────────────────┐
  │ Get ALL effects    │
  │ for all properties │
  └────────────────────┘
        ↓
    Queue all effects
        ↓
Batch execution
    ↓
Run queued effects
    ↓
Complete
```

**Key behaviors:**
- Only triggers **non-computed** effects
- Effects are **queued** for batching
- Specific key notifies **that property only**
- No key notifies **all properties**
- Safe to call **multiple times** (batched)

---

## Basic Usage

### Example 1: Working with Raw Objects

```javascript
const user = state({
  name: 'Alice',
  age: 30
});

effect(() => {
  console.log(`${user.name} is ${user.age} years old`);
});

// Get raw object
const raw = toRaw(user);

// Modify directly
raw.age = 31;

// Manually notify
notify(user, 'age');
// Output: Alice is 31 years old
```

---

### Example 2: External Library Integration

```javascript
const data = state({
  items: [1, 2, 3],
  sorted: false
});

effect(() => {
  console.log('Items:', data.items);
});

// External library modifies array in-place
const raw = toRaw(data);
externalSortLibrary.sort(raw.items);  // Modifies array directly

// Notify that items changed
notify(data, 'items');
// Effect runs with sorted array
```

---

### Example 3: Batch Notifications

```javascript
const app = state({
  x: 0,
  y: 0,
  z: 0
});

effect(() => {
  console.log(`Position: ${app.x}, ${app.y}, ${app.z}`);
});

// Modify multiple properties
const raw = toRaw(app);
raw.x = 10;
raw.y = 20;
raw.z = 30;

// Notify all at once
notify(app);
// Output: Position: 10, 20, 30
```

---

### Example 4: Performance Optimization

```javascript
const canvas = state({
  pixels: new Uint8Array(1000000)
});

effect(() => {
  renderCanvas(canvas.pixels);
});

// Modify many pixels directly for performance
const raw = toRaw(canvas).pixels;
for (let i = 0; i < 1000000; i++) {
  raw[i] = Math.random() * 255;
}

// Single notification after all modifications
notify(canvas, 'pixels');
// Renders once, not a million times
```

---

### Example 5: Custom Property Setters

```javascript
const position = state({
  _x: 0,
  _y: 0
});

// Custom setter that modifies raw
function setPosition(x, y) {
  const raw = toRaw(position);
  raw._x = x;
  raw._y = y;

  // Notify both properties
  notify(position, '_x');
  notify(position, '_y');
}

effect(() => {
  console.log(`Position: ${position._x}, ${position._y}`);
});

setPosition(100, 200);
// Output: Position: 100, 200
```

---

### Example 6: Conditional Notifications

```javascript
const settings = state({
  theme: 'light',
  lastChanged: null
});

effect(() => {
  console.log('Theme:', settings.theme);
});

// Only notify if actually changed
function updateTheme(newTheme) {
  if (toRaw(settings).theme !== newTheme) {
    const raw = toRaw(settings);
    raw.theme = newTheme;
    raw.lastChanged = Date.now();

    notify(settings, 'theme');
    notify(settings, 'lastChanged');
  }
}
```

---

### Example 7: WebGL Buffer Updates

```javascript
const gl = state({
  vertexBuffer: new Float32Array(1000),
  needsUpdate: false
});

effect(() => {
  if (gl.needsUpdate) {
    uploadToGPU(gl.vertexBuffer);
  }
});

// Modify buffer directly
function updateVertices(vertices) {
  const raw = toRaw(gl);
  raw.vertexBuffer.set(vertices);
  raw.needsUpdate = true;

  // Notify both
  notify(gl, 'vertexBuffer');
  notify(gl, 'needsUpdate');
}
```

---

### Example 8: Legacy Code Integration

```javascript
const legacyState = state({
  data: { value: 0 }
});

effect(() => {
  console.log('Legacy value:', legacyState.data.value);
});

// Legacy function that modifies state directly
function legacyUpdate(newValue) {
  // Old code that works with raw objects
  const obj = toRaw(legacyState);
  obj.data.value = newValue;

  // Add notification to make it reactive
  notify(legacyState, 'data');
}

legacyUpdate(42);
// Output: Legacy value: 42
```

---

### Example 9: Array Mutations

```javascript
const list = state({
  items: [1, 2, 3, 4, 5]
});

effect(() => {
  console.log('Items:', list.items.length);
});

// Direct array mutation
function shuffleItems() {
  const raw = toRaw(list).items;

  // Fisher-Yates shuffle (modifies in-place)
  for (let i = raw.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [raw[i], raw[j]] = [raw[j], raw[i]];
  }

  notify(list, 'items');
}
```

---

### Example 10: Debugging and Logging

```javascript
const debug = state({
  logs: [],
  errors: []
});

effect(() => {
  console.log('Logs updated:', debug.logs.length);
});

// Add log without triggering effect
function addLogSilent(message) {
  toRaw(debug).logs.push(message);
  // No notification - silent
}

// Add log and trigger effect
function addLogVerbose(message) {
  toRaw(debug).logs.push(message);
  notify(debug, 'logs');  // Trigger effect
}
```

---

## Advanced Usage: Selective Notifications

```javascript
const complexState = state({
  a: 0,
  b: 0,
  c: 0,
  expensive: 0
});

// Expensive effect
effect(() => {
  console.log('Expensive computation:', complexState.expensive);
  // ... heavy calculation
});

// Cheap effect
effect(() => {
  console.log('Simple:', complexState.a);
});

// Modify multiple properties
const raw = toRaw(complexState);
raw.a = 1;
raw.b = 2;
raw.c = 3;

// Only notify specific properties
notify(complexState, 'a');  // Only triggers cheap effect
notify(complexState, 'b');
notify(complexState, 'c');

// Expensive effect doesn't run because we didn't notify 'expensive'
```

---

## Common Patterns

### Pattern 1: After Raw Modification

```javascript
const raw = toRaw(state);
raw.property = newValue;
notify(state, 'property');
```

### Pattern 2: Notify All Properties

```javascript
// Made many changes
const raw = toRaw(state);
raw.a = 1;
raw.b = 2;
raw.c = 3;
notify(state);  // Notify everything
```

### Pattern 3: Performance Critical Update

```javascript
const raw = toRaw(state);
for (let i = 0; i < 1000000; i++) {
  raw.array[i] = compute(i);
}
notify(state, 'array');  // Single notification
```

### Pattern 4: External Library

```javascript
externalLib.modify(toRaw(state));
notify(state);  // Notify after external changes
```

---

## When to Use notify()

**✅ Use notify() when:**
- Working with raw objects (`toRaw()`)
- Integrating external libraries that modify state
- Performance-critical code that bypasses reactivity
- Batch modifications that should notify once
- Legacy code that needs reactivity added

**❌ Don't use notify() when:**
- Normal property assignments work fine
- State changes go through the proxy
- You can use `set()` or direct assignment
- Effects are already running correctly

---

## Key Takeaways

✅ **Manual control** - Trigger updates explicitly
✅ **Raw object support** - Work with non-reactive objects
✅ **Selective** - Notify specific properties or all
✅ **Batched** - Notifications queued for performance
✅ **Integration** - Bridge to non-reactive code
✅ **Debugging** - Explicit about update timing

---

## What's Next?

- **`batch()`** - Manual batching control
- **`pause()`** - Pause reactivity temporarily
- **`untrack()`** - Run code without tracking dependencies

---

## Summary

`notify()` **manually triggers reactive updates for specific or all properties** of a reactive state object. Use it when modifying state outside the normal reactive system.

**The magic formula:**
```
toRaw(state).prop = value;
notify(state, 'prop');
  =
Manual modification + Manual notification
─────────────────────────────────────────
Bypass reactivity, then trigger it
```

Think of it as **the manual refresh button** — when automatic reactivity can't detect changes (raw objects, external libraries), you explicitly tell the system what changed. Essential for performance optimization and integrating non-reactive code.
