# `asyncState.requestId` - Request Sequence Tracker Property

**Quick Start (30 seconds)**
```javascript
const asyncState = ReactiveUtils.asyncState();

console.log(asyncState.requestId); // 0 (initial)

execute(asyncState, fetchUsers);
console.log(asyncState.requestId); // 1

execute(asyncState, fetchPosts);
console.log(asyncState.requestId); // 2

execute(asyncState, fetchComments);
console.log(asyncState.requestId); // 3

// Each execution increments the ID
```

---

## **What is `asyncState.requestId`?**

`asyncState.requestId` is a **reactive number property** that tracks the sequence number of async requests, incrementing with each new execution. It's used internally to prevent race conditions by ensuring only the latest request updates the state.

**Key characteristics:**
- **Reactive**: Changes trigger effects
- **Auto-Incremented**: Increases with each `execute()`
- **Initially 0**: Starts at `0`
- **Race Condition Prevention**: Ensures latest request wins
- **Read-Only (effectively)**: Should not be manually modified

---

## **Property Type**

```typescript
asyncState.requestId: number
```

---

## **Purpose**

The `requestId` is used internally to prevent race conditions:

```javascript
// Request #1 starts (requestId = 1)
execute(asyncState, slowFetch);

// Request #2 starts (requestId = 2)
execute(asyncState, fastFetch);

// Request #2 completes first (requestId = 2 matches current)
// ✓ Updates state

// Request #1 completes second (requestId = 1 < current 2)
// ✗ Ignored (stale request)
```

---

## **Lifecycle**

```
1. Create:      requestId = 0
2. Execute #1:  requestId = 1
3. Execute #2:  requestId = 2
4. Execute #3:  requestId = 3
5. Reset:       requestId = 0
```

---

## **Examples**

### **Example 1: Request Tracking**
```javascript
const asyncState = ReactiveUtils.asyncState();

effect(() => {
  console.log(`Current request ID: ${asyncState.requestId}`);
});

await execute(asyncState, fetch1); // Logs: Current request ID: 1
await execute(asyncState, fetch2); // Logs: Current request ID: 2
await execute(asyncState, fetch3); // Logs: Current request ID: 3
```

### **Example 2: Debug Race Conditions**
```javascript
const asyncState = ReactiveUtils.asyncState();

async function debugFetch(name, delay) {
  const startId = asyncState.requestId + 1;
  
  await execute(asyncState, async (signal) => {
    console.log(`${name} started (ID: ${startId})`);
    await new Promise(resolve => setTimeout(resolve, delay));
    console.log(`${name} completed (ID: ${startId}, current: ${asyncState.requestId})`);
    return { name, delay };
  });
}

// Start slow request, then fast request
debugFetch('Slow', 2000);  // ID: 1
debugFetch('Fast', 100);   // ID: 2

// Output:
// Slow started (ID: 1)
// Fast started (ID: 2)
// Fast completed (ID: 2, current: 2) ← Updates state
// Slow completed (ID: 1, current: 2) ← Ignored
```

### **Example 3: Analytics**
```javascript
const asyncState = ReactiveUtils.asyncState();

effect(() => {
  if (!asyncState.loading && asyncState.requestId > 0) {
    sendAnalytics({
      event: 'data_loaded',
      requestNumber: asyncState.requestId,
      hasError: !!asyncState.error
    });
  }
});
```

### **Example 4: Reset Counter**
```javascript
const asyncState = ReactiveUtils.asyncState();

execute(asyncState, fetch1); // requestId: 1
execute(asyncState, fetch2); // requestId: 2

reset(asyncState); // requestId: 0

execute(asyncState, fetch3); // requestId: 1 (starts over)
```

---

## **How It Prevents Race Conditions**

```javascript
// Internal logic in execute()
async function execute(asyncState, fn) {
  // Increment and store current request ID
  const requestId = ++asyncState.requestId;
  
  const result = await fn(signal);
  
  // Only update if this is still the latest request
  if (requestId === asyncState.requestId) {
    asyncState.data = result;  // ✓ Update
  } else {
    // This request is stale, ignore it
  }
}
```

**Example scenario:**
```
Time  Request  Action              requestId  Result
0ms   #1       Start (slow)        1          -
100ms #2       Start (fast)        2          (cancels #1)
200ms #2       Complete            2          ✓ Updates (2 === 2)
300ms #1       Complete (aborted)  2          ✗ Ignored (1 < 2)
```

---

## **Common Patterns**

### **Pattern 1: Check Request Number**
```javascript
effect(() => {
  console.log(`Request #${asyncState.requestId}`);
});
```

### **Pattern 2: First vs Subsequent Loads**
```javascript
effect(() => {
  if (asyncState.requestId === 0) {
    // Never loaded
  } else if (asyncState.requestId === 1) {
    // First load
  } else {
    // Subsequent loads (refetch)
  }
});
```

---

## **Key Takeaways**

1. **Request Sequence**: Tracks number of executions
2. **Auto-Incremented**: Increases with each `execute()`
3. **Race Condition Prevention**: Ensures only latest request updates state
4. **Initially 0**: Starts at `0`, first execute sets to `1`
5. **Reset to 0**: `reset()` sets back to `0`
6. **Internal Use**: Primarily used internally, rarely accessed directly

---

## **Summary**

`asyncState.requestId` is a reactive number property that tracks the sequence number of async requests, incrementing with each execution. It's used internally to prevent race conditions by ensuring only the latest request can update the state. While primarily for internal use, it can be useful for debugging race conditions, tracking request counts, and implementing analytics.
