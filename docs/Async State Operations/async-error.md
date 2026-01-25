# `asyncState.error` - Async Error Property

**Quick Start (30 seconds)**
```javascript
const asyncState = ReactiveUtils.asyncState();

console.log(asyncState.error); // null

await execute(asyncState, async () => {
  throw new Error('Failed to load');
});

console.log(asyncState.error); // Error: Failed to load
console.log(asyncState.error.message); // 'Failed to load'

// Reactive error display
effect(() => {
  if (asyncState.error) {
    showError(asyncState.error.message);
  }
});
```

---

## **What is `asyncState.error`?**

`asyncState.error` is a **reactive property** that contains the error object from a failed async operation, or `null` if no error has occurred.

**Key characteristics:**
- **Reactive**: Changes trigger effects
- **Error Object**: Contains Error instance or null
- **Initially Null**: Starts as `null`
- **Set on Error**: Updated when `execute()` fails
- **Cleared on Retry**: Set to `null` when new `execute()` starts

---

## **Property Type**

```typescript
asyncState.error: Error | null
```

---

## **Lifecycle**

```
1. Create:     error = null
2. Execute:    error = null (cleared)
3. Success:    error = null
4. Error:      error = Error instance
5. Retry:      error = null (cleared)
6. Reset:      error = null
```

---

## **Examples**

### **Example 1: Error Message Display**
```javascript
const asyncState = ReactiveUtils.asyncState();

effect(() => {
  const errorEl = document.querySelector('#error-message');
  
  if (asyncState.error) {
    errorEl.textContent = asyncState.error.message;
    errorEl.style.display = 'block';
  } else {
    errorEl.style.display = 'none';
  }
});

await execute(asyncState, fetchData);
```

### **Example 2: Error Types**
```javascript
const asyncState = ReactiveUtils.asyncState();

effect(() => {
  if (asyncState.error) {
    if (asyncState.error.name === 'NetworkError') {
      showMessage('Network error. Please check your connection.');
    } else if (asyncState.error.message.includes('404')) {
      showMessage('Resource not found.');
    } else {
      showMessage('An error occurred. Please try again.');
    }
  }
});
```

### **Example 3: Retry Button**
```javascript
const asyncState = ReactiveUtils.asyncState();

effect(() => {
  const retryBtn = document.querySelector('#retry-btn');
  
  if (asyncState.error) {
    retryBtn.style.display = 'block';
    retryBtn.onclick = () => refetch(asyncState);
  } else {
    retryBtn.style.display = 'none';
  }
});
```

### **Example 4: Error Boundary**
```javascript
const asyncState = ReactiveUtils.asyncState();

effect(() => {
  if (asyncState.error) {
    console.error('Async error:', asyncState.error);
    
    // Send to error tracking service
    sendErrorToTracker({
      message: asyncState.error.message,
      stack: asyncState.error.stack,
      timestamp: new Date().toISOString()
    });
  }
});
```

### **Example 5: Conditional UI**
```javascript
const asyncState = ReactiveUtils.asyncState();

effect(() => {
  const container = document.querySelector('#container');
  
  if (asyncState.loading) {
    container.innerHTML = '<p>Loading...</p>';
  } else if (asyncState.error) {
    container.innerHTML = `
      <div class="error">
        <h3>Error</h3>
        <p>${asyncState.error.message}</p>
        <button onclick="retryLoad()">Retry</button>
      </div>
    `;
  } else if (asyncState.data) {
    renderData(asyncState.data);
  }
});
```

---

## **Common Patterns**

### **Pattern 1: Show Error Message**
```javascript
effect(() => {
  if (asyncState.error) {
    alert(asyncState.error.message);
  }
});
```

### **Pattern 2: Error Display Element**
```javascript
effect(() => {
  errorEl.textContent = asyncState.error ? asyncState.error.message : '';
});
```

### **Pattern 3: Retry on Error**
```javascript
effect(() => {
  if (asyncState.error) {
    retryBtn.style.display = 'block';
    retryBtn.onclick = () => refetch(asyncState);
  } else {
    retryBtn.style.display = 'none';
  }
});
```

---

## **Error Properties**

When `asyncState.error` is set, it contains a standard Error object with:

```javascript
asyncState.error = {
  name: 'Error',           // Error type
  message: 'Error message', // Error description
  stack: '...',            // Stack trace
  // ... other properties
}
```

---

## **Key Takeaways**

1. **Error Object or Null**: Contains Error instance or `null`
2. **Reactive**: Changes trigger effects
3. **Initially Null**: Starts as `null`
4. **Cleared on Retry**: Set to `null` when new `execute()` starts
5. **Common Use**: Error messages, retry buttons, error tracking

---

## **Summary**

`asyncState.error` is a reactive property containing the error object from a failed async operation, or `null` if no error has occurred. It's automatically set when `execute()` fails and cleared when a new execution starts. Use it to display error messages, show retry buttons, log errors to tracking services, and handle different error types appropriately.
