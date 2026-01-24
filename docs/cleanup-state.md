# `cleanup()` - Clean Up State Effects and Watchers

**Quick Start (30 seconds)**
```javascript
const state = ReactiveUtils.state({ count: 0 });

// Create some effects and watchers
const dispose1 = effect(() => {
  console.log('Count:', state.count);
});

const dispose2 = state.$watch('count', (newVal, oldVal) => {
  console.log('Changed:', oldVal, '→', newVal);
});

state.$computed('double', function() {
  return this.count * 2;
});

// Clean up all effects, watchers, and computed properties
cleanup(state);
// Or: state.$cleanup();

// Effects and watchers no longer run
state.count++; // Nothing happens
```

---

## **What is `cleanup()`?**

`cleanup()` is a **function** that removes all effects, watchers, and computed properties associated with a reactive state object, preventing memory leaks and stopping all reactive tracking.

**Key characteristics:**
- **Complete Cleanup**: Removes all reactive tracking
- **Effects Disposed**: All effects stop running
- **Watchers Removed**: All watchers are unregistered
- **Computed Cleared**: Computed properties are deleted
- **Memory Leak Prevention**: Breaks circular references
- **Two APIs**: `cleanup(state)` or `state.$cleanup()`

---

## **Syntax**

```javascript
// Global function
cleanup(state)

// Instance method
state.$cleanup()
```

### **Parameters**
- `state` (required) - The reactive state object
  - **Type**: `Object` (created with `state()`)

### **Returns**
- **Type**: `void`

---

## **How it works**

```javascript
state.$cleanup = function() {
  // 1. Clean up all computed properties
  if (this.__computedCleanups) {
    this.__computedCleanups.forEach(cleanup => cleanup());
    this.__computedCleanups.clear();
  }
  
  // 2. Remove all effects tracking this state
  const stateData = stateRegistry.get(this);
  if (stateData) {
    stateData.effects.forEach((effectSet) => {
      effectSet.forEach(effect => {
        unregisterEffect(effect);
      });
      effectSet.clear();
    });
    stateData.effects.clear();
  }
}
```

**What gets cleaned up:**
1. All computed properties are deleted
2. All effects tracking this state are disposed
3. All watchers on this state are removed
4. Internal registry entries are cleared

---

## **Examples**

### **Example 1: Basic Cleanup**
```javascript
const state = ReactiveUtils.state({ count: 0 });

effect(() => {
  console.log('Count:', state.count);
});

state.count++; // Logs: "Count: 1"

cleanup(state);

state.count++; // Nothing happens (effect disposed)
```

### **Example 2: Component Cleanup**
```javascript
class Component {
  constructor() {
    this.state = ReactiveUtils.state({ data: null });
    
    effect(() => {
      if (this.state.data) {
        this.render();
      }
    });
  }
  
  render() {
    console.log('Rendering:', this.state.data);
  }
  
  destroy() {
    cleanup(this.state); // Clean up all effects
  }
}

const component = new Component();
component.state.data = 'Hello'; // Logs: "Rendering: Hello"

component.destroy();

component.state.data = 'World'; // Nothing happens
```

### **Example 3: Multiple Effects**
```javascript
const state = ReactiveUtils.state({ count: 0, name: '' });

// Effect 1
effect(() => {
  console.log('Count:', state.count);
});

// Effect 2
effect(() => {
  console.log('Name:', state.name);
});

// Effect 3
effect(() => {
  document.title = `${state.name}: ${state.count}`;
});

// Clean up all effects at once
cleanup(state);

state.count++; // No effects run
state.name = 'Test'; // No effects run
```

### **Example 4: Watchers Cleanup**
```javascript
const state = ReactiveUtils.state({ value: 0 });

state.$watch('value', (newVal, oldVal) => {
  console.log('Value changed:', oldVal, '→', newVal);
});

state.value = 10; // Logs: "Value changed: 0 → 10"

cleanup(state);

state.value = 20; // Nothing happens (watcher removed)
```

### **Example 5: Computed Properties Cleanup**
```javascript
const state = ReactiveUtils.state({ count: 0 });

state.$computed('double', function() {
  return this.count * 2;
});

state.$computed('triple', function() {
  return this.count * 3;
});

console.log(state.double); // 0
console.log(state.triple); // 0

cleanup(state);

console.log(state.double); // undefined (computed deleted)
console.log(state.triple); // undefined (computed deleted)
```

### **Example 6: Form Cleanup**
```javascript
const form = Forms.createForm({ email: '', password: '' });

effect(() => {
  console.log('Form values:', form.values);
});

form.setValue('email', 'test@example.com');
// Logs: "Form values: { email: 'test@example.com', password: '' }"

cleanup(form);

form.setValue('password', 'secret');
// Nothing happens (effect disposed)
```

### **Example 7: Navigation Cleanup**
```javascript
const currentRoute = ReactiveUtils.state({ path: '/' });

function setupRoute() {
  effect(() => {
    console.log('Route:', currentRoute.path);
    loadRouteData(currentRoute.path);
  });
}

setupRoute();

// Navigate
currentRoute.path = '/about'; // Loads /about data

// Clean up before unmounting
cleanup(currentRoute);

// Navigate after cleanup
currentRoute.path = '/contact'; // Nothing happens
```

### **Example 8: Multiple State Objects**
```javascript
const state1 = ReactiveUtils.state({ a: 1 });
const state2 = ReactiveUtils.state({ b: 2 });

effect(() => {
  console.log('State 1:', state1.a);
});

effect(() => {
  console.log('State 2:', state2.b);
});

// Clean up only state1
cleanup(state1);

state1.a++; // Nothing happens
state2.b++; // Logs: "State 2: 3" (still works)
```

### **Example 9: Prevent Memory Leaks**
```javascript
function createManyComponents() {
  const components = [];
  
  for (let i = 0; i < 1000; i++) {
    const state = ReactiveUtils.state({ id: i });
    
    effect(() => {
      console.log('Component', state.id);
    });
    
    components.push(state);
  }
  
  return components;
}

const components = createManyComponents();

// Without cleanup: 1000 effects stay in memory!
// With cleanup: effects are properly disposed
components.forEach(state => cleanup(state));
```

### **Example 10: Lifecycle Management**
```javascript
class ViewModel {
  constructor(initialData) {
    this.state = ReactiveUtils.state(initialData);
    this.disposed = false;
  }
  
  setupEffects() {
    effect(() => {
      if (this.state.data) {
        this.syncToServer();
      }
    });
    
    effect(() => {
      this.updateUI();
    });
  }
  
  dispose() {
    if (this.disposed) return;
    
    cleanup(this.state);
    this.disposed = true;
    
    console.log('ViewModel disposed');
  }
  
  syncToServer() {
    // Sync logic
  }
  
  updateUI() {
    // UI update logic
  }
}

const vm = new ViewModel({ data: null });
vm.setupEffects();

// ... use viewmodel

vm.dispose(); // Clean up all effects
```

---

## **Common Patterns**

### **Pattern 1: Component Destroy**
```javascript
class Component {
  destroy() {
    cleanup(this.state);
  }
}
```

### **Pattern 2: Route Cleanup**
```javascript
function unmountRoute(routeState) {
  cleanup(routeState);
}
```

### **Pattern 3: Form Cleanup**
```javascript
function closeForm(form) {
  cleanup(form);
}
```

### **Pattern 4: Test Cleanup**
```javascript
afterEach(() => {
  cleanup(testState);
});
```

---

## **When to Use**

Use `cleanup()` when:

- **Component Unmount**: Destroy component and clean up effects
- **Route Change**: Clean up effects from previous route
- **Form Close**: Dispose form effects and watchers
- **Memory Management**: Prevent leaks from abandoned states
- **Test Teardown**: Clean up state between tests
- **Dynamic Content**: Remove effects from removed content

---

## **What Gets Cleaned**

| Type | What Happens |
|------|-------------|
| **Effects** | Disposed and unregistered |
| **Watchers** | Removed and unregistered |
| **Computed Properties** | Deleted from state object |
| **Registry Entries** | Cleared from tracking |

---

## **Memory Impact**

```javascript
// Without cleanup
const state = ReactiveUtils.state({ count: 0 });
for (let i = 0; i < 1000; i++) {
  effect(() => console.log(state.count));
}
// 1000 effects stay in memory forever!

// With cleanup
const state = ReactiveUtils.state({ count: 0 });
for (let i = 0; i < 1000; i++) {
  effect(() => console.log(state.count));
}
cleanup(state);
// All effects properly disposed ✓
```

---

## **Key Takeaways**

1. **Complete Cleanup**: Removes all reactive tracking
2. **Prevents Leaks**: Essential for memory management
3. **Two APIs**: `cleanup(state)` or `state.$cleanup()`
4. **Disposes Effects**: Stops all effects from running
5. **Removes Watchers**: Unregisters all watch callbacks
6. **Deletes Computed**: Removes computed properties
7. **Common Use**: Component unmount, route changes, tests
8. **One-Time Operation**: Cannot re-enable after cleanup

---

## **Summary**

`cleanup()` removes all effects, watchers, and computed properties associated with a reactive state object, providing complete cleanup of reactive tracking. It's essential for preventing memory leaks in long-lived applications, especially when components are mounted and unmounted, routes change, or forms are opened and closed. Use it in component destroy methods, route cleanup handlers, test teardown, and anywhere you need to fully dispose of a reactive state object's tracking infrastructure. Available as both `cleanup(state)` and `state.$cleanup()`.
