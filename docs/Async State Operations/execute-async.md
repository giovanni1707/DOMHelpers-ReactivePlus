# `execute()` - Execute Async Operation

**Quick Start (30 seconds)**
```javascript
const asyncState = ReactiveUtils.asyncState();

// Execute async operation
await execute(asyncState, async (signal) => {
  const response = await fetch('/api/data', { signal });
  return response.json();
});

// Or use instance method
await asyncState.$execute(async (signal) => {
  const response = await fetch('/api/data', { signal });
  return response.json();
});

// React to loading/data/error
effect(() => {
  if (asyncState.loading) {
    console.log('Loading...');
  } else if (asyncState.error) {
    console.log('Error:', asyncState.error);
  } else if (asyncState.data) {
    console.log('Data:', asyncState.data);
  }
});
```

---

## **What is `execute()`?**

`execute()` is a **function** that executes an async operation on an async state object, automatically managing loading, error, and data states. It handles race conditions, request cancellation, and proper cleanup.

**Key characteristics:**
- **Race Condition Prevention**: Only latest request updates state
- **Automatic Cancellation**: Cancels previous request when new one starts
- **AbortSignal Support**: Provides signal for request cancellation
- **State Management**: Automatically updates loading, error, data
- **Return Object**: Returns success/error status

---

## **Syntax**

```javascript
// Global function
await execute(asyncState, fn)

// Instance method
await asyncState.$execute(fn)
```

### **Parameters**
- `asyncState` (required) - The async state object
  - **Type**: `Object` (created with `asyncState()`)
- `fn` (required) - Async function to execute
  - **Type**: `(signal: AbortSignal) => Promise<any>`
  - **Receives**: `AbortSignal` for cancellation support

### **Returns**
- **Type**: `Promise<Object>`
- **Success**: `{ success: true, data: any }`
- **Stale**: `{ success: false, stale: true }` (outdated request)
- **Aborted**: `{ success: false, aborted: true }`
- **Error**: `{ success: false, error: Error }`

---

## **How it works**

```javascript
async function execute(asyncState, fn) {
  // 1. Cancel previous request
  if (asyncState.abortController) {
    asyncState.abortController.abort();
  }
  
  // 2. Create new abort controller
  const requestId = ++asyncState.requestId;
  asyncState.abortController = new AbortController();
  
  // 3. Set loading state
  asyncState.loading = true;
  asyncState.error = null;
  
  try {
    // 4. Execute async function
    const result = await fn(signal);
    
    // 5. Only update if still latest request
    if (requestId === asyncState.requestId) {
      asyncState.data = result;
    }
    
    return { success: true, data: result };
  } catch (error) {
    // 6. Handle errors
    if (requestId === asyncState.requestId) {
      asyncState.error = error;
    }
    return { success: false, error };
  } finally {
    // 7. Clear loading state
    if (requestId === asyncState.requestId) {
      asyncState.loading = false;
    }
  }
}
```

---

## **Examples**

### **Example 1: Basic API Call**
```javascript
const asyncState = ReactiveUtils.asyncState();

await execute(asyncState, async (signal) => {
  const response = await fetch('/api/users', { signal });
  return response.json();
});

console.log(asyncState.data); // User data
```

### **Example 2: With AbortSignal**
```javascript
const asyncState = ReactiveUtils.asyncState();

await execute(asyncState, async (signal) => {
  const response = await fetch('/api/data', { signal });
  
  if (signal.aborted) {
    throw new Error('Request was cancelled');
  }
  
  return response.json();
});
```

### **Example 3: Reactive UI**
```javascript
const asyncState = ReactiveUtils.asyncState();

effect(() => {
  const status = document.querySelector('#status');
  
  if (asyncState.loading) {
    status.textContent = 'Loading...';
  } else if (asyncState.error) {
    status.textContent = `Error: ${asyncState.error.message}`;
  } else if (asyncState.data) {
    status.textContent = `Loaded ${asyncState.data.length} items`;
  }
});

await execute(asyncState, fetchData);
```

### **Example 4: Handle Result**
```javascript
const result = await execute(asyncState, async (signal) => {
  const response = await fetch('/api/save', {
    method: 'POST',
    signal,
    body: JSON.stringify(data)
  });
  return response.json();
});

if (result.success) {
  console.log('Saved:', result.data);
} else if (result.aborted) {
  console.log('Request was cancelled');
} else if (result.error) {
  console.error('Save failed:', result.error);
}
```

### **Example 5: Search with Auto-Cancel**
```javascript
const searchState = ReactiveUtils.asyncState();

async function search(query) {
  await execute(searchState, async (signal) => {
    // Previous search automatically cancelled
    const response = await fetch(`/api/search?q=${query}`, { signal });
    return response.json();
  });
}

// User types fast - only last search completes
search('a');
search('ab');
search('abc'); // Only this one updates the state
```

### **Example 6: With Timeout**
```javascript
const asyncState = ReactiveUtils.asyncState();

await execute(asyncState, async (signal) => {
  const timeoutId = setTimeout(() => {
    signal.aborted || console.log('Still running...');
  }, 1000);
  
  try {
    const response = await fetch('/api/slow', { signal });
    return response.json();
  } finally {
    clearTimeout(timeoutId);
  }
});
```

### **Example 7: Paginated Data**
```javascript
const asyncState = ReactiveUtils.asyncState();

async function loadPage(page) {
  await execute(asyncState, async (signal) => {
    const response = await fetch(`/api/items?page=${page}`, { signal });
    return response.json();
  });
}

// Navigate pages - auto-cancels pending requests
await loadPage(1);
await loadPage(2);
await loadPage(3);
```

### **Example 8: With Options Callback**
```javascript
const asyncState = ReactiveUtils.asyncState(null, {
  onSuccess: (data) => {
    console.log('Success:', data);
    showNotification('Data loaded successfully');
  },
  onError: (error) => {
    console.error('Error:', error);
    showNotification('Failed to load data', 'error');
  }
});

await execute(asyncState, fetchData);
```

### **Example 9: Conditional Execution**
```javascript
const asyncState = ReactiveUtils.asyncState();

async function loadData(id) {
  if (!id) return;
  
  const result = await execute(asyncState, async (signal) => {
    const response = await fetch(`/api/items/${id}`, { signal });
    return response.json();
  });
  
  if (result.success) {
    updateUI(result.data);
  }
}
```

### **Example 10: Parallel Requests (Multiple States)**
```javascript
const usersState = ReactiveUtils.asyncState();
const postsState = ReactiveUtils.asyncState();
const commentsState = ReactiveUtils.asyncState();

// Execute in parallel
await Promise.all([
  execute(usersState, fetchUsers),
  execute(postsState, fetchPosts),
  execute(commentsState, fetchComments)
]);

// All data available
console.log(usersState.data, postsState.data, commentsState.data);
```

---

## **Common Patterns**

### **Pattern 1: Basic Fetch**
```javascript
await execute(asyncState, async (signal) => {
  const response = await fetch('/api/data', { signal });
  return response.json();
});
```

### **Pattern 2: With Error Handling**
```javascript
const result = await execute(asyncState, async (signal) => {
  const response = await fetch('/api/data', { signal });
  if (!response.ok) throw new Error('Failed to load');
  return response.json();
});

if (!result.success) {
  handleError(result.error);
}
```

### **Pattern 3: Search/Filter**
```javascript
async function search(query) {
  await execute(searchState, async (signal) => {
    const response = await fetch(`/api/search?q=${query}`, { signal });
    return response.json();
  });
}
```

### **Pattern 4: Reactive Display**
```javascript
effect(() => {
  if (asyncState.loading) showSpinner();
  else if (asyncState.error) showError(asyncState.error);
  else if (asyncState.data) showData(asyncState.data);
});
```

---

## **Race Condition Prevention**

```
Time  │ Request #1        Request #2        State
──────┼───────────────────────────────────────────
0ms   │ Start (ID: 1)     -                 loading: true
      │ requestId = 1                       requestId: 1
      │                                     
100ms │ Pending...        Start (ID: 2)     loading: true
      │ (gets aborted)    requestId = 2     requestId: 2
      │                                     
200ms │ Returns (ID: 1)   Pending...        -
      │ IGNORED!          -                 (ID doesn't match)
      │                                     
300ms │ -                 Returns (ID: 2)   loading: false
      │ -                 ACCEPTED!         data: [...]
```

**Only the latest request updates the state!**

---

## **Key Takeaways**

1. **Automatic Loading State**: Sets `loading: true` during execution
2. **Race Condition Safe**: Only latest request updates state
3. **Auto-Cancellation**: Cancels previous request automatically
4. **AbortSignal Provided**: Use for fetch cancellation
5. **Error Handling**: Catches errors and sets `error` property
6. **Return Object**: Provides success/error/aborted status
7. **Reactive**: All state changes trigger effects
8. **Instance or Global**: Use `asyncState.$execute()` or `execute(asyncState)`

---

## **Summary**

`execute()` executes an async operation on an async state object, automatically managing loading, error, and data states. It prevents race conditions by tracking request IDs, automatically cancels previous requests, and provides an AbortSignal for proper request cancellation. The function returns a result object indicating success, error, or cancellation status, making it perfect for API calls, searches, and any async operations that need automatic state management and cancellation support.
