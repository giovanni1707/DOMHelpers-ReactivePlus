# `errorBoundary.wrap()` - Wrap Function with Error Handling

**Quick Start (30 seconds)**
```javascript
const boundary = new ErrorBoundary({
  onError: (error, context) => {
    console.error(`Error in ${context.type}:`, error);
  }
});

// Wrap a risky function
const safeFunction = boundary.wrap(
  () => {
    // Code that might throw
    return dangerousOperation();
  },
  { type: 'operation', name: 'getData' }
);

// Call safely - won't crash if it throws
const result = safeFunction();
```

---

## **What is `errorBoundary.wrap()`?**

`errorBoundary.wrap()` is a **method** that wraps a function with error handling, retry logic, and fallback behavior defined by the error boundary.

**Key characteristics:**
- **Returns Wrapped Function**: Safe version of original function
- **Same Signature**: Accepts same arguments as original
- **Error Protection**: Catches and handles errors
- **Automatic Retry**: Uses boundary's retry settings
- **Context Tracking**: Attaches context to errors

---

## **Syntax**

```javascript
const wrappedFn = boundary.wrap(fn, context)
```

### **Parameters**
- `fn` (required) - Function to wrap
  - **Type**: `Function`
  - **Can be sync or async**
- `context` (optional) - Context object for error tracking
  - **Type**: `Object`
  - **Common fields**: `type`, `name`, `component`, etc.

### **Returns**
- **Type**: `Function` (wrapped version)
- **Signature**: Same as original function
- **Behavior**: Calls original, handles errors

---

## **Context Object**

The context object can include any metadata:

```javascript
{
  type: 'effect' | 'api' | 'computation' | 'validation' | ...,
  name: 'string',
  component: 'string',
  created: timestamp,
  // ... any custom fields
}
```

This context is passed to `onError` callback with additional fields:
- `attempt` - Current retry attempt number
- `maxRetries` - Maximum retries allowed
- `willRetry` - Whether retry will happen

---

## **Examples**

### **Example 1: Wrap Effect**
```javascript
const boundary = new ErrorBoundary();

const safeEffect = boundary.wrap(
  () => {
    const data = fetchData();
    updateUI(data);
  },
  { type: 'effect', name: 'dataUpdater' }
);

effect(safeEffect);
```

### **Example 2: Wrap API Call**
```javascript
const boundary = new ErrorBoundary({
  retry: true,
  maxRetries: 3
});

const safeLoadUsers = boundary.wrap(
  async () => {
    const response = await fetch('/api/users');
    return response.json();
  },
  { type: 'api', endpoint: '/api/users' }
);

const users = await safeLoadUsers();
```

### **Example 3: With Arguments**
```javascript
const boundary = new ErrorBoundary();

const safeCalculate = boundary.wrap(
  (a, b, operation) => {
    if (operation === 'divide' && b === 0) {
      throw new Error('Division by zero');
    }
    return operation === 'add' ? a + b : a / b;
  },
  { type: 'calculation' }
);

const result = safeCalculate(10, 2, 'divide'); // 5
const error = safeCalculate(10, 0, 'divide');  // Handled gracefully
```

### **Example 4: Component Method**
```javascript
class DataComponent {
  constructor() {
    this.boundary = new ErrorBoundary({
      onError: (error, context) => {
        this.showError(error, context.method);
      }
    });
    
    // Wrap methods
    this.loadData = this.boundary.wrap(
      this.loadData.bind(this),
      { type: 'component', method: 'loadData' }
    );
  }
  
  loadData() {
    // Original implementation
  }
}
```

### **Example 5: Multiple Wrapped Functions**
```javascript
const boundary = new ErrorBoundary({
  fallback: (error, context) => {
    console.log(`Fallback for ${context.name}`);
    return null;
  }
});

const safeParser = boundary.wrap(
  (json) => JSON.parse(json),
  { type: 'parser', name: 'jsonParser' }
);

const safeGetter = boundary.wrap(
  (obj, path) => path.split('.').reduce((o, k) => o[k], obj),
  { type: 'getter', name: 'pathGetter' }
);

const safeTransform = boundary.wrap(
  (data) => complexTransformation(data),
  { type: 'transformer', name: 'dataTransformer' }
);
```

### **Example 6: Event Handler**
```javascript
const boundary = new ErrorBoundary({
  onError: (error, context) => {
    showToast(`Error in ${context.eventType}: ${error.message}`);
  }
});

const button = querySelector('#btn');

button.addEventListener('click', boundary.wrap(
  (event) => {
    processClick(event);
    updateState();
  },
  { type: 'event', eventType: 'click', element: 'button' }
));
```

### **Example 7: Validation Chain**
```javascript
const boundary = new ErrorBoundary({
  retry: false,
  fallback: (error, context) => ({
    valid: false,
    field: context.field,
    error: error.message
  })
});

const validators = {
  email: boundary.wrap(
    (value) => {
      if (!value.includes('@')) throw new Error('Invalid email');
      return { valid: true };
    },
    { type: 'validation', field: 'email' }
  ),
  
  password: boundary.wrap(
    (value) => {
      if (value.length < 8) throw new Error('Password too short');
      return { valid: true };
    },
    { type: 'validation', field: 'password' }
  )
};
```

### **Example 8: Async with Context**
```javascript
const boundary = new ErrorBoundary({
  retry: true,
  maxRetries: 3,
  retryDelay: 1000
});

async function loadUserData(userId) {
  const safeLoad = boundary.wrap(
    async (id) => {
      const response = await fetch(`/api/users/${id}`);
      if (!response.ok) throw new Error('Failed to load');
      return response.json();
    },
    { type: 'api', resource: 'user', userId }
  );
  
  return await safeLoad(userId);
}
```

### **Example 9: Preserve `this` Context**
```javascript
class Form {
  constructor() {
    this.boundary = new ErrorBoundary();
    this.data = {};
  }
  
  submit() {
    const safeSubmit = this.boundary.wrap(
      function() {
        // `this` refers to Form instance
        this.validate();
        this.send();
      }.bind(this),
      { type: 'form', action: 'submit' }
    );
    
    safeSubmit();
  }
}
```

### **Example 10: Dynamic Context**
```javascript
const boundary = new ErrorBoundary({
  onError: (error, context) => {
    logError({
      ...context,
      url: window.location.href,
      timestamp: new Date().toISOString()
    });
  }
});

function createSafeHandler(handlerName) {
  return boundary.wrap(
    handlers[handlerName],
    {
      type: 'handler',
      name: handlerName,
      created: Date.now()
    }
  );
}
```

---

## **Common Patterns**

### **Pattern 1: Wrap Effect**
```javascript
effect(boundary.wrap(effectFn, { type: 'effect' }));
```

### **Pattern 2: Wrap API Call**
```javascript
const safeAPI = boundary.wrap(apiCall, { type: 'api' });
```

### **Pattern 3: Wrap Event Handler**
```javascript
el.addEventListener('click', boundary.wrap(handler, { type: 'event' }));
```

### **Pattern 4: Wrap Method**
```javascript
this.method = boundary.wrap(this.method.bind(this), { type: 'method' });
```

---

## **Return Value Behavior**

| Scenario | Return Value |
|----------|--------------|
| Success | Original function's return value |
| Error + Retry Success | Return value from successful retry |
| Error + Fallback | Fallback function's return value |
| Error + No Fallback | `undefined` |

---

## **Key Takeaways**

1. **Returns Function**: Creates wrapped version, doesn't execute
2. **Same Signature**: Wrapped function accepts same arguments
3. **Sync or Async**: Works with both synchronous and asynchronous functions
4. **Context Tracking**: Attach metadata for error identification
5. **Preserves Behavior**: Return values and `this` context preserved
6. **Error Protection**: Prevents uncaught errors from crashing app
7. **Common Use**: Effects, event handlers, API calls, validation
8. **Composable**: Can wrap already wrapped functions

---

## **Summary**

`errorBoundary.wrap()` wraps a function with error handling, retry logic, and fallback behavior. It returns a new function with the same signature that safely executes the original function, catching errors and handling them according to the boundary's configuration. The optional context parameter allows attaching metadata for error tracking. Use it to protect any function from uncaught errors while maintaining full control over error handling behavior.
