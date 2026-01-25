# `scope()` - Create Cleanup Scope

**Quick Start (30 seconds)**
```javascript
// Create a cleanup scope
const dispose = ReactiveUtils.scope((add) => {
  // Register cleanups within the scope
  add(() => console.log('Cleanup 1'));
  add(() => console.log('Cleanup 2'));
  
  // Setup effects
  add(effect(() => {
    console.log('Effect running');
  }));
  
  // Setup event listeners
  const handler = () => console.log('Click');
  document.addEventListener('click', handler);
  add(() => document.removeEventListener('click', handler));
});

// Later: dispose the entire scope
dispose();
// Logs: "Cleanup 1", "Cleanup 2", removes effect, removes listener
```

---

## **What is `scope()`?**

`scope()` is a **function** that creates a cleanup scope by wrapping a collector in a convenient API. It executes a setup function that receives an `add` callback for registering cleanups, then returns a single disposal function.

**Key characteristics:**
- **Scoped Cleanup**: All cleanups in one scope
- **Immediate Execution**: Setup function runs immediately
- **Single Disposal**: Returns one function to clean up everything
- **Collector Wrapper**: Uses collector internally
- **Clean API**: Simple add callback interface

---

## **Syntax**

```javascript
const dispose = ReactiveUtils.scope(fn)
```

### **Parameters**
- `fn` (required) - Setup function
  - **Type**: `(add: Function) => void`
  - **Parameter**: `add(cleanup)` - Function to register cleanups
  - **Executes**: Immediately when scope() is called

### **Returns**
- **Type**: `Function`
- **Behavior**: Disposes all registered cleanups when called

---

## **How it works**

```javascript
function scope(fn) {
  const collector = this.collector();
  
  fn((cleanup) => collector.add(cleanup));
  
  return () => collector.cleanup();
}
```

**Flow:**
1. Creates a collector
2. Calls `fn` with `add` callback
3. `add` registers cleanups in collector
4. Returns disposal function that calls `collector.cleanup()`

---

## **Examples**

### **Example 1: Basic Scope**
```javascript
const dispose = ReactiveUtils.scope((add) => {
  console.log('Setting up...');
  
  add(() => console.log('Tearing down...'));
});

dispose();
// Logs: "Setting up..."
// Logs: "Tearing down..."
```

### **Example 2: Multiple Effects in Scope**
```javascript
const state = ReactiveUtils.state({ count: 0, name: '' });

const dispose = ReactiveUtils.scope((add) => {
  // Effect 1
  add(effect(() => {
    console.log('Count:', state.count);
  }));
  
  // Effect 2
  add(effect(() => {
    console.log('Name:', state.name);
  }));
  
  // Effect 3
  add(effect(() => {
    document.title = `${state.name}: ${state.count}`;
  }));
});

// Clean up all effects at once
dispose();
```

### **Example 3: Event Listeners Scope**
```javascript
const dispose = ReactiveUtils.scope((add) => {
  const button = querySelector('#btn');
  const input = querySelector('#input');
  const form = querySelector('#form');
  
  const handleClick = () => console.log('Click');
  const handleInput = () => console.log('Input');
  const handleSubmit = (e) => {
    e.preventDefault();
    console.log('Submit');
  };
  
  button.addEventListener('click', handleClick);
  add(() => button.removeEventListener('click', handleClick));
  
  input.addEventListener('input', handleInput);
  add(() => input.removeEventListener('input', handleInput));
  
  form.addEventListener('submit', handleSubmit);
  add(() => form.removeEventListener('submit', handleSubmit));
});

// Remove all listeners
dispose();
```

### **Example 4: Component Lifecycle**
```javascript
class Component {
  mount() {
    this.dispose = ReactiveUtils.scope((add) => {
      // Reactive state
      this.state = ReactiveUtils.state({ data: null });
      
      // Data fetching effect
      add(effect(async () => {
        this.state.data = await fetchData();
      }));
      
      // Auto-save effect
      add(effect(() => {
        if (this.state.data) {
          localStorage.setItem('data', JSON.stringify(this.state.data));
        }
      }));
      
      // Keyboard shortcut
      const handleKeyboard = (e) => {
        if (e.ctrlKey && e.key === 's') {
          this.save();
        }
      };
      document.addEventListener('keydown', handleKeyboard);
      add(() => document.removeEventListener('keydown', handleKeyboard));
      
      // Cleanup on navigate away
      window.addEventListener('beforeunload', this.confirmLeave);
      add(() => window.removeEventListener('beforeunload', this.confirmLeave));
    });
  }
  
  unmount() {
    if (this.dispose) {
      this.dispose();
    }
  }
}
```

### **Example 5: Timer Scope**
```javascript
const dispose = ReactiveUtils.scope((add) => {
  // Interval
  const interval = setInterval(() => {
    console.log('Tick');
  }, 1000);
  add(() => clearInterval(interval));
  
  // Timeout
  const timeout = setTimeout(() => {
    console.log('Delayed action');
  }, 5000);
  add(() => clearTimeout(timeout));
  
  // Animation frame
  let frameId;
  const animate = () => {
    console.log('Frame');
    frameId = requestAnimationFrame(animate);
  };
  frameId = requestAnimationFrame(animate);
  add(() => cancelAnimationFrame(frameId));
});

// Cancel all timers
dispose();
```

### **Example 6: WebSocket Scope**
```javascript
const dispose = ReactiveUtils.scope((add) => {
  const ws = new WebSocket('wss://example.com/socket');
  
  ws.onopen = () => {
    console.log('Connected');
  };
  
  ws.onmessage = (event) => {
    console.log('Message:', event.data);
  };
  
  ws.onerror = (error) => {
    console.error('WebSocket error:', error);
  };
  
  // Clean up WebSocket
  add(() => {
    console.log('Closing WebSocket');
    ws.close();
  });
  
  // Also clean up listeners
  add(() => {
    ws.onopen = null;
    ws.onmessage = null;
    ws.onerror = null;
  });
});

// Close WebSocket and remove listeners
dispose();
```

### **Example 7: API Request Scope**
```javascript
const dispose = ReactiveUtils.scope((add) => {
  const abortController = new AbortController();
  
  fetch('/api/data', { signal: abortController.signal })
    .then(response => response.json())
    .then(data => console.log(data))
    .catch(error => {
      if (error.name !== 'AbortError') {
        console.error('Fetch error:', error);
      }
    });
  
  // Abort fetch on cleanup
  add(() => {
    console.log('Aborting fetch');
    abortController.abort();
  });
});

// Abort the request
dispose();
```

### **Example 8: Observer Scope**
```javascript
const dispose = ReactiveUtils.scope((add) => {
  const element = querySelector('#target');
  
  // Resize Observer
  const resizeObserver = new ResizeObserver((entries) => {
    console.log('Resized:', entries[0].contentRect);
  });
  resizeObserver.observe(element);
  add(() => resizeObserver.disconnect());
  
  // Intersection Observer
  const intersectionObserver = new IntersectionObserver((entries) => {
    console.log('Intersection:', entries[0].isIntersecting);
  });
  intersectionObserver.observe(element);
  add(() => intersectionObserver.disconnect());
  
  // Mutation Observer
  const mutationObserver = new MutationObserver((mutations) => {
    console.log('Mutations:', mutations.length);
  });
  mutationObserver.observe(element, { childList: true, subtree: true });
  add(() => mutationObserver.disconnect());
});

// Disconnect all observers
dispose();
```

### **Example 9: Route Scope**
```javascript
function setupRoute(path) {
  return ReactiveUtils.scope((add) => {
    console.log(`Setting up route: ${path}`);
    
    // Load route data
    const state = ReactiveUtils.state({ data: null });
    
    add(effect(async () => {
      state.data = await fetchRouteData(path);
    }));
    
    // Route-specific listeners
    const handlePopState = () => {
      console.log('History changed');
    };
    window.addEventListener('popstate', handlePopState);
    add(() => window.removeEventListener('popstate', handlePopState));
    
    // Analytics
    sendPageView(path);
    add(() => sendPageLeave(path));
  });
}

// When route changes
let currentDispose = setupRoute('/home');

// Later: navigate to new route
currentDispose(); // Clean up old route
currentDispose = setupRoute('/about'); // Setup new route
```

### **Example 10: Feature Flag Scope**
```javascript
function enableFeature(featureName) {
  return ReactiveUtils.scope((add) => {
    console.log(`Enabling feature: ${featureName}`);
    
    // Feature-specific code
    if (featureName === 'analytics') {
      const tracker = initAnalytics();
      add(() => tracker.shutdown());
    } else if (featureName === 'chat') {
      const chat = initChat();
      add(() => chat.disconnect());
    }
    
    // Feature state
    const state = ReactiveUtils.state({ enabled: true });
    
    add(effect(() => {
      console.log(`${featureName} enabled:`, state.enabled);
    }));
    
    // Cleanup notification
    add(() => {
      console.log(`Disabling feature: ${featureName}`);
    });
  });
}

const disposeAnalytics = enableFeature('analytics');
const disposeChat = enableFeature('chat');

// Later: disable features
disposeAnalytics();
disposeChat();
```

---

## **Common Patterns**

### **Pattern 1: Component Mount/Unmount**
```javascript
class Component {
  mount() {
    this.dispose = ReactiveUtils.scope((add) => {
      // Setup effects, listeners, etc.
    });
  }
  
  unmount() {
    this.dispose?.();
  }
}
```

### **Pattern 2: Effect Group**
```javascript
const dispose = ReactiveUtils.scope((add) => {
  add(effect(() => { /* effect 1 */ }));
  add(effect(() => { /* effect 2 */ }));
  add(effect(() => { /* effect 3 */ }));
});
```

### **Pattern 3: Event Listener Group**
```javascript
const dispose = ReactiveUtils.scope((add) => {
  el.addEventListener('event', handler);
  add(() => el.removeEventListener('event', handler));
});
```

---

## **Comparison with collector()**

| Feature | `collector()` | `scope(fn)` |
|---------|---------------|-------------|
| Returns | Collector object | Disposal function |
| Setup | Manual `add()` calls | Callback with `add` parameter |
| Execution | Explicit `collector.cleanup()` | Call returned function |
| API | Object with methods | Single function |
| Use Case | Manual control | Scoped setup/teardown |

```javascript
// collector() - manual control
const collector = ReactiveUtils.collector();
collector.add(cleanup1);
collector.add(cleanup2);
collector.cleanup(); // Explicit

// scope() - scoped control
const dispose = ReactiveUtils.scope((add) => {
  add(cleanup1);
  add(cleanup2);
});
dispose(); // Simple
```

---

## **Key Takeaways**

1. **Scoped Setup**: All setup code in one function
2. **Single Disposal**: Returns one function to clean everything
3. **Immediate Execution**: Setup function runs immediately
4. **Clean API**: Simple `add(cleanup)` interface
5. **Collector Wrapper**: Uses collector internally
6. **Common Use**: Component lifecycle, route changes, feature flags
7. **Error Handling**: Individual cleanup errors don't stop others
8. **Composable**: Can nest scopes

---

## **Summary**

`scope()` creates a cleanup scope by executing a setup function that receives an `add` callback for registering cleanups. It returns a single disposal function that cleans up all registered resources. The setup function runs immediately, making it perfect for component lifecycle management, route initialization, feature flag setup, and any scenario where you need to group multiple cleanups into a single scope with a simple disposal API.
