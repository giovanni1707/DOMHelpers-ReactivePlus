# `asyncState.isError` - Error State Computed Property

**Quick Start (30 seconds)**
```javascript
const asyncState = ReactiveUtils.asyncState();

console.log(asyncState.isError); // false (no error)

await execute(asyncState, async () => {
  throw new Error('Failed to load');
});

console.log(asyncState.isError); // true (has error, not loading)

// Reactive error handling
effect(() => {
  if (asyncState.isError) {
    showErrorMessage(asyncState.error.message);
  }
});
```

---

## **What is `asyncState.isError`?**

`asyncState.isError` is a **reactive computed property** that returns `true` when an error has occurred (not loading and error exists).

**Key characteristics:**
- **Computed Property**: Automatically updates based on state
- **Reactive**: Triggers effects when value changes
- **Boolean**: Always `true` or `false`
- **Read-Only**: Cannot be set directly
- **Error Indicator**: True only when in error state

---

## **Formula**

```javascript
isError = !loading && error !== null
```

---

## **Computed Implementation**

```javascript
asyncState.$computed('isError', function() {
  return !this.loading && this.error !== null;
});
```

---

## **State Matrix**

| loading | error | isError |
|---------|-------|---------|
| `false` | `null` | `false` (no error) |
| `true` | `null` | `false` (loading) |
| `false` | `Error` | `true` ✓ (error) |
| `true` | `Error` | `false` (retrying) |

---

## **Examples**

### **Example 1: Error Banner**
```javascript
const asyncState = ReactiveUtils.asyncState();

effect(() => {
  const errorBanner = document.querySelector('#error-banner');
  
  if (asyncState.isError) {
    errorBanner.textContent = asyncState.error.message;
    errorBanner.style.display = 'block';
  } else {
    errorBanner.style.display = 'none';
  }
});
```

### **Example 2: Retry Button**
```javascript
const asyncState = ReactiveUtils.asyncState();

effect(() => {
  const retryBtn = document.querySelector('#retry-btn');
  
  if (asyncState.isError) {
    retryBtn.style.display = 'block';
    retryBtn.onclick = () => refetch(asyncState);
  } else {
    retryBtn.style.display = 'none';
  }
});
```

### **Example 3: Error Styling**
```javascript
const asyncState = ReactiveUtils.asyncState();

effect(() => {
  const container = document.querySelector('#container');
  
  if (asyncState.isError) {
    container.classList.add('error-state');
  } else {
    container.classList.remove('error-state');
  }
});
```

### **Example 4: Error Logging**
```javascript
const asyncState = ReactiveUtils.asyncState();

let previousError = false;

effect(() => {
  if (asyncState.isError && !previousError) {
    // Error just occurred
    console.error('Async error:', asyncState.error);
    
    sendErrorToService({
      message: asyncState.error.message,
      stack: asyncState.error.stack,
      timestamp: new Date().toISOString()
    });
    
    previousError = true;
  } else if (!asyncState.isError) {
    previousError = false;
  }
});
```

### **Example 5: Conditional UI**
```javascript
const asyncState = ReactiveUtils.asyncState();

effect(() => {
  if (asyncState.isError) {
    showErrorView(asyncState.error);
  } else if (asyncState.isSuccess) {
    showDataView(asyncState.data);
  } else if (asyncState.loading) {
    showLoadingView();
  } else {
    showEmptyView();
  }
});
```

---

## **Common Patterns**

### **Pattern 1: Error Display**
```javascript
effect(() => {
  if (asyncState.isError) {
    errorEl.textContent = asyncState.error.message;
  }
});
```

### **Pattern 2: Retry Button**
```javascript
effect(() => {
  retryBtn.style.display = asyncState.isError ? 'block' : 'none';
});
```

### **Pattern 3: Error Class**
```javascript
effect(() => {
  container.classList.toggle('error', asyncState.isError);
});
```

---

## **Comparison with Related Properties**

| Property | When `true` |
|----------|-------------|
| `isError` | Has error, not loading |
| `isSuccess` | Data loaded, no error |
| `isIdle` | No data, no error, not loading |
| `error !== null` | Has error (even if loading) |

```javascript
// States
asyncState.error = new Error('Failed');
asyncState.loading = true;

asyncState.isError;      // false (loading)
asyncState.error !== null; // true (has error)

asyncState.loading = false;
asyncState.isError;      // true (error + not loading)
```

---

## **Key Takeaways**

1. **Computed Property**: Auto-updates based on loading and error
2. **True When Error**: Only `true` when error exists and not loading
3. **Read-Only**: Cannot be set directly
4. **Reactive**: Triggers effects when value changes
5. **Common Use**: Error messages, retry buttons, error styling

---

## **Summary**

`asyncState.isError` is a reactive computed property that returns `true` when an error has occurred (not loading and error exists). It automatically updates based on the loading and error properties, making it perfect for displaying error messages, showing retry buttons, applying error styling, and handling error states in your UI.
