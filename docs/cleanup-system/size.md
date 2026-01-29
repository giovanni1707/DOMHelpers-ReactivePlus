# `collector.size` - Cleanup Function Count Property

**Quick Start (30 seconds)**
```javascript
const collector = ReactiveUtils.collector();

console.log(collector.size); // 0 (empty)

collector.add(() => console.log('Cleanup 1'));
console.log(collector.size); // 1

collector.add(() => console.log('Cleanup 2'));
collector.add(() => console.log('Cleanup 3'));
console.log(collector.size); // 3

collector.cleanup();
console.log(collector.size); // 0 (cleaned up)
```

---

## **What is `collector.size`?**

`collector.size` is a **getter property** that returns the current number of cleanup functions registered in the collector.

**Key characteristics:**
- **Getter Property**: Read-only, computed value
- **Live Count**: Updates as cleanups are added
- **Zero After Cleanup**: Returns 0 after disposal
- **Number Type**: Always returns a number

---

## **Syntax**

```javascript
const count = collector.size
```

### **Type**
```typescript
collector.size: number
```

### **Returns**
- **Type**: `number`
- **Value**: Number of registered cleanup functions
- **Range**: `0` to `Infinity`

---

## **How it works**

```javascript
get size() {
  return cleanups.length;
}
```

---

## **Examples**

### **Example 1: Check Before Cleanup**
```javascript
const collector = ReactiveUtils.collector();

collector.add(() => console.log('Cleanup 1'));
collector.add(() => console.log('Cleanup 2'));

if (collector.size > 0) {
  console.log(`${collector.size} cleanups to run`);
  collector.cleanup();
}
```

### **Example 2: Track Additions**
```javascript
const collector = ReactiveUtils.collector();

console.log('Initial:', collector.size); // 0

collector.add(() => {});
console.log('After 1st add:', collector.size); // 1

collector.add(() => {});
console.log('After 2nd add:', collector.size); // 2

collector.add(() => {});
console.log('After 3rd add:', collector.size); // 3
```

### **Example 3: Conditional Logic**
```javascript
const collector = ReactiveUtils.collector();

// Add some cleanups conditionally
if (featureEnabled) {
  collector.add(() => cleanupFeature());
}

// Check if any cleanups were added
if (collector.size === 0) {
  console.log('No cleanup needed');
} else {
  console.log(`Will clean up ${collector.size} items`);
  collector.cleanup();
}
```

### **Example 4: Logging**
```javascript
const collector = ReactiveUtils.collector();

function addCleanup(cleanup) {
  collector.add(cleanup);
  console.log(`Total cleanups: ${collector.size}`);
}

addCleanup(() => console.log('Cleanup 1')); // Logs: "Total cleanups: 1"
addCleanup(() => console.log('Cleanup 2')); // Logs: "Total cleanups: 2"
addCleanup(() => console.log('Cleanup 3')); // Logs: "Total cleanups: 3"
```

### **Example 5: Size Changes Over Time**
```javascript
const collector = ReactiveUtils.collector();

console.log('Start:', collector.size); // 0

collector.add(() => {});
collector.add(() => {});
collector.add(() => {});
console.log('After adds:', collector.size); // 3

collector.cleanup();
console.log('After cleanup:', collector.size); // 0
```

### **Example 6: Component Resource Tracking**
```javascript
class Component {
  constructor() {
    this.collector = ReactiveUtils.collector();
  }
  
  addResource(resource) {
    this.collector.add(() => resource.dispose());
    console.log(`Component has ${this.collector.size} resources`);
  }
  
  getResourceCount() {
    return this.collector.size;
  }
  
  destroy() {
    console.log(`Cleaning up ${this.collector.size} resources`);
    this.collector.cleanup();
  }
}

const component = new Component();
component.addResource(resource1); // "Component has 1 resources"
component.addResource(resource2); // "Component has 2 resources"
console.log(component.getResourceCount()); // 2
component.destroy(); // "Cleaning up 2 resources"
```

### **Example 7: Assert Non-Empty**
```javascript
const collector = ReactiveUtils.collector();

// Setup code...
collector.add(() => cleanup1());
collector.add(() => cleanup2());

// Assert we have cleanups before proceeding
console.assert(collector.size > 0, 'No cleanups registered!');

collector.cleanup();
```

### **Example 8: Progress Indicator**
```javascript
const collector = ReactiveUtils.collector();
let cleaned = 0;

collector.add(() => {
  cleaned++;
  console.log(`Cleanup ${cleaned}/${collector.size}`);
});

collector.add(() => {
  cleaned++;
  console.log(`Cleanup ${cleaned}/${collector.size}`);
});

collector.add(() => {
  cleaned++;
  console.log(`Cleanup ${cleaned}/${collector.size}`);
});

// Note: size is captured before cleanup starts
const total = collector.size;
collector.cleanup();
// Logs: "Cleanup 1/3", "Cleanup 2/3", "Cleanup 3/3"
```

### **Example 9: Empty Check**
```javascript
const collector = ReactiveUtils.collector();

function safeCleanup() {
  if (collector.size === 0) {
    console.log('Nothing to clean up');
    return;
  }
  
  console.log(`Cleaning up ${collector.size} items`);
  collector.cleanup();
}

safeCleanup(); // "Nothing to clean up"

collector.add(() => {});
safeCleanup(); // "Cleaning up 1 items"
```

### **Example 10: Statistics**
```javascript
const collector = ReactiveUtils.collector();

const stats = {
  get cleanupCount() {
    return collector.size;
  },
  get isDisposed() {
    return collector.disposed;
  },
  get hasCleanups() {
    return collector.size > 0;
  }
};

collector.add(() => {});
collector.add(() => {});

console.log('Stats:', stats);
// { cleanupCount: 2, isDisposed: false, hasCleanups: true }

collector.cleanup();
console.log('Stats:', stats);
// { cleanupCount: 0, isDisposed: true, hasCleanups: false }
```

---

## **Common Patterns**

### **Pattern 1: Check Before Cleanup**
```javascript
if (collector.size > 0) {
  collector.cleanup();
}
```

### **Pattern 2: Empty Check**
```javascript
if (collector.size === 0) {
  console.log('No cleanups');
}
```

### **Pattern 3: Log Count**
```javascript
console.log(`${collector.size} cleanups registered`);
```

### **Pattern 4: Assert Non-Empty**
```javascript
console.assert(collector.size > 0, 'Expected cleanups');
```

---

## **Lifecycle**

```
Creation:      size = 0
After add():   size = 1
After add():   size = 2
After add():   size = 3
After cleanup(): size = 0
```

---

## **Comparison with `disposed`**

| Property | Type | Meaning |
|----------|------|---------|
| `size` | `number` | Number of cleanup functions |
| `disposed` | `boolean` | Whether cleanup has been called |

```javascript
const collector = ReactiveUtils.collector();

collector.add(() => {});
collector.add(() => {});

console.log(collector.size); // 2
console.log(collector.disposed); // false

collector.cleanup();

console.log(collector.size); // 0
console.log(collector.disposed); // true
```

---

## **Key Takeaways**

1. **Getter Property**: Read-only computed value
2. **Live Count**: Updates as cleanups are added
3. **Number Type**: Always returns a number
4. **Zero After Cleanup**: Resets to 0 when cleaned
5. **Check Before Cleanup**: Useful for conditional cleanup
6. **Tracking**: Monitor how many cleanups are registered
7. **Debugging**: Helpful for understanding cleanup state

---

## **Summary**

`collector.size` is a getter property that returns the current number of cleanup functions registered in the collector. It updates live as cleanups are added and resets to 0 after `collector.cleanup()` is called. Use it to check if cleanups exist before running disposal, log the number of registered cleanups, track resource counts in components, and debug cleanup state. It's a simple but essential property for understanding and managing collector state.
