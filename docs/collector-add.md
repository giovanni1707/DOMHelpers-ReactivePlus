# `collector.add()` - Add Cleanup Function

**Quick Start (30 seconds)**
```javascript
const collector = ReactiveUtils.collector();

// Add cleanup functions
collector.add(() => console.log('Cleanup 1'));
collector.add(() => console.log('Cleanup 2'));
collector.add(() => console.log('Cleanup 3'));

// Chainable
collector
  .add(() => clearInterval(timer))
  .add(() => ws.close())
  .add(() => observer.disconnect());

console.log(collector.size); // 6

collector.cleanup();
// Executes all 6 cleanup functions
```

---

## **What is `collector.add()`?**

`collector.add()` is a **method** on a collector object that registers a cleanup function to be executed when `collector.cleanup()` is called. It returns the collector for method chaining.

**Key characteristics:**
- **Registers Cleanup**: Adds function to cleanup queue
- **Chainable**: Returns `this` for chaining
- **Type Check**: Only accepts functions
- **Order Preserved**: Executes in registration order
- **Disposed Check**: Warns if collector is disposed

---

## **Syntax**

```javascript
collector.add(cleanup)
```

### **Parameters**
- `cleanup` (required) - Cleanup function to register
  - **Type**: `Function`
  - **Returns**: Ignored
  - **Executes**: When `collector.cleanup()` is called

### **Returns**
- **Type**: `collector` object (for chaining)

---

## **How it works**

```javascript
add(cleanup) {
  if (isDisposed) {
    console.warn('Cannot add to disposed collector');
    return this;
  }
  
  if (typeof cleanup === 'function') {
    cleanups.push(cleanup);
  }
  return this;
}
```

---

## **Examples**

### **Example 1: Basic Usage**
```javascript
const collector = ReactiveUtils.collector();

collector.add(() => {
  console.log('Cleaning up resource');
});

collector.cleanup(); // Logs: "Cleaning up resource"
```

### **Example 2: Multiple Cleanups**
```javascript
const collector = ReactiveUtils.collector();

// Add multiple cleanups
collector.add(() => console.log('Cleanup 1'));
collector.add(() => console.log('Cleanup 2'));
collector.add(() => console.log('Cleanup 3'));

collector.cleanup();
// Logs in order: "Cleanup 1", "Cleanup 2", "Cleanup 3"
```

### **Example 3: Chaining**
```javascript
const collector = ReactiveUtils.collector();

collector
  .add(() => console.log('First'))
  .add(() => console.log('Second'))
  .add(() => console.log('Third'));

collector.cleanup();
// Logs: "First", "Second", "Third"
```

### **Example 4: Effect Disposal**
```javascript
const collector = ReactiveUtils.collector();
const state = ReactiveUtils.state({ count: 0 });

// Add effect disposal
const dispose = effect(() => {
  console.log('Count:', state.count);
});

collector.add(dispose);

// Later: clean up the effect
collector.cleanup(); // Disposes the effect
```

### **Example 5: Event Listeners**
```javascript
const collector = ReactiveUtils.collector();
const button = document.querySelector('#btn');

const handleClick = () => console.log('Clicked');
button.addEventListener('click', handleClick);

// Register cleanup
collector.add(() => {
  button.removeEventListener('click', handleClick);
});

// Clean up listener
collector.cleanup();
```

### **Example 6: Timers**
```javascript
const collector = ReactiveUtils.collector();

// Interval
const interval = setInterval(() => console.log('Tick'), 1000);
collector.add(() => clearInterval(interval));

// Timeout
const timeout = setTimeout(() => console.log('Delayed'), 5000);
collector.add(() => clearTimeout(timeout));

// Animation frame
let frameId = requestAnimationFrame(animate);
collector.add(() => cancelAnimationFrame(frameId));

// Clean up all timers
collector.cleanup();
```

### **Example 7: Resource Management**
```javascript
const collector = ReactiveUtils.collector();

// WebSocket
const ws = new WebSocket('wss://example.com');
collector.add(() => ws.close());

// Observer
const observer = new IntersectionObserver(callback);
collector.add(() => observer.disconnect());

// Media stream
navigator.mediaDevices.getUserMedia({ video: true })
  .then(stream => {
    collector.add(() => {
      stream.getTracks().forEach(track => track.stop());
    });
  });

// Clean up all resources
collector.cleanup();
```

### **Example 8: Disposed Collector Warning**
```javascript
const collector = ReactiveUtils.collector();

collector.add(() => console.log('Cleanup 1'));
collector.cleanup();

// Try to add after cleanup
collector.add(() => console.log('Cleanup 2'));
// Warns: "Cannot add to disposed collector"

console.log(collector.size); // 0 (not added)
```

### **Example 9: Non-Function Ignored**
```javascript
const collector = ReactiveUtils.collector();

collector.add(() => console.log('Function')); // Added
collector.add('not a function'); // Ignored (type check)
collector.add(123); // Ignored
collector.add(null); // Ignored

console.log(collector.size); // 1 (only the function)
```

### **Example 10: Async Cleanup**
```javascript
const collector = ReactiveUtils.collector();

collector.add(async () => {
  await saveDataToServer();
  console.log('Data saved');
});

collector.add(async () => {
  await closeConnection();
  console.log('Connection closed');
});

// Note: cleanup() doesn't await, these run in background
collector.cleanup();
```

---

## **Common Patterns**

### **Pattern 1: Effect Cleanup**
```javascript
collector.add(effect(() => { /* ... */ }));
```

### **Pattern 2: Event Listener**
```javascript
el.addEventListener('click', handler);
collector.add(() => el.removeEventListener('click', handler));
```

### **Pattern 3: Timer Cleanup**
```javascript
const id = setInterval(fn, 1000);
collector.add(() => clearInterval(id));
```

### **Pattern 4: Resource Cleanup**
```javascript
const resource = createResource();
collector.add(() => resource.dispose());
```

### **Pattern 5: Chaining**
```javascript
collector
  .add(cleanup1)
  .add(cleanup2)
  .add(cleanup3);
```

---

## **Execution Order**

Cleanups are executed in the order they were added:

```javascript
const collector = ReactiveUtils.collector();

collector.add(() => console.log('1'));
collector.add(() => console.log('2'));
collector.add(() => console.log('3'));

collector.cleanup();
// Output: "1", "2", "3"
```

---

## **Error Handling**

Individual cleanup errors don't stop execution:

```javascript
const collector = ReactiveUtils.collector();

collector.add(() => console.log('Before error'));
collector.add(() => {
  throw new Error('Cleanup failed');
});
collector.add(() => console.log('After error'));

collector.cleanup();
// Logs: "Before error"
// Logs error: "Cleanup failed"
// Logs: "After error" (still runs!)
```

---

## **Key Takeaways**

1. **Registers Function**: Adds cleanup to queue
2. **Chainable**: Returns collector for chaining
3. **Type Check**: Only functions are added
4. **Order Preserved**: Executes in registration order
5. **Disposed Check**: Warns if adding to disposed collector
6. **Error Resilient**: Individual errors don't stop others
7. **Common Use**: Effects, listeners, timers, resources
8. **Async Support**: Can add async functions (not awaited)

---

## **Summary**

`collector.add()` registers a cleanup function to be executed when the collector is cleaned up. It performs type checking to ensure only functions are added, warns if the collector is already disposed, and returns the collector for method chaining. Cleanups are executed in registration order with error handling that prevents one failure from stopping others, making it perfect for managing multiple cleanup operations in component lifecycle, resource management, and effect disposal scenarios.
