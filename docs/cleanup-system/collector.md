# `collector()` - Create Cleanup Collector

**Quick Start (30 seconds)**
```javascript
// Create a collector
const collector = ReactiveUtils.collector();

// Add cleanup functions
collector.add(() => console.log('Cleanup 1'));
collector.add(() => console.log('Cleanup 2'));
collector.add(() => console.log('Cleanup 3'));

console.log(collector.size); // 3
console.log(collector.disposed); // false

// Run all cleanups
collector.cleanup();
// Logs: "Cleanup 1", "Cleanup 2", "Cleanup 3"

console.log(collector.disposed); // true
console.log(collector.size); // 0
```

---

## **What is `collector()`?**

`collector()` is a **factory function** that creates a cleanup collector object for managing multiple cleanup functions. It provides a centralized way to register and execute cleanup logic when resources need to be released.

**Key characteristics:**
- **Factory Function**: Returns a collector object
- **Chainable**: Methods return `this` for chaining
- **Auto-Execute**: Runs all cleanups in order
- **Error Handling**: Catches and logs individual cleanup errors
- **Disposed State**: Tracks whether cleanup has been called

---

## **Syntax**

```javascript
const collector = ReactiveUtils.collector()
```

### **Parameters**
- None

### **Returns**
- **Type**: `Object` (collector instance)
- **Properties**:
  - `add(cleanup)` - Add cleanup function
  - `cleanup()` - Execute all cleanups
  - `size` - Number of cleanup functions (getter)
  - `disposed` - Whether cleanup has been called (getter)

---

## **How it works**

```javascript
function collector() {
  const cleanups = [];
  let isDisposed = false;
  
  return {
    add(cleanup) {
      if (isDisposed) {
        console.warn('Cannot add to disposed collector');
        return this;
      }
      
      if (typeof cleanup === 'function') {
        cleanups.push(cleanup);
      }
      return this;
    },
    
    cleanup() {
      if (isDisposed) return;
      
      isDisposed = true;
      cleanups.forEach(cleanup => {
        try {
          cleanup();
        } catch (error) {
          console.error('Collector error:', error);
        }
      });
      cleanups.length = 0;
    },
    
    get size() {
      return cleanups.length;
    },
    
    get disposed() {
      return isDisposed;
    }
  };
}
```

---

## **Examples**

### **Example 1: Basic Usage**
```javascript
const collector = ReactiveUtils.collector();

// Add cleanup functions
collector.add(() => {
  console.log('Cleaning up resource 1');
});

collector.add(() => {
  console.log('Cleaning up resource 2');
});

// Execute all cleanups
collector.cleanup();
// Logs: "Cleaning up resource 1", "Cleaning up resource 2"
```

### **Example 2: Component Lifecycle**
```javascript
class Component {
  constructor() {
    this.collector = ReactiveUtils.collector();
    
    // Add effect cleanup
    this.collector.add(
      effect(() => {
        console.log('Effect running');
      })
    );
    
    // Add event listener cleanup
    const handler = () => console.log('Click');
    document.addEventListener('click', handler);
    this.collector.add(() => {
      document.removeEventListener('click', handler);
    });
    
    // Add timer cleanup
    const timer = setInterval(() => console.log('Tick'), 1000);
    this.collector.add(() => clearInterval(timer));
  }
  
  destroy() {
    // Clean up all resources at once
    this.collector.cleanup();
  }
}

const component = new Component();
// ... later
component.destroy(); // Cleans up effects, listeners, timers
```

### **Example 3: Effect Cleanup Collection**
```javascript
const collector = ReactiveUtils.collector();
const state = ReactiveUtils.state({ count: 0 });

// Multiple effects
collector.add(
  effect(() => {
    console.log('Count:', state.count);
  })
);

collector.add(
  effect(() => {
    document.title = `Count: ${state.count}`;
  })
);

collector.add(
  effect(() => {
    localStorage.setItem('count', state.count);
  })
);

console.log(collector.size); // 3

// Clean up all effects
collector.cleanup();
```

### **Example 4: API Cleanup**
```javascript
const collector = ReactiveUtils.collector();

async function loadData() {
  // Create abort controller
  const abortController = new AbortController();
  
  // Add abort cleanup
  collector.add(() => {
    abortController.abort();
  });
  
  // Add response cleanup
  const response = await fetch('/api/data', {
    signal: abortController.signal
  });
  
  collector.add(async () => {
    // Close response if needed
    if (response.body) {
      await response.body.cancel();
    }
  });
  
  return response.json();
}

// Later: abort all pending requests
collector.cleanup();
```

### **Example 5: Event Listeners**
```javascript
const collector = ReactiveUtils.collector();

function setupListeners() {
  const button = querySelector('#btn');
  const input = querySelector('#input');
  
  const clickHandler = () => console.log('Click');
  const inputHandler = () => console.log('Input');
  
  button.addEventListener('click', clickHandler);
  input.addEventListener('input', inputHandler);
  
  // Add cleanup for both
  collector.add(() => {
    button.removeEventListener('click', clickHandler);
    input.removeEventListener('input', inputHandler);
  });
}

setupListeners();

// Clean up all listeners
collector.cleanup();
```

### **Example 6: Chainable API**
```javascript
const collector = ReactiveUtils.collector()
  .add(() => console.log('Cleanup 1'))
  .add(() => console.log('Cleanup 2'))
  .add(() => console.log('Cleanup 3'));

console.log(collector.size); // 3

collector.cleanup();
```

### **Example 7: Multiple Resource Types**
```javascript
const collector = ReactiveUtils.collector();

// WebSocket
const ws = new WebSocket('wss://example.com');
collector.add(() => ws.close());

// Animation frame
const frameId = requestAnimationFrame(animate);
collector.add(() => cancelAnimationFrame(frameId));

// Intersection Observer
const observer = new IntersectionObserver(callback);
collector.add(() => observer.disconnect());

// Media stream
navigator.mediaDevices.getUserMedia({ video: true })
  .then(stream => {
    collector.add(() => {
      stream.getTracks().forEach(track => track.stop());
    });
  });

// Clean up everything
function onUnmount() {
  collector.cleanup();
}
```

### **Example 8: Error Handling**
```javascript
const collector = ReactiveUtils.collector();

collector.add(() => {
  console.log('Good cleanup');
});

collector.add(() => {
  throw new Error('Cleanup failed!');
  // Error is caught and logged, doesn't stop other cleanups
});

collector.add(() => {
  console.log('This still runs');
});

collector.cleanup();
// Logs: "Good cleanup"
// Logs error: "Cleanup failed!"
// Logs: "This still runs"
```

### **Example 9: Conditional Cleanup**
```javascript
const collector = ReactiveUtils.collector();

function setupFeature(enableAnalytics) {
  const timer = setInterval(doWork, 1000);
  collector.add(() => clearInterval(timer));
  
  if (enableAnalytics) {
    const analyticsTimer = setInterval(sendAnalytics, 5000);
    collector.add(() => clearInterval(analyticsTimer));
  }
}

setupFeature(true);
console.log(collector.size); // 2

setupFeature(false);
console.log(collector.size); // 1
```

### **Example 10: Nested Collectors**
```javascript
class App {
  constructor() {
    this.mainCollector = ReactiveUtils.collector();
    
    // Add sub-component collectors
    const header = this.createHeader();
    const sidebar = this.createSidebar();
    const content = this.createContent();
    
    // Add sub-collectors to main
    this.mainCollector.add(() => header.cleanup());
    this.mainCollector.add(() => sidebar.cleanup());
    this.mainCollector.add(() => content.cleanup());
  }
  
  createHeader() {
    const collector = ReactiveUtils.collector();
    // ... add header cleanups
    return collector;
  }
  
  createSidebar() {
    const collector = ReactiveUtils.collector();
    // ... add sidebar cleanups
    return collector;
  }
  
  createContent() {
    const collector = ReactiveUtils.collector();
    // ... add content cleanups
    return collector;
  }
  
  destroy() {
    // Cleans up all sub-components
    this.mainCollector.cleanup();
  }
}
```

---

## **Common Patterns**

### **Pattern 1: Component Cleanup**
```javascript
class Component {
  constructor() {
    this.collector = ReactiveUtils.collector();
  }
  
  destroy() {
    this.collector.cleanup();
  }
}
```

### **Pattern 2: Multiple Effects**
```javascript
const collector = ReactiveUtils.collector();

collector
  .add(effect(() => { /* effect 1 */ }))
  .add(effect(() => { /* effect 2 */ }))
  .add(effect(() => { /* effect 3 */ }));

collector.cleanup(); // Dispose all
```

### **Pattern 3: Event Listeners**
```javascript
const collector = ReactiveUtils.collector();

el.addEventListener('click', handler);
collector.add(() => el.removeEventListener('click', handler));
```

---

## **Collector API**

| Method/Property | Type | Description |
|----------------|------|-------------|
| `add(cleanup)` | Method | Add cleanup function (chainable) |
| `cleanup()` | Method | Execute all cleanups |
| `size` | Getter | Number of cleanup functions |
| `disposed` | Getter | Whether cleanup has been called |

---

## **Key Takeaways**

1. **Centralized Cleanup**: Manage multiple cleanups in one place
2. **Chainable**: Methods return `this` for method chaining
3. **Error Resilient**: Individual cleanup errors don't stop others
4. **Disposed State**: Prevents adding cleanups after disposal
5. **Empty After Cleanup**: Cleanups array is cleared after execution
6. **Common Use**: Components, effects, events, timers, resources
7. **Nested Support**: Collectors can contain other collectors
8. **Type Check**: Only adds function types

---

## **Summary**

`collector()` creates a cleanup collector object for managing multiple cleanup functions in a centralized way. It provides `add()` to register cleanups, `cleanup()` to execute them all, and properties `size` and `disposed` to inspect state. Cleanups are executed in order with error handling, making it perfect for component lifecycle management, effect disposal, event listener cleanup, and resource management. The collector becomes disposed after cleanup, preventing new additions.
