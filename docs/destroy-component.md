# `destroy(component)` - Destroy Component and Clean Up Resources

**Quick Start (30 seconds)**
```javascript
// Create a component
const counter = ReactiveUtils.reactive({ count: 0 })
  .effect(() => {
    console.log('Count:', counter.state.count);
  })
  .build();

// Use the component
counter.count = 5;

// Destroy the component and clean up
ReactiveUtils.destroy(counter);
// All effects are stopped, no more console logs

// Or use shorthand
destroy(counter);
```

---

## **What is `destroy(component)`?**

`destroy(component)` is a **utility function** that destroys a reactive component and cleans up all its resources by calling the component's internal `$destroy` method.

**Key characteristics:**
- **Cleanup**: Stops all effects, watchers, and bindings
- **Resource Management**: Prevents memory leaks
- **Safe**: Handles invalid components gracefully
- **Namespace Method**: Available as `ReactiveUtils.destroy()`
- **Shorthand**: Available as global `destroy()` function

---

## **Syntax**

```javascript
ReactiveUtils.destroy(component)

// Or with shorthand
destroy(component)
```

### **Parameters**
- **`component`** (Object): The reactive component to destroy

### **Returns**
- **Type**: `void`

---

## **How it works**

```javascript
ReactiveUtils.destroy = function(component) {
  if (!component || !component.$destroy) {
    console.error('[Namespace Methods] Invalid component or $destroy not available');
    return;
  }

  component.$destroy();
};
```

**What happens:**
1. Validates component has `$destroy` method
2. Calls `component.$destroy()` to run cleanup
3. All effects, watchers, and bindings are stopped
4. Component becomes inactive

---

## **Examples**

### **Example 1: Basic Component Destroy**
```javascript
const app = ReactiveUtils.reactive({ count: 0 })
  .effect(() => {
    console.log('Count changed:', app.state.count);
  })
  .build();

app.count = 1; // Logs: "Count changed: 1"
app.count = 2; // Logs: "Count changed: 2"

// Destroy the component
ReactiveUtils.destroy(app);

app.count = 3; // No log (effect stopped)
```

### **Example 2: Cleaning Up Event Listeners**
```javascript
const button = document.getElementById('btn');

const app = ReactiveUtils.reactive({ clicks: 0 })
  .effect(() => {
    const handler = () => app.state.clicks++;
    button.addEventListener('click', handler);

    return () => button.removeEventListener('click', handler);
  })
  .build();

// Later, when unmounting
ReactiveUtils.destroy(app);
// Event listener is removed
```

### **Example 3: Component Lifecycle**
```javascript
function createCounter() {
  const counter = ReactiveUtils.reactive({ count: 0 })
    .computed({
      double: function() { return this.count * 2; }
    })
    .effect(() => {
      document.getElementById('count').textContent = counter.state.count;
    })
    .build();

  return counter;
}

const counter = createCounter();

// Use counter
counter.count = 5;

// Unmount component
function unmount() {
  ReactiveUtils.destroy(counter);
  console.log('Counter destroyed');
}

unmount();
```

### **Example 4: Multiple Components**
```javascript
const components = [];

// Create multiple components
for (let i = 0; i < 5; i++) {
  const comp = ReactiveUtils.reactive({ id: i, value: 0 })
    .effect(() => {
      console.log(`Component ${comp.state.id}: ${comp.state.value}`);
    })
    .build();

  components.push(comp);
}

// Destroy all components
function destroyAll() {
  components.forEach(comp => ReactiveUtils.destroy(comp));
  console.log('All components destroyed');
}

destroyAll();
```

### **Example 5: Safe Destroy**
```javascript
function safeDestroy(component) {
  try {
    ReactiveUtils.destroy(component);
    console.log('Component destroyed successfully');
  } catch (error) {
    console.error('Failed to destroy component:', error);
  }
}

const app = ReactiveUtils.reactive({ data: 'test' }).build();

safeDestroy(app);
safeDestroy(null); // Handled safely
safeDestroy({}); // Handled safely (no $destroy method)
```

### **Example 6: Component Registry**
```javascript
class ComponentRegistry {
  constructor() {
    this.components = new Map();
  }

  register(id, component) {
    this.components.set(id, component);
    console.log(`Registered component: ${id}`);
  }

  destroy(id) {
    const component = this.components.get(id);

    if (component) {
      ReactiveUtils.destroy(component);
      this.components.delete(id);
      console.log(`Destroyed component: ${id}`);
    }
  }

  destroyAll() {
    this.components.forEach((component, id) => {
      ReactiveUtils.destroy(component);
      console.log(`Destroyed: ${id}`);
    });

    this.components.clear();
  }
}

const registry = new ComponentRegistry();

const app = ReactiveUtils.reactive({ name: 'App' }).build();
registry.register('app', app);

registry.destroy('app');
```

### **Example 7: Auto-Destroy on Timer**
```javascript
function createTemporaryComponent(lifetime = 5000) {
  const temp = ReactiveUtils.reactive({ active: true })
    .effect(() => {
      if (temp.state.active) {
        console.log('Component is active');
      }
    })
    .build();

  // Auto-destroy after lifetime
  setTimeout(() => {
    console.log('Auto-destroying component');
    ReactiveUtils.destroy(temp);
  }, lifetime);

  return temp;
}

const temp = createTemporaryComponent(3000);
// Component will be destroyed after 3 seconds
```

### **Example 8: Component with Cleanup Tracking**
```javascript
const cleanupLog = [];

const app = ReactiveUtils.reactive({ count: 0 })
  .effect(() => {
    console.log('Effect running');

    return () => {
      cleanupLog.push('Effect cleaned up');
    };
  })
  .watch({
    count: (newVal, oldVal) => {
      console.log(`Count: ${oldVal} -> ${newVal}`);
    }
  })
  .build();

app.count = 5;

ReactiveUtils.destroy(app);

console.log('Cleanup log:', cleanupLog);
// ['Effect cleaned up']
```

### **Example 9: Conditional Destroy**
```javascript
const components = {
  header: ReactiveUtils.reactive({ title: 'Header' }).build(),
  sidebar: ReactiveUtils.reactive({ collapsed: false }).build(),
  footer: ReactiveUtils.reactive({ text: 'Footer' }).build()
};

function destroyIf(condition) {
  Object.entries(components).forEach(([name, component]) => {
    if (condition(name, component)) {
      console.log(`Destroying ${name}`);
      ReactiveUtils.destroy(component);
    }
  });
}

// Destroy all except header
destroyIf((name) => name !== 'header');
```

### **Example 10: Router with Component Lifecycle**
```javascript
class SimpleRouter {
  constructor() {
    this.currentComponent = null;
  }

  navigate(componentFactory) {
    // Destroy current component
    if (this.currentComponent) {
      console.log('Destroying previous component');
      ReactiveUtils.destroy(this.currentComponent);
    }

    // Create new component
    this.currentComponent = componentFactory();
    console.log('New component created');
  }
}

const router = new SimpleRouter();

function HomePage() {
  return ReactiveUtils.reactive({ page: 'home' })
    .effect(() => console.log('Home page effect'))
    .build();
}

function AboutPage() {
  return ReactiveUtils.reactive({ page: 'about' })
    .effect(() => console.log('About page effect'))
    .build();
}

router.navigate(HomePage);
router.navigate(AboutPage); // Destroys HomePage first
```

---

## **Common Patterns**

### **Pattern 1: Simple Destroy**
```javascript
ReactiveUtils.destroy(component);
```

### **Pattern 2: Destroy with Shorthand**
```javascript
destroy(component);
```

### **Pattern 3: Destroy Multiple**
```javascript
components.forEach(comp => ReactiveUtils.destroy(comp));
```

### **Pattern 4: Destroy on Unmount**
```javascript
function unmount() {
  ReactiveUtils.destroy(component);
}
```

### **Pattern 5: Safe Destroy**
```javascript
if (component && component.$destroy) {
  ReactiveUtils.destroy(component);
}
```

---

## **When to Use**

| Scenario | Use destroy() |
|----------|---------------|
| Component unmount | ✓ Yes |
| Page navigation | ✓ Yes |
| Cleanup before re-create | ✓ Yes |
| Memory leak prevention | ✓ Yes |
| Stop all effects | ✓ Yes |
| Application shutdown | ✓ Yes |

---

## **What Gets Destroyed**

- ✓ All effects registered via `effect()`
- ✓ All watchers registered via `watch()`
- ✓ All bindings registered via `bind()`
- ✓ Custom cleanup functions
- ✗ The state object itself (still accessible, but inactive)

---

## **Error Handling**

```javascript
// Invalid component (null)
ReactiveUtils.destroy(null);
// Error: Invalid component or $destroy not available

// Invalid component (no $destroy method)
ReactiveUtils.destroy({});
// Error: Invalid component or $destroy not available

// Valid component
ReactiveUtils.destroy(validComponent);
// Success (no error)
```

---

## **vs. `builder.destroy()`**

| Feature | `destroy(component)` | `builder.destroy()` |
|---------|---------------------|---------------------|
| When | After `build()` | Before `build()` |
| Target | Built component | Builder itself |
| Usage | Namespace method | Builder method |
| Typical | Production use | During setup |

```javascript
// builder.destroy() - during setup
const builder = ReactiveUtils.reactive({ count: 0 })
  .effect(() => console.log('Effect'));

builder.destroy(); // Cleanup before build

// destroy(component) - after build
const component = ReactiveUtils.reactive({ count: 0 })
  .effect(() => console.log('Effect'))
  .build();

ReactiveUtils.destroy(component); // Cleanup after build
```

---

## **Best Practices**

1. **Always destroy components when unmounting**
   ```javascript
   function unmount() {
     ReactiveUtils.destroy(component);
   }
   ```

2. **Check before destroy in uncertain scenarios**
   ```javascript
   if (component && component.$destroy) {
     ReactiveUtils.destroy(component);
   }
   ```

3. **Clear references after destroying**
   ```javascript
   ReactiveUtils.destroy(component);
   component = null;
   ```

4. **Destroy in reverse order of creation**
   ```javascript
   components.reverse().forEach(c => ReactiveUtils.destroy(c));
   ```

5. **Use in cleanup functions**
   ```javascript
   onUnmount(() => {
     ReactiveUtils.destroy(component);
   });
   ```

---

## **Key Takeaways**

1. **Cleanup**: Destroys component and cleans up all resources
2. **Prevents Leaks**: Essential for memory leak prevention
3. **Stops Effects**: All effects and watchers are stopped
4. **Safe**: Handles invalid inputs gracefully
5. **Lifecycle**: Part of component lifecycle management
6. **Namespace**: Available as `ReactiveUtils.destroy()`
7. **Shorthand**: Available as global `destroy()`
8. **Required**: Must have `$destroy` method on component
9. **One-Way**: Cannot undo destroy operation
10. **Best Practice**: Always destroy on unmount

---

## **Summary**

`destroy(component)` is a namespace method that destroys a reactive component and cleans up all its resources by calling the component's internal `$destroy` method. It stops all effects, watchers, and bindings registered on the component, preventing memory leaks and ensuring proper cleanup. The method safely handles invalid components by checking for the `$destroy` method before calling it. Use `destroy()` whenever unmounting components, navigating between pages, or shutting down parts of your application. It's available both as `ReactiveUtils.destroy()` and as a global `destroy()` shorthand function. Always destroy components when they're no longer needed to prevent memory leaks and resource exhaustion. The destroy operation is one-way and cannot be undone - once destroyed, the component's effects will no longer run, though the state object remains accessible but inactive.
