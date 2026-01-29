# `builder.build()` - Build and Return State

**Quick Start (30 seconds)**
```javascript
const builder = ReactiveUtils.reactive({ count: 0 })
  .computed({
    double: function() { return this.count * 2; }
  })
  .action('increment', function() {
    this.count++;
  });

// Build returns the final state object
const app = builder.build();

console.log(app.count); // 0
console.log(app.double); // 0
app.increment();
console.log(app.count); // 1
console.log(app.double); // 2

// State has destroy method
app.destroy(); // Clean up all effects
```

---

## **What is `builder.build()`?**

`builder.build()` is a **builder method** that finalizes the component construction and returns the reactive state object with a `destroy()` method for cleanup.

**Key characteristics:**
- **Finalization**: Completes the building process
- **Returns State**: Returns the reactive state object
- **Adds Destroy**: Attaches `destroy()` method to state
- **Non-Chainable**: Returns state, not builder
- **One-Time**: Typically called once at the end

---

## **Syntax**

```javascript
const state = builder.build()
```

### **Parameters**
- None

### **Returns**
- **Type**: Reactive state object (Proxy) with `destroy()` method

---

## **How it works**

```javascript
builder.build() {
  state.destroy = () => cleanups.forEach(c => c());
  return state;
}
```

**What happens:**
1. Adds `destroy()` method to state
2. `destroy()` calls all cleanup functions
3. Returns the state object
4. State has all computed, actions, etc.

---

## **Examples**

### **Example 1: Basic Build**
```javascript
const app = ReactiveUtils.reactive({ count: 0 })
  .action('increment', function() {
    this.count++;
  })
  .build();

console.log(typeof app.destroy); // 'function'
app.increment();
console.log(app.count); // 1
```

### **Example 2: Full Component**
```javascript
const counter = ReactiveUtils.reactive({ count: 0 })
  .computed({
    double: function() { return this.count * 2; },
    triple: function() { return this.count * 3; }
  })
  .watch({
    count: (newVal) => console.log('Count:', newVal)
  })
  .effect(() => {
    document.title = `Count: ${counter.state.count}`;
  })
  .actions({
    increment: function() { this.count++; },
    decrement: function() { this.count--; },
    reset: function() { this.count = 0; }
  })
  .build();

// Now use the component
counter.increment();
```

### **Example 3: Factory Pattern**
```javascript
function createCounter(initialCount = 0) {
  return ReactiveUtils.reactive({ count: initialCount })
    .computed({
      double: function() { return this.count * 2; }
    })
    .action('increment', function() {
      this.count++;
    })
    .build();
}

const counter1 = createCounter(0);
const counter2 = createCounter(10);
```

### **Example 4: Conditional Building**
```javascript
const builder = ReactiveUtils.reactive({ debug: false, value: 0 });

if (process.env.NODE_ENV === 'development') {
  builder.state.debug = true;
  builder.effect(() => {
    console.log('Debug:', builder.state.value);
  });
}

const app = builder.build();
```

### **Example 5: Store Multiple Instances**
```javascript
const components = [];

for (let i = 0; i < 3; i++) {
  const component = ReactiveUtils.reactive({ id: i, value: 0 })
    .action('increment', function() {
      this.value++;
    })
    .build();

  components.push(component);
}

// Use components
components[0].increment();
components[1].increment();

// Cleanup all
components.forEach(c => c.destroy());
```

### **Example 6: With Cleanup**
```javascript
const app = ReactiveUtils.reactive({ active: true })
  .effect(() => {
    if (!app.state.active) return;

    const interval = setInterval(() => {
      console.log('Tick');
    }, 1000);

    return () => clearInterval(interval);
  })
  .build();

// Later...
app.destroy(); // Clears interval
```

### **Example 7: Component Lifecycle**
```javascript
function createComponent(config) {
  const component = ReactiveUtils.reactive(config.state || {})
    .computed(config.computed || {})
    .actions(config.actions || {})
    .build();

  if (config.mounted) {
    config.mounted.call(component);
  }

  // Override destroy to call unmounted
  const originalDestroy = component.destroy;
  component.destroy = function() {
    if (config.unmounted) {
      config.unmounted.call(this);
    }
    originalDestroy.call(this);
  };

  return component;
}

const app = createComponent({
  state: { count: 0 },
  mounted() {
    console.log('Component mounted');
  },
  unmounted() {
    console.log('Component unmounted');
  }
});
```

### **Example 8: Return Value Usage**
```javascript
// Direct usage
const app = ReactiveUtils.reactive({ count: 0 })
  .action('increment', function() { this.count++; })
  .build();

// Or store builder and build later
const builder = ReactiveUtils.reactive({ count: 0 })
  .action('increment', function() { this.count++; });

// ... later
const app = builder.build();
```

### **Example 9: Builder vs Built State**
```javascript
const builder = ReactiveUtils.reactive({ count: 0 })
  .action('increment', function() { this.count++; });

// Before build - access via builder.state
console.log(builder.state.count); // 0

// After build - use returned state directly
const app = builder.build();
console.log(app.count); // 0

// Same object
console.log(app === builder.state); // true
```

### **Example 10: Multiple Build Calls**
```javascript
const builder = ReactiveUtils.reactive({ count: 0 });

const app1 = builder.build();
const app2 = builder.build();

// Both reference the same state
console.log(app1 === app2); // true
console.log(app1 === builder.state); // true

// All destroy methods are stacked
app1.destroy(); // Calls all cleanups
app2.destroy(); // Called again (may cause issues)
```

---

## **Common Patterns**

### **Pattern 1: Immediate Build**
```javascript
const app = ReactiveUtils.reactive({ count: 0 })
  .build();
```

### **Pattern 2: Chained Build**
```javascript
const app = ReactiveUtils.reactive({ count: 0 })
  .computed({ /* ... */ })
  .actions({ /* ... */ })
  .build();
```

### **Pattern 3: Factory**
```javascript
function create(config) {
  return ReactiveUtils.reactive(config)
    .computed({ /* ... */ })
    .build();
}
```

### **Pattern 4: Conditional**
```javascript
const builder = ReactiveUtils.reactive({ count: 0 });

if (condition) {
  builder.effect(/* ... */);
}

const app = builder.build();
```

---

## **Build vs No Build**

| Aspect | Before build() | After build() |
|--------|---------------|---------------|
| State Access | `builder.state.prop` | `app.prop` |
| Chainable | ✓ Yes | ✗ No |
| Has destroy | ✗ No* | ✓ Yes |
| Typical Use | Configuration | Production use |

*Builder has `destroy()` but it's for cleanup before build

---

## **Best Practices**

1. **Build at the end**
   ```javascript
   const app = ReactiveUtils.reactive({ count: 0 })
     .computed({ /* ... */ })
     .actions({ /* ... */ })
     .build(); // Last call
   ```

2. **Store the result**
   ```javascript
   const app = builder.build();
   // Use app, not builder
   ```

3. **Call destroy when done**
   ```javascript
   const app = builder.build();
   // ... use app ...
   app.destroy(); // Cleanup
   ```

4. **Don't call build multiple times**
   ```javascript
   // Avoid
   const app1 = builder.build();
   const app2 = builder.build(); // Same object, multiple destroy methods
   ```

5. **Factory pattern for reuse**
   ```javascript
   function createComponent() {
     return ReactiveUtils.reactive({ /* ... */ })
       .build();
   }
   ```

---

## **Key Takeaways**

1. **Finalization**: Completes building process
2. **Returns State**: Returns reactive state object
3. **Adds Destroy**: Attaches cleanup method
4. **Non-Chainable**: Returns state, not builder
5. **One-Time**: Call once at end of configuration
6. **Same Reference**: Returns `builder.state`
7. **Cleanup**: State has `destroy()` method
8. **Production**: Built state is what you use
9. **Factory**: Use in factories for reusability
10. **End of Chain**: Final method in builder chain

---

## **Summary**

`builder.build()` finalizes component construction and returns the reactive state object with an attached `destroy()` method for cleanup. The method completes the building process by adding a destroy function that executes all registered cleanup callbacks (from effects, watchers, bindings). The returned state object is the same as `builder.state`, containing all configured computed properties, actions, and reactive behavior. Call `build()` once at the end of your builder chain to get the final, production-ready component. The built state should be used directly rather than accessing through `builder.state`. Always call `destroy()` on the state when the component is no longer needed to prevent memory leaks.
