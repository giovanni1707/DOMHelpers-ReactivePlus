# `abort()` - Abort Async Operation

**Quick Start (30 seconds)**
```javascript
const asyncState = ReactiveUtils.asyncState();

// Start async operation
execute(asyncState, async (signal) => {
  const response = await fetch('/api/slow', { signal });
  return response.json();
});

// User clicks cancel button
abort(asyncState);
// Or: asyncState.$abort();

console.log(asyncState.loading); // false (aborted)
```

---

## **What is `abort()`?**

`abort()` is a **function** that manually cancels the currently running async operation by calling `abort()` on the internal AbortController and clearing the loading state.

**Key characteristics:**
- **Manual Cancellation**: Stops current async operation
- **Clears Loading**: Sets `loading: false`
- **Triggers AbortError**: Fetch requests throw AbortError
- **Cleanup**: Removes abort controller reference
- **Safe to Call**: No-op if nothing is running

---

## **Syntax**

```javascript
// Global function
abort(asyncState)

// Instance method
asyncState.$abort()
```

### **Parameters**
- `asyncState` (required) - The async state object
  - **Type**: `Object` (created with `asyncState()`)

### **Returns**
- **Type**: `void`

---

## **How it works**

```javascript
function abort(asyncState) {
  if (asyncState.abortController) {
    asyncState.abortController.abort(); // Triggers AbortError in fetch
    asyncState.loading = false;
    asyncState.abortController = null;
  }
}
```

---

## **Examples**

### **Example 1: Cancel Button**
```javascript
const asyncState = ReactiveUtils.asyncState();

document.querySelector('#load-btn').addEventListener('click', () => {
  execute(asyncState, async (signal) => {
    const response = await fetch('/api/data', { signal });
    return response.json();
  });
});

document.querySelector('#cancel-btn').addEventListener('click', () => {
  abort(asyncState);
});
```

### **Example 2: Timeout**
```javascript
const asyncState = ReactiveUtils.asyncState();

execute(asyncState, fetchData);

// Abort after 5 seconds
setTimeout(() => {
  if (asyncState.loading) {
    abort(asyncState);
    console.log('Request timed out');
  }
}, 5000);
```

### **Example 3: Navigation Away**
```javascript
const asyncState = ReactiveUtils.asyncState();

// Component mount
onMount(() => {
  execute(asyncState, fetchData);
});

// Component unmount - cancel pending requests
onUnmount(() => {
  abort(asyncState);
});
```

### **Example 4: Search with Debounce**
```javascript
const searchState = ReactiveUtils.asyncState();
let debounceTimer;

function search(query) {
  // Cancel current search
  abort(searchState);
  
  clearTimeout(debounceTimer);
  debounceTimer = setTimeout(() => {
    execute(searchState, async (signal) => {
      const response = await fetch(`/api/search?q=${query}`, { signal });
      return response.json();
    });
  }, 300);
}
```

### **Example 5: Conditional Abort**
```javascript
const asyncState = ReactiveUtils.asyncState();

function loadData(id) {
  // Only abort if something is loading
  if (asyncState.loading) {
    abort(asyncState);
  }
  
  execute(asyncState, async (signal) => {
    const response = await fetch(`/api/items/${id}`, { signal });
    return response.json();
  });
}
```

---

## **Common Patterns**

### **Pattern 1: Cancel Button**
```javascript
cancelBtn.onclick = () => abort(asyncState);
```

### **Pattern 2: Cleanup on Unmount**
```javascript
onUnmount(() => abort(asyncState));
```

### **Pattern 3: Timeout**
```javascript
setTimeout(() => {
  if (asyncState.loading) abort(asyncState);
}, 5000);
```

---

## **Key Takeaways**

1. **Manual Control**: Explicitly cancel async operations
2. **Clears Loading**: Sets `loading: false` immediately
3. **Safe**: No-op if nothing is running
4. **Cleanup**: Removes abort controller reference
5. **Common Use**: Cancel buttons, navigation, timeouts

---

## **Summary**

`abort()` manually cancels the currently running async operation by aborting the internal AbortController and clearing the loading state. It's useful for implementing cancel buttons, cleaning up on navigation/unmount, and enforcing timeouts. Safe to call anytime, even when nothing is running.
