# `DevTools.trackEffect()` - Track Effect Function

**Quick Start (30 seconds)**
```javascript
ReactiveUtils.DevTools.enable();

const state = ReactiveUtils.state({ count: 0 });

// Create effect
const dispose = effect(() => {
  console.log('Count:', state.count);
});

// Track it
ReactiveUtils.DevTools.trackEffect(dispose, 'CountLogger');

// Effect metadata is now tracked
const effects = ReactiveUtils.DevTools.effects;
console.log(effects.size); // 1
```

---

## **What is `DevTools.trackEffect()`?**

`DevTools.trackEffect()` is a **method** that registers an effect function for tracking, storing metadata about when it was created and how many times it has run.

**Key characteristics:**
- **Registers Effect**: Adds effect to tracking system
- **Metadata**: Stores creation time and run count
- **Named**: Assign readable names to effects
- **Unique ID**: Each effect gets unique ID
- **Debug Aid**: Helps understand effect behavior
- **No-Op When Disabled**: Does nothing if DevTools disabled

---

## **Syntax**

```javascript
ReactiveUtils.DevTools.trackEffect(effect, name?)
```

### **Parameters**
- `effect` (required) - Effect dispose function
  - **Type**: `Function` (returned from `effect()`)
- `name` (optional) - Human-readable name
  - **Type**: `string`
  - **Default**: `'Effect {id}'`

### **Returns**
- **Type**: `void`

---

## **How it works**

```javascript
trackEffect(effect, name) {
  if (!this.enabled) return;
  
  const id = this.effects.size + 1;
  this.effects.set(effect, {
    id,
    name: name || `Effect ${id}`,
    created: Date.now(),
    runs: 0
  });
}
```

---

## **Examples**

### **Example 1: Basic Tracking**
```javascript
ReactiveUtils.DevTools.enable();

const state = ReactiveUtils.state({ count: 0 });

const dispose = effect(() => {
  console.log('Count:', state.count);
});

ReactiveUtils.DevTools.trackEffect(dispose, 'CountLogger');
```

### **Example 2: Multiple Effects**
```javascript
ReactiveUtils.DevTools.enable();

const state = ReactiveUtils.state({ count: 0 });

const dispose1 = effect(() => {
  console.log('Effect 1:', state.count);
});

const dispose2 = effect(() => {
  console.log('Effect 2:', state.count * 2);
});

const dispose3 = effect(() => {
  document.title = `Count: ${state.count}`;
});

ReactiveUtils.DevTools.trackEffect(dispose1, 'Logger');
ReactiveUtils.DevTools.trackEffect(dispose2, 'Doubler');
ReactiveUtils.DevTools.trackEffect(dispose3, 'TitleUpdater');
```

### **Example 3: Without Name**
```javascript
ReactiveUtils.DevTools.enable();

const dispose = effect(() => {
  // effect code
});

ReactiveUtils.DevTools.trackEffect(dispose); // Name: "Effect 1"
```

### **Example 4: Component Effects**
```javascript
class Component {
  constructor(name) {
    this.effects = [];
    this.name = name;
  }
  
  addEffect(fn, effectName) {
    const dispose = effect(fn);
    
    if (ReactiveUtils.DevTools.enabled) {
      ReactiveUtils.DevTools.trackEffect(
        dispose, 
        `${this.name}:${effectName}`
      );
    }
    
    this.effects.push(dispose);
    return dispose;
  }
}

const header = new Component('Header');
header.addEffect(() => { /* ... */ }, 'DataLoader');
header.addEffect(() => { /* ... */ }, 'UIUpdater');
```

### **Example 5: Debug Effect Performance**
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

// Make changes
state.data = [1, 2, 3];
state.data = [4, 5, 6];

console.log(`Effect ran ${runCount} times`);
```

### **Example 6: Conditional Tracking**
```javascript
function createTrackedEffect(fn, name, track = false) {
  const dispose = effect(fn);
  
  if (track && ReactiveUtils.DevTools.enabled) {
    ReactiveUtils.DevTools.trackEffect(dispose, name);
  }
  
  return dispose;
}

const tracked = createTrackedEffect(() => {}, 'Tracked', true);
const untracked = createTrackedEffect(() => {}, 'Untracked', false);
```

### **Example 7: Effect Factory**
```javascript
class EffectFactory {
  static create(fn, name) {
    const dispose = effect(fn);
    
    if (ReactiveUtils.DevTools.enabled) {
      ReactiveUtils.DevTools.trackEffect(dispose, name);
    }
    
    return dispose;
  }
}

const dispose1 = EffectFactory.create(() => {}, 'Effect1');
const dispose2 = EffectFactory.create(() => {}, 'Effect2');
```

### **Example 8: View Effect Metadata**
```javascript
ReactiveUtils.DevTools.enable();

const dispose = effect(() => {
  // effect code
});

ReactiveUtils.DevTools.trackEffect(dispose, 'MyEffect');

// View metadata
const effectData = ReactiveUtils.DevTools.effects.get(dispose);
console.log(effectData);
// {
//   id: 1,
//   name: 'MyEffect',
//   created: 1234567890,
//   runs: 0
// }
```

### **Example 9: Debug Specific Effects**
```javascript
ReactiveUtils.DevTools.enable();

const state = ReactiveUtils.state({ value: 0 });

const dispose1 = effect(() => console.log('Effect 1:', state.value));
const dispose2 = effect(() => console.log('Effect 2:', state.value));
const dispose3 = effect(() => console.log('Effect 3:', state.value));

// Only track effect2 for debugging
ReactiveUtils.DevTools.trackEffect(dispose2, 'DebugEffect');
```

### **Example 10: Cleanup Tracking**
```javascript
ReactiveUtils.DevTools.enable();

const state = ReactiveUtils.state({ count: 0 });
const trackedEffects = new Map();

function createEffect(fn, name) {
  const dispose = effect(fn);
  
  ReactiveUtils.DevTools.trackEffect(dispose, name);
  trackedEffects.set(name, dispose);
  
  return dispose;
}

function cleanupEffect(name) {
  const dispose = trackedEffects.get(name);
  if (dispose) {
    dispose(); // Dispose effect
    trackedEffects.delete(name);
  }
}

createEffect(() => {}, 'Effect1');
createEffect(() => {}, 'Effect2');

cleanupEffect('Effect1'); // Clean up specific effect
```

---

## **Common Patterns**

### **Pattern 1: Track All Effects**
```javascript
const dispose = effect(fn);
ReactiveUtils.DevTools.trackEffect(dispose, 'EffectName');
```

### **Pattern 2: Conditional Tracking**
```javascript
if (ReactiveUtils.DevTools.enabled) {
  ReactiveUtils.DevTools.trackEffect(dispose, 'EffectName');
}
```

### **Pattern 3: Effect Factory**
```javascript
function createEffect(fn, name) {
  const dispose = effect(fn);
  ReactiveUtils.DevTools.trackEffect(dispose, name);
  return dispose;
}
```

---

## **Effect Entry Format**

Each tracked effect creates an entry:

```javascript
{
  id: 1,                   // Unique ID
  name: 'CountLogger',     // Effect name
  created: 1234567890,     // Creation timestamp
  runs: 0                  // Number of times run
}
```

---

## **Key Takeaways**

1. **Registers Effect**: Adds to tracking system
2. **Metadata**: Stores creation time and run count
3. **Named**: Provide readable names for debugging
4. **Unique IDs**: Each effect gets unique identifier
5. **Optional Name**: Auto-generates if not provided
6. **No-Op When Disabled**: Safe to call when disabled
7. **Debug Aid**: Helps understand effect behavior

---

## **Summary**

`DevTools.trackEffect()` registers an effect function for tracking, assigning it a unique ID and optional name. It stores metadata about when the effect was created and how many times it has run, helping you understand effect behavior during development. Use it to debug specific effects, monitor effect execution, and identify performance issues. The method is a no-op when DevTools is disabled, making it safe to leave in code during production builds.
