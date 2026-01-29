# `reset()` - Reset Async State

**Quick Start (30 seconds)**
```javascript
const asyncState = ReactiveUtils.asyncState(null);

await execute(asyncState, fetchData);
console.log(asyncState.data); // [1, 2, 3]
console.log(asyncState.requestId); // 1

reset(asyncState);
// Or: asyncState.$reset();

console.log(asyncState.data); // null (initial value)
console.log(asyncState.loading); // false
console.log(asyncState.error); // null
console.log(asyncState.requestId); // 0
```

---

## **What is `reset()`?**

`reset()` is a **function** that resets an async state object back to its initial values, canceling any pending operations and clearing all data, errors, and loading state.

**Key characteristics:**
- **Full Reset**: Returns to initial state
- **Aborts Pending**: Cancels running operations
- **Clears Data**: Sets data back to initial value
- **Clears Error**: Removes error state
- **Resets Counter**: Sets requestId back to 0

---

## **Syntax**

```javascript
// Global function
reset(asyncState)

// Instance method
asyncState.$reset()
```

### **Parameters**
- `asyncState` (required) - The async state object
  - **Type**: `Object` (created with `asyncState(initialValue)`)

### **Returns**
- **Type**: `void`

---

## **How it works**

```javascript
function reset(asyncState) {
  // 1. Abort any pending operations
  asyncState.$abort();
  
  // 2. Reset to initial values
  asyncState.data = initialValue;
  asyncState.loading = false;
  asyncState.error = null;
  asyncState.requestId = 0;
}
```

---

## **Examples**

### **Example 1: Clear Results**
```javascript
const searchState = ReactiveUtils.asyncState(null);

// Load search results
await execute(searchState, fetchResults);

// User clears search
querySelector('#clear-btn').addEventListener('click', () => {
  reset(searchState);
  querySelector('#search-input').value = '';
});
```

### **Example 2: Form Submission**
```javascript
const submitState = ReactiveUtils.asyncState(null);

async function handleSubmit() {
  const result = await execute(submitState, async (signal) => {
    const response = await fetch('/api/submit', {
      method: 'POST',
      signal,
      body: JSON.stringify(formData)
    });
    return response.json();
  });
  
  if (result.success) {
    // Clear form and state after successful submit
    form.reset();
    reset(submitState);
  }
}
```

### **Example 3: Retry After Error**
```javascript
const asyncState = ReactiveUtils.asyncState();

async function loadData() {
  const result = await execute(asyncState, fetchData);
  
  if (result.error) {
    // Show error for 3 seconds, then reset
    setTimeout(() => {
      reset(asyncState);
    }, 3000);
  }
}
```

### **Example 4: Navigation/Tab Change**
```javascript
const dataState = ReactiveUtils.asyncState();

function switchTab(tabId) {
  // Reset state when switching tabs
  reset(dataState);
  
  // Load new tab data
  execute(dataState, () => fetchTabData(tabId));
}
```

### **Example 5: Logout**
```javascript
const userState = ReactiveUtils.asyncState(null);

function logout() {
  // Clear all user data
  reset(userState);
  
  // Clear other state...
  localStorage.clear();
  
  // Redirect
  window.location.href = '/login';
}
```

---

## **Common Patterns**

### **Pattern 1: Clear Button**
```javascript
clearBtn.onclick = () => reset(asyncState);
```

### **Pattern 2: After Successful Action**
```javascript
if (result.success) {
  reset(asyncState);
}
```

### **Pattern 3: Navigation Change**
```javascript
onRouteChange(() => {
  reset(asyncState);
});
```

---

## **State Before/After Reset**

```javascript
// Before reset
asyncState = {
  data: [1, 2, 3],
  loading: false,
  error: null,
  requestId: 5,
  abortController: null
}

reset(asyncState);

// After reset
asyncState = {
  data: null,        // Initial value
  loading: false,
  error: null,
  requestId: 0,
  abortController: null
}
```

---

## **Key Takeaways**

1. **Full Reset**: Returns to initial state
2. **Aborts First**: Cancels pending operations
3. **Clears Everything**: Data, error, loading, requestId
4. **Preserves Initial Value**: Sets data to initial value passed to `asyncState()`
5. **Common Use**: Clear buttons, navigation, logout, retry logic

---

## **Summary**

`reset()` resets an async state object back to its initial values, first aborting any pending operations and then clearing all data, errors, and loading state. The requestId counter is reset to 0, and the data property is set back to the initial value provided when creating the async state. It's commonly used for clear buttons, navigation changes, logout, and retry logic after errors.
