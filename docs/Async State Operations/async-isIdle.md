# `asyncState.isIdle` - Idle State Computed Property

**Quick Start (30 seconds)**
```javascript
const asyncState = ReactiveUtils.asyncState();

console.log(asyncState.isIdle); // true (initial state)

execute(asyncState, fetchData);
console.log(asyncState.isIdle); // false (loading started)

// After completion
// asyncState.isIdle === false (has data)

// Reactive idle state handling
effect(() => {
  if (asyncState.isIdle) {
    console.log('Waiting for first load...');
  }
});
```

---

## **What is `asyncState.isIdle`?**

`asyncState.isIdle` is a **reactive computed property** that returns `true` when the async state is in its initial idle state (no data, no error, not loading).

**Key characteristics:**
- **Computed Property**: Automatically updates based on state
- **Reactive**: Triggers effects when value changes
- **Boolean**: Always `true` or `false`
- **Read-Only**: Cannot be set directly
- **Initial State**: True only in initial/reset state

---

## **Formula**

```javascript
isIdle = !loading && data === null && error === null
```

---

## **Computed Implementation**

```javascript
asyncState.$computed('isIdle', function() {
  return !this.loading && this.data === null && this.error === null;
});
```

---

## **State Matrix**

| loading | data | error | isIdle |
|---------|------|-------|--------|
| `false` | `null` | `null` | `true` ✓ (idle) |
| `true` | `null` | `null` | `false` (loading) |
| `false` | `data` | `null` | `false` (has data) |
| `false` | `null` | `Error` | `false` (has error) |

---

## **Lifecycle**

```
Create:    isIdle = true  (initial state)
Execute:   isIdle = false (loading started)
Success:   isIdle = false (has data)
Error:     isIdle = false (has error)
Reset:     isIdle = true  (back to initial)
```

---

## **Examples**

### **Example 1: Initial State Message**
```javascript
const asyncState = ReactiveUtils.asyncState();

effect(() => {
  const message = querySelector('#message');
  
  if (asyncState.isIdle) {
    message.textContent = 'Click "Load Data" to begin';
  }
});
```

### **Example 2: Conditional Rendering**
```javascript
const asyncState = ReactiveUtils.asyncState();

effect(() => {
  if (asyncState.isIdle) {
    showEmptyState();
  } else if (asyncState.loading) {
    showLoadingState();
  } else if (asyncState.isError) {
    showErrorState(asyncState.error);
  } else if (asyncState.isSuccess) {
    showDataState(asyncState.data);
  }
});
```

### **Example 3: Load Button**
```javascript
const asyncState = ReactiveUtils.asyncState();

effect(() => {
  const loadBtn = querySelector('#load-btn');
  
  if (asyncState.isIdle) {
    loadBtn.textContent = 'Load Data';
    loadBtn.className = 'primary';
  } else {
    loadBtn.textContent = 'Reload Data';
    loadBtn.className = 'secondary';
  }
});
```

### **Example 4: First Load vs Refresh**
```javascript
const asyncState = ReactiveUtils.asyncState();

async function loadData() {
  const isFirstLoad = asyncState.isIdle;
  
  await execute(asyncState, fetchData);
  
  if (isFirstLoad) {
    console.log('Initial data loaded');
    showWelcomeMessage();
  } else {
    console.log('Data refreshed');
    showRefreshNotification();
  }
}
```

### **Example 5: Analytics**
```javascript
const asyncState = ReactiveUtils.asyncState();

effect(() => {
  if (!asyncState.isIdle && asyncState.requestId === 1) {
    // First load completed
    sendAnalytics('first_data_load', {
      success: asyncState.isSuccess,
      error: asyncState.error?.message
    });
  }
});
```

---

## **Common Patterns**

### **Pattern 1: Empty State**
```javascript
effect(() => {
  if (asyncState.isIdle) {
    showEmptyState();
  }
});
```

### **Pattern 2: Initial Message**
```javascript
effect(() => {
  message.textContent = asyncState.isIdle 
    ? 'No data loaded yet' 
    : 'Data loaded';
});
```

### **Pattern 3: First Load Check**
```javascript
if (asyncState.isIdle) {
  // First load
} else {
  // Refresh/reload
}
```

---

## **State Transitions**

```
Initial State (isIdle: true)
    ↓
User clicks load
    ↓
Execute starts (isIdle: false, loading: true)
    ↓
Request completes
    ↓
Success (isIdle: false, isSuccess: true)
OR
Error (isIdle: false, isError: true)
    ↓
User clicks reset
    ↓
Back to idle (isIdle: true)
```

---

## **Comparison with Related Properties**

| State | isIdle | loading | isSuccess | isError |
|-------|--------|---------|-----------|---------|
| Initial | `true` | `false` | `false` | `false` |
| Loading | `false` | `true` | `false` | `false` |
| Success | `false` | `false` | `true` | `false` |
| Error | `false` | `false` | `false` | `true` |

---

## **When to Use**

Use `isIdle` when you need to:

- Show empty/initial state message
- Distinguish first load from refresh
- Display "Get Started" instructions
- Hide features until first load
- Track analytics for first load

---

## **Key Takeaways**

1. **Computed Property**: Auto-updates based on loading, data, error
2. **True in Initial State**: Only `true` when never loaded or after reset
3. **Read-Only**: Cannot be set directly
4. **Reactive**: Triggers effects when value changes
5. **Common Use**: Empty states, initial messages, first load detection

---

## **Summary**

`asyncState.isIdle` is a reactive computed property that returns `true` when the async state is in its initial idle state (no data, no error, not loading). It automatically updates based on the loading, data, and error properties, making it perfect for showing empty states, initial messages, distinguishing first loads from refreshes, and providing "get started" instructions to users.
