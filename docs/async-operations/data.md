# `asyncState.data` - Async Data Property

**Quick Start (30 seconds)**
```javascript
const asyncState = ReactiveUtils.asyncState(null);

console.log(asyncState.data); // null (initial value)

await execute(asyncState, async () => {
  const response = await fetch('/api/users');
  return response.json();
});

console.log(asyncState.data); // [{ id: 1, name: 'John' }, ...]

// Reactive access
effect(() => {
  if (asyncState.data) {
    console.log('Data loaded:', asyncState.data);
  }
});
```

---

## **What is `asyncState.data`?**

`asyncState.data` is a **reactive property** that contains the result of a successfully completed async operation. It starts with the initial value and is updated when `execute()` completes successfully.

**Key characteristics:**
- **Reactive**: Changes trigger effects
- **Initially Set**: Starts with initial value from `asyncState(initialValue)`
- **Updated on Success**: Set when `execute()` succeeds
- **Persists**: Remains after loading completes
- **Read/Write**: Can be accessed and modified directly

---

## **Property Type**

```typescript
asyncState.data: any
```

**Initial Value**: Set when creating async state
```javascript
const asyncState = ReactiveUtils.asyncState(initialValue);
```

---

## **Lifecycle**

```
1. Create:    asyncState.data = initialValue
2. Execute:   asyncState.data = initialValue (unchanged during loading)
3. Success:   asyncState.data = result
4. Error:     asyncState.data = unchanged (previous value)
5. Reset:     asyncState.data = initialValue
```

---

## **Examples**

### **Example 1: Display Data**
```javascript
const asyncState = ReactiveUtils.asyncState(null);

effect(() => {
  const container = querySelector('#data-container');
  
  if (asyncState.loading) {
    container.innerHTML = '<p>Loading...</p>';
  } else if (asyncState.error) {
    container.innerHTML = `<p>Error: ${asyncState.error.message}</p>`;
  } else if (asyncState.data) {
    container.innerHTML = asyncState.data
      .map(item => `<div>${item.name}</div>`)
      .join('');
  }
});

await execute(asyncState, fetchData);
```

### **Example 2: Computed Values**
```javascript
const asyncState = ReactiveUtils.asyncState(null);

const itemCount = computed(() => {
  return asyncState.data ? asyncState.data.length : 0;
});

effect(() => {
  console.log(`Total items: ${itemCount()}`);
});
```

### **Example 3: Conditional Rendering**
```javascript
const asyncState = ReactiveUtils.asyncState(null);

effect(() => {
  const emptyState = querySelector('#empty-state');
  const dataView = querySelector('#data-view');
  
  if (!asyncState.data || asyncState.data.length === 0) {
    emptyState.style.display = 'block';
    dataView.style.display = 'none';
  } else {
    emptyState.style.display = 'none';
    dataView.style.display = 'block';
    renderData(asyncState.data);
  }
});
```

### **Example 4: Transform Data**
```javascript
const asyncState = ReactiveUtils.asyncState(null);

await execute(asyncState, fetchRawData);

// Transform if needed
if (asyncState.data) {
  asyncState.data = asyncState.data.map(item => ({
    ...item,
    displayName: `${item.firstName} ${item.lastName}`
  }));
}
```

### **Example 5: Paginated Data**
```javascript
const asyncState = ReactiveUtils.asyncState([]);

async function loadMore() {
  await execute(asyncState, async () => {
    const response = await fetch(`/api/items?page=${page}`);
    const newItems = await response.json();
    
    // Append to existing data
    return [...asyncState.data, ...newItems];
  });
}
```

---

## **Common Patterns**

### **Pattern 1: Check if Data Exists**
```javascript
if (asyncState.data) {
  renderData(asyncState.data);
}
```

### **Pattern 2: Data Length Check**
```javascript
if (asyncState.data && asyncState.data.length > 0) {
  showData();
} else {
  showEmptyState();
}
```

### **Pattern 3: Reactive Display**
```javascript
effect(() => {
  if (asyncState.data) {
    updateUI(asyncState.data);
  }
});
```

---

## **Key Takeaways**

1. **Reactive Property**: Changes trigger effects
2. **Initially Set**: Starts with initial value
3. **Updated on Success**: Set when `execute()` succeeds
4. **Unchanged on Error**: Keeps previous value if request fails
5. **Common Use**: Display results, conditional rendering, computed values

---

## **Summary**

`asyncState.data` is a reactive property containing the result of a successfully completed async operation. It starts with the initial value provided when creating the async state and is updated when `execute()` completes successfully. The property remains unchanged if an error occurs, allowing you to keep displaying previous data while showing error messages.
