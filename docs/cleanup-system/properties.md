# Collector Properties - Complete Reference

**Quick Start (30 seconds)**
```javascript
const collector = ReactiveUtils.collector();

// Check initial state
console.log(collector.size); // 0
console.log(collector.disposed); // false

// Add cleanup functions
collector.add(() => console.log('Cleanup 1'));
collector.add(() => console.log('Cleanup 2'));
collector.add(() => console.log('Cleanup 3'));

// Check size
console.log(collector.size); // 3
console.log(collector.disposed); // false

// Dispose collector
collector.cleanup();

// Check after disposal
console.log(collector.size); // 0
console.log(collector.disposed); // true

// Try to add after disposal
collector.add(() => console.log('Cleanup 4'));
console.log(collector.size); // 0 (cannot add to disposed collector)
```

---

## **What are Collector Properties?**

Collector properties are **read-only getters** that provide information about the state of a cleanup collector, allowing you to inspect how many cleanup functions are registered and whether the collector has been disposed.

**The 2 core properties:**
1. **`size`** - Returns the number of cleanup functions registered
2. **`disposed`** - Returns whether the collector has been disposed

**Key characteristics:**
- **Read-Only**: Cannot be set directly
- **Getters**: Computed from internal state
- **Always Current**: Reflect live collector state
- **Type Safe**: Predictable return types
- **Non-Reactive**: Not reactive state (plain getters)

---

## **Property Reference**

### **1. `collector.size`**

**Type**: Getter (number)
**Returns**: `number`
**Reactive**: No (plain getter)

Returns the current number of cleanup functions registered in the collector.

```javascript
const collector = ReactiveUtils.collector();

console.log(collector.size); // 0

collector.add(() => console.log('Cleanup 1'));
console.log(collector.size); // 1

collector.add(() => console.log('Cleanup 2'));
collector.add(() => console.log('Cleanup 3'));
console.log(collector.size); // 3

collector.cleanup();
console.log(collector.size); // 0 (all cleaned up)
```

**Implementation:**
```javascript
get size() {
  return cleanups.length;
}
```

**When it changes:**
- Increases when `add()` is called with valid function
- Stays same when `add()` is called on disposed collector
- Stays same when `add()` is called with non-function
- Resets to 0 when `cleanup()` is called

**Use cases:**
- Check if collector has cleanup functions
- Validate cleanup registration
- Debug memory leaks
- Monitor cleanup count
- Conditional cleanup logic

---

### **2. `collector.disposed`**

**Type**: Getter (boolean)
**Returns**: `boolean`
**Reactive**: No (plain getter)

Returns whether the collector has been disposed (cleanup has been executed).

```javascript
const collector = ReactiveUtils.collector();

console.log(collector.disposed); // false

collector.add(() => console.log('Cleanup'));
console.log(collector.disposed); // false (still active)

collector.cleanup();
console.log(collector.disposed); // true (disposed)

// Cannot add to disposed collector
collector.add(() => console.log('New cleanup'));
// Warning: Cannot add to disposed collector
console.log(collector.size); // 0 (not added)
```

**Implementation:**
```javascript
get disposed() {
  return isDisposed;
}
```

**When it changes:**
- Initially `false`
- Becomes `true` after `cleanup()` is called
- Remains `true` forever (cannot be reset)

**Use cases:**
- Prevent adding to disposed collector
- Validate collector state
- Debug cleanup execution
- Guard against double disposal
- Conditional cleanup logic

---

## **Examples**

### **Example 1: Monitoring Cleanup Count**
```javascript
const collector = ReactiveUtils.collector();

function addEventListeners() {
  const button = document.getElementById('btn');
  const input = document.getElementById('input');

  const handleClick = () => console.log('Clicked');
  const handleInput = () => console.log('Input changed');

  button.addEventListener('click', handleClick);
  input.addEventListener('input', handleInput);

  collector.add(() => button.removeEventListener('click', handleClick));
  collector.add(() => input.removeEventListener('input', handleInput));

  console.log(`Registered ${collector.size} cleanup functions`); // 2
}

addEventListeners();
```

### **Example 2: Validating Before Cleanup**
```javascript
const collector = ReactiveUtils.collector();

collector.add(() => console.log('Cleanup 1'));
collector.add(() => console.log('Cleanup 2'));

function safeCleanup() {
  if (collector.disposed) {
    console.log('Already disposed');
    return;
  }

  if (collector.size === 0) {
    console.log('No cleanup functions registered');
    return;
  }

  console.log(`Cleaning up ${collector.size} functions`);
  collector.cleanup();
  console.log(`Disposed: ${collector.disposed}`);
}

safeCleanup();
```

### **Example 3: Preventing Double Disposal**
```javascript
const collector = ReactiveUtils.collector();

collector.add(() => console.log('Cleanup'));

function cleanup() {
  if (collector.disposed) {
    console.warn('Collector already disposed!');
    return;
  }

  collector.cleanup();
  console.log('Cleanup complete');
}

cleanup(); // Cleanup complete
cleanup(); // Collector already disposed!
```

### **Example 4: Conditional Registration**
```javascript
const collector = ReactiveUtils.collector();
const maxCleanups = 10;

function registerCleanup(fn) {
  if (collector.disposed) {
    console.error('Cannot register: collector is disposed');
    return false;
  }

  if (collector.size >= maxCleanups) {
    console.error(`Cannot register: max cleanups (${maxCleanups}) reached`);
    return false;
  }

  collector.add(fn);
  console.log(`Registered cleanup (${collector.size}/${maxCleanups})`);
  return true;
}

for (let i = 0; i < 15; i++) {
  registerCleanup(() => console.log(`Cleanup ${i}`));
}
```

### **Example 5: Cleanup Progress Indicator**
```javascript
const collector = ReactiveUtils.collector();

// Add multiple cleanups
for (let i = 0; i < 5; i++) {
  collector.add(() => {
    console.log(`Cleaning up resource ${i + 1}`);
  });
}

console.log(`\n📊 Cleanup Status:`);
console.log(`Total cleanups: ${collector.size}`);
console.log(`Disposed: ${collector.disposed}`);

collector.cleanup();

console.log(`\n📊 After Cleanup:`);
console.log(`Remaining cleanups: ${collector.size}`);
console.log(`Disposed: ${collector.disposed}`);
```

### **Example 6: Resource Manager**
```javascript
class ResourceManager {
  constructor() {
    this.collector = ReactiveUtils.collector();
  }

  addResource(resource) {
    if (this.collector.disposed) {
      throw new Error('ResourceManager is disposed');
    }

    this.collector.add(() => resource.dispose());
    console.log(`Resources: ${this.collector.size}`);
  }

  getStats() {
    return {
      resourceCount: this.collector.size,
      isActive: !this.collector.disposed
    };
  }

  dispose() {
    if (this.collector.disposed) {
      console.log('Already disposed');
      return;
    }

    console.log(`Disposing ${this.collector.size} resources`);
    this.collector.cleanup();
  }
}

const manager = new ResourceManager();
manager.addResource({ dispose: () => console.log('Resource 1 disposed') });
manager.addResource({ dispose: () => console.log('Resource 2 disposed') });

console.log(manager.getStats()); // { resourceCount: 2, isActive: true }

manager.dispose();
console.log(manager.getStats()); // { resourceCount: 0, isActive: false }
```

### **Example 7: Cleanup Limit Warning**
```javascript
const collector = ReactiveUtils.collector();
const CLEANUP_WARNING_THRESHOLD = 50;

function addCleanup(fn) {
  if (collector.disposed) {
    console.error('Collector is disposed');
    return;
  }

  collector.add(fn);

  if (collector.size >= CLEANUP_WARNING_THRESHOLD) {
    console.warn(
      `⚠️  High cleanup count: ${collector.size} functions registered. ` +
      `Possible memory leak?`
    );
  }
}

// Simulate many cleanups
for (let i = 0; i < 60; i++) {
  addCleanup(() => console.log(`Cleanup ${i}`));
}
```

### **Example 8: Batch Registration Status**
```javascript
const collector = ReactiveUtils.collector();

function registerBatch(cleanupFns) {
  const initialSize = collector.size;

  if (collector.disposed) {
    console.error('Cannot register batch: collector is disposed');
    return { success: false, registered: 0 };
  }

  cleanupFns.forEach(fn => collector.add(fn));

  const registered = collector.size - initialSize;

  console.log(`Batch registration:`);
  console.log(`  Attempted: ${cleanupFns.length}`);
  console.log(`  Registered: ${registered}`);
  console.log(`  Total: ${collector.size}`);

  return { success: true, registered };
}

const cleanups = [
  () => console.log('Cleanup 1'),
  () => console.log('Cleanup 2'),
  () => console.log('Cleanup 3')
];

registerBatch(cleanups);
```

### **Example 9: State Inspector**
```javascript
const collector = ReactiveUtils.collector();

function inspectCollector() {
  const state = {
    size: collector.size,
    disposed: collector.disposed,
    isEmpty: collector.size === 0,
    hasCleanups: collector.size > 0,
    canAdd: !collector.disposed
  };

  console.log('\n🔍 Collector State:');
  console.log('─'.repeat(40));
  console.log(`Size: ${state.size}`);
  console.log(`Disposed: ${state.disposed}`);
  console.log(`Empty: ${state.isEmpty}`);
  console.log(`Has Cleanups: ${state.hasCleanups}`);
  console.log(`Can Add: ${state.canAdd}`);
  console.log('─'.repeat(40));

  return state;
}

inspectCollector();

collector.add(() => console.log('Cleanup 1'));
collector.add(() => console.log('Cleanup 2'));

inspectCollector();

collector.cleanup();

inspectCollector();
```

### **Example 10: Auto-Cleanup on Threshold**
```javascript
const collector = ReactiveUtils.collector();
const AUTO_CLEANUP_THRESHOLD = 100;

function addWithAutoCleanup(fn) {
  if (collector.disposed) {
    console.log('Creating new collector');
    // In real app, you'd reassign or reinitialize
    return;
  }

  collector.add(fn);

  if (collector.size >= AUTO_CLEANUP_THRESHOLD) {
    console.log(
      `⚠️  Threshold reached (${collector.size}). Auto-cleaning...`
    );
    collector.cleanup();
  }
}

// Simulate many registrations
for (let i = 0; i < 105; i++) {
  addWithAutoCleanup(() => console.log(`Cleanup ${i}`));
}
```

### **Example 11: Cleanup Statistics**
```javascript
const collector = ReactiveUtils.collector();
const stats = {
  totalAdded: 0,
  totalCleaned: 0,
  cleanupCount: 0
};

function addTracked(fn) {
  if (collector.disposed) {
    console.log('Collector is disposed');
    return;
  }

  collector.add(fn);
  stats.totalAdded++;

  console.log(`Added: ${stats.totalAdded}, Current: ${collector.size}`);
}

function cleanupTracked() {
  if (collector.disposed) {
    console.log('Already disposed');
    return;
  }

  stats.totalCleaned += collector.size;
  stats.cleanupCount++;

  collector.cleanup();

  console.log(`\n📊 Cleanup Statistics:`);
  console.log(`Total added: ${stats.totalAdded}`);
  console.log(`Total cleaned: ${stats.totalCleaned}`);
  console.log(`Cleanup calls: ${stats.cleanupCount}`);
  console.log(`Currently registered: ${collector.size}`);
  console.log(`Disposed: ${collector.disposed}`);
}

addTracked(() => console.log('Cleanup 1'));
addTracked(() => console.log('Cleanup 2'));
addTracked(() => console.log('Cleanup 3'));

cleanupTracked();
```

### **Example 12: Cleanup Validation**
```javascript
const collector = ReactiveUtils.collector();

function validateCollector() {
  const issues = [];

  if (collector.disposed && collector.size > 0) {
    issues.push('ERROR: Disposed collector has non-zero size');
  }

  if (!collector.disposed && collector.size === 0) {
    issues.push('WARNING: Active collector has no cleanups registered');
  }

  if (collector.size < 0) {
    issues.push('ERROR: Negative size (corrupted state)');
  }

  if (issues.length > 0) {
    console.log('🔴 Validation Issues:');
    issues.forEach(issue => console.log(`  - ${issue}`));
    return false;
  }

  console.log('✓ Collector state is valid');
  return true;
}

// Test different states
validateCollector(); // WARNING: Active collector has no cleanups

collector.add(() => console.log('Cleanup'));
validateCollector(); // Valid

collector.cleanup();
validateCollector(); // Valid
```

### **Example 13: Component Lifecycle Tracking**
```javascript
class Component {
  constructor(name) {
    this.name = name;
    this.collector = ReactiveUtils.collector();
  }

  mount() {
    console.log(`[${this.name}] Mounting...`);

    // Register cleanups
    this.collector.add(() => console.log(`[${this.name}] Cleanup: event listeners`));
    this.collector.add(() => console.log(`[${this.name}] Cleanup: timers`));
    this.collector.add(() => console.log(`[${this.name}] Cleanup: subscriptions`));

    console.log(`[${this.name}] Mounted with ${this.collector.size} cleanups`);
  }

  unmount() {
    if (this.collector.disposed) {
      console.warn(`[${this.name}] Already unmounted`);
      return;
    }

    console.log(`[${this.name}] Unmounting... (${this.collector.size} cleanups)`);
    this.collector.cleanup();
    console.log(`[${this.name}] Unmounted (disposed: ${this.collector.disposed})`);
  }

  getStatus() {
    return {
      name: this.name,
      cleanups: this.collector.size,
      mounted: !this.collector.disposed
    };
  }
}

const component = new Component('MyComponent');
console.log(component.getStatus()); // { name: 'MyComponent', cleanups: 0, mounted: true }

component.mount();
console.log(component.getStatus()); // { name: 'MyComponent', cleanups: 3, mounted: true }

component.unmount();
console.log(component.getStatus()); // { name: 'MyComponent', cleanups: 0, mounted: false }
```

### **Example 14: Cleanup Queue Manager**
```javascript
class CleanupQueue {
  constructor(maxSize = 50) {
    this.collector = ReactiveUtils.collector();
    this.maxSize = maxSize;
  }

  add(cleanup) {
    if (this.collector.disposed) {
      throw new Error('Queue is disposed');
    }

    if (this.isFull()) {
      throw new Error(`Queue is full (${this.maxSize} max)`);
    }

    this.collector.add(cleanup);
  }

  isFull() {
    return this.collector.size >= this.maxSize;
  }

  isEmpty() {
    return this.collector.size === 0;
  }

  getCapacity() {
    return {
      used: this.collector.size,
      available: this.maxSize - this.collector.size,
      total: this.maxSize,
      percentage: (this.collector.size / this.maxSize * 100).toFixed(1) + '%'
    };
  }

  dispose() {
    if (!this.collector.disposed) {
      this.collector.cleanup();
    }
  }
}

const queue = new CleanupQueue(10);

for (let i = 0; i < 7; i++) {
  queue.add(() => console.log(`Cleanup ${i}`));
}

console.log(queue.getCapacity());
// { used: 7, available: 3, total: 10, percentage: '70.0%' }

queue.dispose();
```

### **Example 15: Nested Collectors**
```javascript
const parentCollector = ReactiveUtils.collector();
const childCollector = ReactiveUtils.collector();

// Parent owns child
parentCollector.add(() => {
  if (!childCollector.disposed) {
    console.log(`Cleaning up child (${childCollector.size} cleanups)`);
    childCollector.cleanup();
  }
});

// Add cleanups to child
childCollector.add(() => console.log('Child cleanup 1'));
childCollector.add(() => console.log('Child cleanup 2'));

// Add cleanups to parent
parentCollector.add(() => console.log('Parent cleanup 1'));
parentCollector.add(() => console.log('Parent cleanup 2'));

console.log('\n📊 Before Cleanup:');
console.log(`Parent: ${parentCollector.size} cleanups, disposed: ${parentCollector.disposed}`);
console.log(`Child: ${childCollector.size} cleanups, disposed: ${childCollector.disposed}`);

// Cleanup parent (will cleanup child too)
parentCollector.cleanup();

console.log('\n📊 After Cleanup:');
console.log(`Parent: ${parentCollector.size} cleanups, disposed: ${parentCollector.disposed}`);
console.log(`Child: ${childCollector.size} cleanups, disposed: ${childCollector.disposed}`);
```

### **Example 16: Conditional Cleanup**
```javascript
const collector = ReactiveUtils.collector();

function conditionalCleanup(condition) {
  if (collector.disposed) {
    console.log('Collector already disposed');
    return;
  }

  if (!condition) {
    console.log('Condition not met, skipping cleanup');
    return;
  }

  if (collector.size === 0) {
    console.log('No cleanups to execute');
    return;
  }

  console.log(`Executing ${collector.size} cleanups`);
  collector.cleanup();
}

collector.add(() => console.log('Cleanup 1'));
collector.add(() => console.log('Cleanup 2'));

conditionalCleanup(false); // Condition not met
conditionalCleanup(true);  // Executing 2 cleanups
conditionalCleanup(true);  // Already disposed
```

### **Example 17: Cleanup Health Check**
```javascript
const collector = ReactiveUtils.collector();

function healthCheck() {
  const health = {
    size: collector.size,
    disposed: collector.disposed,
    status: 'unknown',
    recommendation: ''
  };

  if (collector.disposed) {
    health.status = 'disposed';
    health.recommendation = 'Collector is disposed and cannot be reused';
  } else if (collector.size === 0) {
    health.status = 'empty';
    health.recommendation = 'No cleanups registered';
  } else if (collector.size < 10) {
    health.status = 'healthy';
    health.recommendation = 'Normal operation';
  } else if (collector.size < 50) {
    health.status = 'warning';
    health.recommendation = 'Consider periodic cleanup';
  } else {
    health.status = 'critical';
    health.recommendation = 'High cleanup count - check for memory leaks';
  }

  console.log('\n🏥 Health Check:');
  console.log(`Status: ${health.status.toUpperCase()}`);
  console.log(`Size: ${health.size}`);
  console.log(`Disposed: ${health.disposed}`);
  console.log(`Recommendation: ${health.recommendation}`);

  return health;
}

// Test different states
healthCheck(); // empty

for (let i = 0; i < 5; i++) {
  collector.add(() => console.log(`Cleanup ${i}`));
}
healthCheck(); // healthy

for (let i = 0; i < 60; i++) {
  collector.add(() => console.log(`Cleanup ${i}`));
}
healthCheck(); // critical
```

### **Example 18: Cleanup Metrics Dashboard**
```javascript
const collector = ReactiveUtils.collector();
const metrics = {
  addedCount: 0,
  addAttempts: 0,
  addFailures: 0,
  cleanupCalls: 0
};

function monitoredAdd(fn) {
  metrics.addAttempts++;

  if (collector.disposed) {
    metrics.addFailures++;
    console.error('Add failed: collector disposed');
    return false;
  }

  if (typeof fn !== 'function') {
    metrics.addFailures++;
    console.error('Add failed: not a function');
    return false;
  }

  collector.add(fn);
  metrics.addedCount++;

  updateDashboard();
  return true;
}

function monitoredCleanup() {
  if (!collector.disposed) {
    metrics.cleanupCalls++;
    collector.cleanup();
  }

  updateDashboard();
}

function updateDashboard() {
  console.log('\n📊 Metrics Dashboard:');
  console.log('─'.repeat(50));
  console.log(`Add Attempts: ${metrics.addAttempts}`);
  console.log(`Successfully Added: ${metrics.addedCount}`);
  console.log(`Add Failures: ${metrics.addFailures}`);
  console.log(`Cleanup Calls: ${metrics.cleanupCalls}`);
  console.log(`Current Size: ${collector.size}`);
  console.log(`Disposed: ${collector.disposed}`);
  console.log(`Success Rate: ${(metrics.addedCount / metrics.addAttempts * 100).toFixed(1)}%`);
  console.log('─'.repeat(50));
}

// Test
monitoredAdd(() => console.log('Cleanup 1'));
monitoredAdd(() => console.log('Cleanup 2'));
monitoredAdd('not a function'); // Failure
monitoredCleanup();
monitoredAdd(() => console.log('Cleanup 3')); // Failure (disposed)
```

### **Example 19: Lifecycle State Machine**
```javascript
class CollectorStateMachine {
  constructor() {
    this.collector = ReactiveUtils.collector();
  }

  getState() {
    if (this.collector.disposed) {
      return 'DISPOSED';
    }

    if (this.collector.size === 0) {
      return 'EMPTY';
    }

    if (this.collector.size < 10) {
      return 'ACTIVE';
    }

    return 'FULL';
  }

  canTransitionTo(targetState) {
    const currentState = this.getState();

    const transitions = {
      EMPTY: ['ACTIVE', 'DISPOSED'],
      ACTIVE: ['FULL', 'EMPTY', 'DISPOSED'],
      FULL: ['DISPOSED', 'EMPTY'],
      DISPOSED: [] // Terminal state
    };

    return transitions[currentState]?.includes(targetState) || false;
  }

  add(fn) {
    if (!this.canTransitionTo('ACTIVE') && !this.canTransitionTo('FULL')) {
      console.error(`Cannot add: invalid transition from ${this.getState()}`);
      return false;
    }

    this.collector.add(fn);
    console.log(`State: ${this.getState()} (size: ${this.collector.size})`);
    return true;
  }

  dispose() {
    if (!this.canTransitionTo('DISPOSED')) {
      console.error(`Cannot dispose: invalid transition from ${this.getState()}`);
      return false;
    }

    this.collector.cleanup();
    console.log(`State: ${this.getState()}`);
    return true;
  }
}

const sm = new CollectorStateMachine();
console.log('Initial:', sm.getState()); // EMPTY

sm.add(() => console.log('Cleanup 1'));
console.log('After add:', sm.getState()); // ACTIVE

sm.dispose();
console.log('After dispose:', sm.getState()); // DISPOSED
```

### **Example 20: Memory Leak Detector**
```javascript
const collector = ReactiveUtils.collector();
const checkInterval = 1000; // Check every second
let lastSize = 0;
let growthCount = 0;
const GROWTH_THRESHOLD = 5; // Alert after 5 consecutive growths

function startLeakDetection() {
  const intervalId = setInterval(() => {
    if (collector.disposed) {
      console.log('Collector disposed, stopping leak detection');
      clearInterval(intervalId);
      return;
    }

    const currentSize = collector.size;

    if (currentSize > lastSize) {
      growthCount++;

      console.log(`⚠️  Collector growing: ${lastSize} → ${currentSize} (${growthCount}/${GROWTH_THRESHOLD})`);

      if (growthCount >= GROWTH_THRESHOLD) {
        console.error(
          `\n🔴 MEMORY LEAK DETECTED!\n` +
          `Collector has grown continuously for ${GROWTH_THRESHOLD} checks.\n` +
          `Current size: ${currentSize} cleanups\n` +
          `This may indicate a memory leak.`
        );

        // Optionally cleanup to prevent further growth
        // collector.cleanup();
      }
    } else if (currentSize < lastSize) {
      growthCount = 0; // Reset if size decreased
    }

    lastSize = currentSize;
  }, checkInterval);

  // Add to collector so it gets cleaned up too
  collector.add(() => clearInterval(intervalId));

  return intervalId;
}

// Start monitoring
startLeakDetection();

// Simulate leak
const leakInterval = setInterval(() => {
  collector.add(() => console.log('Potential leak cleanup'));
}, 500);

// Stop leak after 10 seconds
setTimeout(() => {
  clearInterval(leakInterval);
  console.log('\nStopped simulated leak');
}, 10000);
```

---

## **Common Patterns**

### **Pattern 1: Check Size**
```javascript
if (collector.size > 0) {
  console.log(`Has ${collector.size} cleanups`);
}
```

### **Pattern 2: Check Disposed**
```javascript
if (collector.disposed) {
  console.log('Collector is disposed');
}
```

### **Pattern 3: Validate Before Add**
```javascript
if (!collector.disposed) {
  collector.add(cleanup);
}
```

### **Pattern 4: Conditional Cleanup**
```javascript
if (!collector.disposed && collector.size > 0) {
  collector.cleanup();
}
```

### **Pattern 5: Status Check**
```javascript
const status = {
  size: collector.size,
  active: !collector.disposed
};
```

---

## **Property Comparison Table**

| Property | Type | Returns | When Updated | Use Case |
|----------|------|---------|--------------|----------|
| `size` | number | Cleanup count | On add/cleanup | Monitor cleanup count |
| `disposed` | boolean | Disposal state | On cleanup | Check if disposed |

---

## **State Matrix**

| size | disposed | State | Can Add? |
|------|----------|-------|----------|
| 0 | false | Empty | ✓ Yes |
| N > 0 | false | Active | ✓ Yes |
| 0 | true | Disposed (empty) | ✗ No |
| N > 0 | true | Invalid* | ✗ No |

*Should never occur - cleanup() sets size to 0 and disposed to true atomically

---

## **Behavioral Notes**

### **After `add()`**
- `size` increases by 1 (if function and not disposed)
- `disposed` remains unchanged

### **After `cleanup()`**
- `size` becomes 0
- `disposed` becomes true
- Both change atomically

### **After attempting `add()` on disposed**
- `size` remains 0
- `disposed` remains true
- Warning logged to console

---

## **Key Takeaways**

1. **`size`** - Returns number of registered cleanup functions
2. **`disposed`** - Returns whether collector has been disposed
3. **Read-Only** - Both are getters, cannot be set directly
4. **Non-Reactive** - Plain getters, not reactive state
5. **Always Accurate** - Reflect current collector state
6. **Validation** - Use to validate operations
7. **Debug Aid** - Essential for debugging cleanup issues
8. **State Inspection** - Check state before operations
9. **Atomic Changes** - Both update atomically on cleanup
10. **Terminal State** - Once disposed, cannot be undisposed

---

## **Summary**

Collector properties provide read-only access to cleanup collector state through two simple getters: `size` (returns the number of registered cleanup functions) and `disposed` (returns whether the collector has been disposed via cleanup()). The `size` property increases when cleanup functions are added via `add()` and resets to 0 when `cleanup()` is called, making it useful for monitoring cleanup registration, validating operations, and detecting potential memory leaks from excessive cleanup accumulation. The `disposed` property starts as `false` and permanently becomes `true` after `cleanup()` is called, preventing further additions and enabling validation before operations. Both properties are non-reactive plain getters that always reflect the current internal state of the collector. Use `size` to check if cleanups are registered and monitor cleanup counts, and use `disposed` to validate that operations don't target disposed collectors. These properties are essential for building robust cleanup logic, debugging lifecycle issues, preventing memory leaks, and implementing defensive programming patterns around resource cleanup.
