# `asyncState.loading` - Async Loading State Property

**Quick Start (30 seconds)**
```javascript
const asyncState = ReactiveUtils.asyncState();

console.log(asyncState.loading); // false

execute(asyncState, fetchData);
console.log(asyncState.loading); // true

// After completion
// asyncState.loading === false

// Reactive loading indicator
effect(() => {
  const spinner = querySelector('#spinner');
  spinner.style.display = asyncState.loading ? 'block' : 'none';
});
```

---

## **What is `asyncState.loading`?**

`asyncState.loading` is a **reactive boolean property** that indicates whether an async operation is currently in progress.

**Key characteristics:**
- **Reactive**: Changes trigger effects
- **Boolean**: Always `true` or `false`
- **Auto-Managed**: Set by `execute()` and `abort()`
- **Initially False**: Starts as `false`
- **Read/Write**: Can be accessed and modified

---

## **Property Type**

```typescript
asyncState.loading: boolean
```

---

## **Lifecycle**

```
1. Create:     loading = false
2. Execute:    loading = true
3. Complete:   loading = false
4. Error:      loading = false
5. Abort:      loading = false
6. Reset:      loading = false
```

---

## **Examples**

### **Example 1: Loading Spinner**
```javascript
const asyncState = ReactiveUtils.asyncState();

effect(() => {
  const spinner = querySelector('#loading-spinner');
  
  if (asyncState.loading) {
    spinner.classList.add('visible');
  } else {
    spinner.classList.remove('visible');
  }
});

await execute(asyncState, fetchData);
```

### **Example 2: Disable Button During Loading**
```javascript
const asyncState = ReactiveUtils.asyncState();

effect(() => {
  const submitBtn = querySelector('#submit-btn');
  
  submitBtn.disabled = asyncState.loading;
  submitBtn.textContent = asyncState.loading ? 'Loading...' : 'Submit';
});
```

### **Example 3: Loading Overlay**
```javascript
const asyncState = ReactiveUtils.asyncState();

effect(() => {
  const overlay = querySelector('#overlay');
  
  if (asyncState.loading) {
    overlay.style.display = 'flex';
    document.body.style.overflow = 'hidden';
  } else {
    overlay.style.display = 'none';
    document.body.style.overflow = 'auto';
  }
});
```

### **Example 4: Multiple States**
```javascript
const usersState = ReactiveUtils.asyncState();
const postsState = ReactiveUtils.asyncState();

effect(() => {
  const isLoading = usersState.loading || postsState.loading;
  
  if (isLoading) {
    showGlobalSpinner();
  } else {
    hideGlobalSpinner();
  }
});
```

### **Example 5: Loading Text**
```javascript
const asyncState = ReactiveUtils.asyncState();

effect(() => {
  const status = querySelector('#status');
  
  if (asyncState.loading) {
    status.textContent = 'Loading data, please wait...';
  } else if (asyncState.error) {
    status.textContent = 'Failed to load';
  } else {
    status.textContent = 'Ready';
  }
});
```

---

## **Common Patterns**

### **Pattern 1: Show/Hide Spinner**
```javascript
effect(() => {
  spinner.style.display = asyncState.loading ? 'block' : 'none';
});
```

### **Pattern 2: Disable Controls**
```javascript
effect(() => {
  button.disabled = asyncState.loading;
});
```

### **Pattern 3: Loading Text**
```javascript
effect(() => {
  button.textContent = asyncState.loading ? 'Loading...' : 'Load Data';
});
```

---

## **Key Takeaways**

1. **Boolean Property**: Always `true` or `false`
2. **Auto-Managed**: Updated by `execute()` and `abort()`
3. **Reactive**: Changes trigger effects
4. **Initially False**: Starts as `false`
5. **Common Use**: Spinners, disabled buttons, loading text

---

## **Summary**

`asyncState.loading` is a reactive boolean property indicating whether an async operation is currently in progress. It's automatically managed by `execute()` (sets to `true`) and updated when operations complete, error, or are aborted (sets to `false`). Use it to show loading spinners, disable buttons, and provide loading feedback to users.
