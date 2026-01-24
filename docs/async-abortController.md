# `asyncState.abortController` - AbortController Property

**Quick Start (30 seconds)**
```javascript
const asyncState = ReactiveUtils.asyncState();

console.log(asyncState.abortController); // null (no request running)

execute(asyncState, async (signal) => {
  console.log(asyncState.abortController); // AbortController instance
  const response = await fetch('/api/data', { signal });
  return response.json();
});

// After completion
console.log(asyncState.abortController); // null (request finished)

// Manual abort
if (asyncState.abortController) {
  asyncState.abortController.abort();
}
```

---

## **What is `asyncState.abortController`?**

`asyncState.abortController` is a **reactive property** that contains the current AbortController instance used for cancelling the ongoing async operation, or `null` if no operation is running.

**Key characteristics:**
- **Reactive**: Changes trigger effects
- **AbortController Instance**: Standard Web API AbortController
- **Initially Null**: Starts as `null`
- **Created on Execute**: New instance created for each execution
- **Cleared on Complete**: Set to `null` when operation finishes

---

## **Property Type**

```typescript
asyncState.abortController: AbortController | null
```

---

## **Lifecycle**

```
1. Create:     abortController = null
2. Execute:    abortController = new AbortController()
3. Complete:   abortController = null
4. Error:      abortController = null
5. Abort:      abortController = null
6. Reset:      abortController = null
```

---

## **AbortController API**

When set, `asyncState.abortController` provides:

```javascript
asyncState.abortController = {
  signal: AbortSignal,    // Signal to pass to fetch()
  abort(): void           // Method to cancel operation
}
```

---

## **Examples**

### **Example 1: Check if Request is Running**
```javascript
const asyncState = ReactiveUtils.asyncState();

effect(() => {
  const isRunning = asyncState.abortController !== null;
  console.log('Request running:', isRunning);
});

execute(asyncState, fetchData);
```

### **Example 2: Manual Abort**
```javascript
const asyncState = ReactiveUtils.asyncState();

execute(asyncState, fetchData);

// User clicks cancel
document.querySelector('#cancel-btn').addEventListener('click', () => {
  if (asyncState.abortController) {
    asyncState.abortController.abort();
  }
});
```

### **Example 3: Abort After Timeout**
```javascript
const asyncState = ReactiveUtils.asyncState();

execute(asyncState, fetchData);

// Abort if still running after 5 seconds
setTimeout(() => {
  if (asyncState.abortController) {
    asyncState.abortController.abort();
    console.log('Request timed out');
  }
}, 5000);
```

### **Example 4: Access Signal**
```javascript
const asyncState = ReactiveUtils.asyncState();

execute(asyncState, async (signal) => {
  // signal === asyncState.abortController.signal
  console.log(signal === asyncState.abortController.signal); // true
  
  const response = await fetch('/api/data', { signal });
  return response.json();
});
```

### **Example 5: Listen to Abort Event**
```javascript
const asyncState = ReactiveUtils.asyncState();

execute(asyncState, async (signal) => {
  signal.addEventListener('abort', () => {
    console.log('Request was aborted');
    cleanup();
  });
  
  const response = await fetch('/api/data', { signal });
  return response.json();
});
```

---

## **Common Patterns**

### **Pattern 1: Check if Running**
```javascript
if (asyncState.abortController) {
  // Request is running
}
```

### **Pattern 2: Manual Abort**
```javascript
asyncState.abortController?.abort();
```

### **Pattern 3: Timeout Abort**
```javascript
setTimeout(() => {
  asyncState.abortController?.abort();
}, 5000);
```

---

## **When is it Set?**

| Timing | Value |
|--------|-------|
| Before execute() | `null` |
| During execute() | `AbortController` instance |
| After success | `null` |
| After error | `null` |
| After abort() | `null` |
| After reset() | `null` |

---

## **Why It Exists**

The `abortController` property exists to:

1. **Enable Manual Cancellation**: Allows external code to abort requests
2. **Indicate Running State**: Non-null means request is active
3. **Support Fetch Cancellation**: Provides signal for fetch API
4. **Cleanup Resources**: Ensures proper cleanup on navigation/unmount

---

## **Key Takeaways**

1. **AbortController or Null**: Contains AbortController instance or `null`
2. **Created Per Request**: New instance for each `execute()`
3. **Cleared on Complete**: Set to `null` when request finishes
4. **Manual Abort**: Use `abortController.abort()` to cancel
5. **Reactive**: Can be observed in effects
6. **Internal Use**: Primarily managed internally by `execute()`

---

## **Summary**

`asyncState.abortController` is a reactive property containing the current AbortController instance used for cancelling the ongoing async operation, or `null` if no operation is running. It's created automatically when `execute()` starts and cleared when the operation completes, errors, or is aborted. While primarily for internal use, it can be accessed to manually abort operations, check if a request is running, or listen to abort events.
