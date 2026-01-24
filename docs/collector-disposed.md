# `collector.disposed` - Disposal State Property

**Quick Start (30 seconds)**
```javascript
const collector = ReactiveUtils.collector();

console.log(collector.disposed); // false (active)

collector.add(() => console.log('Cleanup'));
console.log(collector.disposed); // false (still active)

collector.cleanup();
console.log(collector.disposed); // true (disposed)

// Try to add after disposal
collector.add(() => console.log('Another cleanup'));
// Warns: "Cannot add to disposed collector"
```

---

## **What is `collector.disposed`?**

`collector.disposed` is a **getter property** that returns whether the collector has been disposed (cleanup has been called).

**Key characteristics:**
- **Getter Property**: Read-only, computed value
- **Boolean Type**: Always `true` or `false`
- **One-Way State**: Once `true`, stays `true`
- **Disposal Indicator**: Shows if cleanup has run

---

## **Syntax**

```javascript
const isDisposed = collector.disposed
```

### **Type**
```typescript
collector.disposed: boolean
```

### **Returns**
- **Type**: `boolean`
- **Values**:
  - `false` - Collector is active, can add cleanups
  - `true` - Collector is disposed, cannot add cleanups

---

## **How it works**

```javascript
get disposed() {
  return isDisposed;
}

// Set to true when cleanup() is called
cleanup() {
  if (isDisposed) return;
  
  isDisposed = true;
  // ... execute cleanups
}
```

---

## **Lifecycle**

```
Creation:        disposed = false
After add():     disposed = false
After cleanup(): disposed = true (permanent)
```

---

## **Examples**

### **Example 1: Check Before Adding**
```javascript
const collector = ReactiveUtils.collector();

function addCleanup(cleanup) {
  if (collector.disposed) {
    console.log('Cannot add - collector disposed');
    return;
  }
  
  collector.add(cleanup);
}

addCleanup(() => {}); // Added successfully

collector.cleanup();

addCleanup(() => {}); // Logs: "Cannot add - collector disposed"
```

### **Example 2: Component State**
```javascript
class Component {
  constructor() {
    this.collector = ReactiveUtils.collector();
  }
  
  isActive() {
    return !this.collector.disposed;
  }
  
  addResource(resource) {
    if (this.isActive()) {
      this.collector.add(() => resource.dispose());
    } else {
      console.warn('Component already destroyed');
    }
  }
  
  destroy() {
    if (!this.collector.disposed) {
      this.collector.cleanup();
    }
  }
}
```

### **Example 3: Conditional Cleanup**
```javascript
const collector = ReactiveUtils.collector();

collector.add(() => console.log('Cleanup'));

function cleanup() {
  if (collector.disposed) {
    console.log('Already cleaned up');
    return;
  }
  
  console.log('Cleaning up...');
  collector.cleanup();
}

cleanup(); // Logs: "Cleaning up...", "Cleanup"
cleanup(); // Logs: "Already cleaned up"
```

### **Example 4: Status Display**
```javascript
const collector = ReactiveUtils.collector();

function showStatus() {
  const status = collector.disposed ? 'Disposed' : 'Active';
  const count = collector.size;
  
  console.log(`Status: ${status}, Cleanups: ${count}`);
}

showStatus(); // "Status: Active, Cleanups: 0"

collector.add(() => {});
showStatus(); // "Status: Active, Cleanups: 1"

collector.cleanup();
showStatus(); // "Status: Disposed, Cleanups: 0"
```

### **Example 5: Prevent Double Disposal**
```javascript
const collector = ReactiveUtils.collector();

collector.add(() => console.log('Cleanup'));

function safeDispose() {
  if (!collector.disposed) {
    collector.cleanup();
  }
}

safeDispose(); // Runs cleanup
safeDispose(); // No-op (already disposed)
safeDispose(); // No-op
```

### **Example 6: Assert Active**
```javascript
const collector = ReactiveUtils.collector();

function requireActive(operation) {
  if (collector.disposed) {
    throw new Error(`Cannot ${operation}: collector disposed`);
  }
}

collector.add(() => {});

requireActive('add cleanup'); // OK

collector.cleanup();

try {
  requireActive('add cleanup');
} catch (error) {
  console.error(error.message);
  // "Cannot add cleanup: collector disposed"
}
```

### **Example 7: State Machine**
```javascript
const collector = ReactiveUtils.collector();

function getState() {
  if (collector.disposed) {
    return 'disposed';
  } else if (collector.size === 0) {
    return 'empty';
  } else {
    return 'active';
  }
}

console.log(getState()); // "empty"

collector.add(() => {});
console.log(getState()); // "active"

collector.cleanup();
console.log(getState()); // "disposed"
```

### **Example 8: Warning on Add**
```javascript
const collector = ReactiveUtils.collector();

const originalAdd = collector.add.bind(collector);

collector.add = function(cleanup) {
  if (this.disposed) {
    console.warn('⚠️ Adding to disposed collector!');
  }
  return originalAdd(cleanup);
};

collector.add(() => {}); // Works

collector.cleanup();

collector.add(() => {}); // Warns: "⚠️ Adding to disposed collector!"
```

### **Example 9: Lifecycle Logging**
```javascript
const collector = ReactiveUtils.collector();

function logState(action) {
  console.log(`[${action}] Disposed: ${collector.disposed}, Size: ${collector.size}`);
}

logState('Create');           // "Disposed: false, Size: 0"

collector.add(() => {});
logState('Add 1');           // "Disposed: false, Size: 1"

collector.add(() => {});
logState('Add 2');           // "Disposed: false, Size: 2"

collector.cleanup();
logState('Cleanup');         // "Disposed: true, Size: 0"
```

### **Example 10: Resource Pool**
```javascript
class ResourcePool {
  constructor() {
    this.collector = ReactiveUtils.collector();
    this.resources = new Set();
  }
  
  acquire(resource) {
    if (this.collector.disposed) {
      throw new Error('Pool is closed');
    }
    
    this.resources.add(resource);
    this.collector.add(() => resource.dispose());
  }
  
  close() {
    if (!this.collector.disposed) {
      console.log(`Closing pool with ${this.resources.size} resources`);
      this.collector.cleanup();
      this.resources.clear();
    }
  }
  
  isClosed() {
    return this.collector.disposed;
  }
}

const pool = new ResourcePool();
console.log(pool.isClosed()); // false

pool.acquire(resource1);
pool.acquire(resource2);

pool.close();
console.log(pool.isClosed()); // true

try {
  pool.acquire(resource3);
} catch (error) {
  console.error(error.message); // "Pool is closed"
}
```

---

## **Common Patterns**

### **Pattern 1: Check Before Add**
```javascript
if (!collector.disposed) {
  collector.add(cleanup);
}
```

### **Pattern 2: Check Before Cleanup**
```javascript
if (!collector.disposed) {
  collector.cleanup();
}
```

### **Pattern 3: Status Check**
```javascript
const isActive = !collector.disposed;
```

### **Pattern 4: Assert Active**
```javascript
if (collector.disposed) {
  throw new Error('Collector disposed');
}
```

---

## **Comparison with `size`**

| Property | Type | Meaning |
|----------|------|---------|
| `disposed` | `boolean` | Whether cleanup has been called |
| `size` | `number` | Number of cleanup functions |

```javascript
const collector = ReactiveUtils.collector();

// Initial state
collector.disposed; // false
collector.size;     // 0

// After adding
collector.add(() => {});
collector.disposed; // false
collector.size;     // 1

// After cleanup
collector.cleanup();
collector.disposed; // true
collector.size;     // 0
```

---

## **Key Takeaways**

1. **Getter Property**: Read-only boolean value
2. **One-Way State**: Once true, stays true
3. **Disposal Indicator**: Shows if cleanup has run
4. **Prevents Additions**: Used to block adding to disposed collector
5. **Check Before Operations**: Useful for validation
6. **Lifecycle Tracking**: Monitor collector state
7. **Common Use**: Guards, assertions, status checks

---

## **Summary**

`collector.disposed` is a getter property that returns whether the collector has been disposed (cleanup has been called). It starts as `false` and becomes `true` permanently after `collector.cleanup()` is called. Use it to check if the collector is active before adding cleanups, prevent double disposal, validate operations, track lifecycle state, and implement guards that throw errors when operations are attempted on disposed collectors.
