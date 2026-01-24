# `refetch()` - Refetch with Last Function

**Quick Start (30 seconds)**
```javascript
const asyncState = ReactiveUtils.asyncState();

// Initial fetch
await execute(asyncState, async (signal) => {
  const response = await fetch('/api/data', { signal });
  return response.json();
});

// Later... user clicks refresh
await refetch(asyncState);
// Or: await asyncState.$refetch();

// Runs the same fetch function again
```

---

## **What is `refetch()`?**

`refetch()` is a **function** that re-executes the last async function that was passed to `execute()`, providing an easy way to refresh/reload data without repeating the function definition.

**Key characteristics:**
- **Remembers Function**: Stores last executed function
- **One-Line Refresh**: Simple data reloading
- **Same Parameters**: Re-runs with saved function
- **Returns Promise**: Like `execute()`
- **Error if No Function**: Returns error if never executed

---

## **Syntax**

```javascript
// Global function
await refetch(asyncState)

// Instance method
await asyncState.$refetch()
```

### **Parameters**
- `asyncState` (required) - The async state object
  - **Type**: `Object` (created with `asyncState()`)

### **Returns**
- **Type**: `Promise<Object>`
- **Success**: `{ success: true, data: any }`
- **No Function**: `{ success: false, error: Error('No function to refetch') }`

---

## **How it works**

```javascript
async function refetch(asyncState) {
  if (asyncState.lastFn) {
    return asyncState.$execute(asyncState.lastFn);
  }
  
  return {
    success: false,
    error: new Error('No function to refetch')
  };
}
```

---

## **Examples**

### **Example 1: Refresh Button**
```javascript
const asyncState = ReactiveUtils.asyncState();

// Initial load
await execute(asyncState, async (signal) => {
  const response = await fetch('/api/users', { signal });
  return response.json();
});

// Refresh button
document.querySelector('#refresh-btn').addEventListener('click', async () => {
  await refetch(asyncState);
});
```

### **Example 2: Auto-Refresh Timer**
```javascript
const asyncState = ReactiveUtils.asyncState();

// Initial fetch
await execute(asyncState, fetchData);

// Auto-refresh every 30 seconds
setInterval(() => {
  refetch(asyncState);
}, 30000);
```

### **Example 3: After Mutation**
```javascript
const listState = ReactiveUtils.asyncState();

// Load list
await execute(listState, async (signal) => {
  const response = await fetch('/api/items', { signal });
  return response.json();
});

// After adding new item
async function addItem(item) {
  await fetch('/api/items', {
    method: 'POST',
    body: JSON.stringify(item)
  });
  
  // Refresh list
  await refetch(listState);
}
```

### **Example 4: Retry After Error**
```javascript
const asyncState = ReactiveUtils.asyncState();

effect(() => {
  if (asyncState.error) {
    const retryBtn = document.querySelector('#retry-btn');
    retryBtn.style.display = 'block';
    
    retryBtn.onclick = () => refetch(asyncState);
  }
});
```

### **Example 5: Polling**
```javascript
const asyncState = ReactiveUtils.asyncState();

// Initial load
await execute(asyncState, fetchStatus);

// Poll every 5 seconds
let pollInterval = setInterval(() => {
  refetch(asyncState);
}, 5000);

// Stop polling
function stopPolling() {
  clearInterval(pollInterval);
}
```

### **Example 6: Pull-to-Refresh**
```javascript
const asyncState = ReactiveUtils.asyncState();

await execute(asyncState, fetchData);

// Pull-to-refresh gesture
let startY = 0;
document.addEventListener('touchstart', (e) => {
  startY = e.touches[0].clientY;
});

document.addEventListener('touchend', (e) => {
  const endY = e.changedTouches[0].clientY;
  const distance = endY - startY;
  
  if (distance > 100 && window.scrollY === 0) {
    refetch(asyncState);
  }
});
```

### **Example 7: Conditional Refetch**
```javascript
const asyncState = ReactiveUtils.asyncState();

await execute(asyncState, fetchData);

async function refreshIfStale() {
  const age = Date.now() - asyncState.lastFetched;
  const fiveMinutes = 5 * 60 * 1000;
  
  if (age > fiveMinutes) {
    await refetch(asyncState);
  }
}
```

### **Example 8: Keyboard Shortcut**
```javascript
const asyncState = ReactiveUtils.asyncState();

await execute(asyncState, fetchData);

// Ctrl+R to refresh
document.addEventListener('keydown', (e) => {
  if (e.ctrlKey && e.key === 'r') {
    e.preventDefault();
    refetch(asyncState);
  }
});
```

---

## **Common Patterns**

### **Pattern 1: Refresh Button**
```javascript
refreshBtn.onclick = () => refetch(asyncState);
```

### **Pattern 2: Auto-Refresh**
```javascript
setInterval(() => refetch(asyncState), 30000);
```

### **Pattern 3: After Mutation**
```javascript
async function saveChanges() {
  await saveData();
  await refetch(listState);
}
```

### **Pattern 4: Retry on Error**
```javascript
if (asyncState.error) {
  retryBtn.onclick = () => refetch(asyncState);
}
```

---

## **Comparison with execute()**

| Method | When to use |
|--------|-------------|
| `execute(asyncState, fn)` | First time or different function |
| `refetch(asyncState)` | Reload with same function |

```javascript
// Initial load
await execute(asyncState, fetchUsers);

// Refresh with same function
await refetch(asyncState);

// Different function
await execute(asyncState, fetchAdmins);

// Refresh with new function
await refetch(asyncState); // Fetches admins now
```

---

## **Key Takeaways**

1. **Remembers Function**: Stores last executed function
2. **One-Line Refresh**: Simple `refetch(asyncState)`
3. **Requires Previous Execute**: Must call `execute()` first
4. **Updates Last Function**: Each `execute()` updates what `refetch()` will run
5. **Common Use**: Refresh buttons, auto-refresh, retry, polling
6. **Same Behavior**: Acts exactly like re-calling `execute()`

---

## **Summary**

`refetch()` re-executes the last async function that was passed to `execute()`, providing a simple way to refresh/reload data without repeating the function definition. It's perfect for refresh buttons, auto-refresh timers, retry after error, polling, and any scenario where you need to reload data using the same function. The function must be called after at least one `execute()` call, otherwise it returns an error.
