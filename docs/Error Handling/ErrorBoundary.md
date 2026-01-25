# `ErrorBoundary` - Error Boundary Class

**Quick Start (30 seconds)**
```javascript
const boundary = new ErrorBoundary({
  onError: (error, context) => {
    console.error('Error in', context.type, ':', error);
    sendToErrorTracking(error);
  },
  retry: true,
  maxRetries: 3
});

// Wrap effect with error boundary
const safeEffect = boundary.wrap(
  () => {
    // Effect code that might throw
    const data = fetchAndProcess();
    updateDOM(data);
  },
  { type: 'effect', name: 'dataFetcher' }
);

effect(safeEffect);
```

---

## **What is `ErrorBoundary`?**

`ErrorBoundary` is a **class** for wrapping functions with error handling, providing automatic retry logic, error callbacks, and fallback values. It prevents one failing operation from crashing your entire application.

**Key characteristics:**
- **Isolated Error Handling**: Catches and handles errors gracefully
- **Automatic Retry**: Optionally retries failed operations
- **Error Callbacks**: Custom error handling logic
- **Fallback Values**: Return fallback when all retries fail
- **Context Tracking**: Track where errors occur

---

## **Syntax**

```javascript
const boundary = new ErrorBoundary(options)
```

### **Parameters**

`options` (optional) - Configuration object:

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `onError` | `function` | Console.error | Error callback function |
| `fallback` | `function` | `undefined` | Fallback value function |
| `retry` | `boolean` | `true` | Enable automatic retry |
| `maxRetries` | `number` | `3` | Maximum retry attempts |
| `retryDelay` | `number` | `0` | Delay between retries (ms) |

### **Returns**
- **Type**: `ErrorBoundary` instance

---

## **Methods**

### `boundary.wrap(fn, context)`

Wraps a function with error handling:

```javascript
const wrappedFn = boundary.wrap(fn, context)
```

**Parameters:**
- `fn` - Function to wrap
- `context` - Context object with metadata

**Returns:** Wrapped function

---

## **How it works**

```javascript
class ErrorBoundary {
  constructor(options = {}) {
    this.onError = options.onError || defaultErrorHandler;
    this.fallback = options.fallback;
    this.retry = options.retry !== false;
    this.maxRetries = options.maxRetries || 3;
    this.retryDelay = options.retryDelay || 0;
  }
  
  wrap(fn, context = {}) {
    let retries = 0;
    
    return (...args) => {
      const attempt = () => {
        try {
          return fn(...args);
        } catch (error) {
          retries++;
          
          const shouldRetry = this.retry && retries < this.maxRetries;
          
          this.onError(error, {
            ...context,
            attempt: retries,
            maxRetries: this.maxRetries,
            willRetry: shouldRetry
          });
          
          if (shouldRetry) {
            if (this.retryDelay > 0) {
              setTimeout(attempt, this.retryDelay);
            } else {
              return attempt();
            }
          } else if (this.fallback) {
            return this.fallback(error, context);
          }
          
          return undefined;
        }
      };
      
      return attempt();
    };
  }
}
```

---

## **Examples**

### **Example 1: Basic Error Handling**
```javascript
const boundary = new ErrorBoundary({
  onError: (error, context) => {
    console.error(`[${context.type}] Error:`, error.message);
  }
});

const safeFunction = boundary.wrap(
  () => {
    throw new Error('Something failed');
  },
  { type: 'computation' }
);

safeFunction(); // Logs error, doesn't crash
```

### **Example 2: With Retry**
```javascript
let attempt = 0;

const boundary = new ErrorBoundary({
  retry: true,
  maxRetries: 3,
  retryDelay: 1000,
  onError: (error, context) => {
    console.log(`Attempt ${context.attempt}/${context.maxRetries}`);
  }
});

const unstableFunction = boundary.wrap(
  () => {
    attempt++;
    if (attempt < 3) {
      throw new Error('Temporary failure');
    }
    return 'Success!';
  },
  { type: 'api-call' }
);

// Retries twice, succeeds on third attempt
const result = unstableFunction();
```

### **Example 3: With Fallback**
```javascript
const boundary = new ErrorBoundary({
  retry: false,
  fallback: (error, context) => {
    console.log('Using fallback value');
    return { data: [], error: error.message };
  }
});

const fetchData = boundary.wrap(
  () => {
    throw new Error('Network error');
  },
  { type: 'data-fetch' }
);

const result = fetchData();
console.log(result); // { data: [], error: 'Network error' }
```

### **Example 4: Safe Effects**
```javascript
const boundary = new ErrorBoundary({
  onError: (error, context) => {
    sendToErrorService({
      error: error.message,
      stack: error.stack,
      context
    });
  }
});

// Wrap effect to prevent crashes
effect(boundary.wrap(
  () => {
    const data = state.riskyData;
    processAndRender(data);
  },
  { type: 'effect', component: 'DataView' }
));
```

### **Example 5: API Call with Retry**
```javascript
const apiBoundary = new ErrorBoundary({
  retry: true,
  maxRetries: 5,
  retryDelay: 2000,
  onError: (error, context) => {
    if (context.attempt < context.maxRetries) {
      showNotification(`Retrying... (${context.attempt}/${context.maxRetries})`);
    } else {
      showNotification('Failed after all retries', 'error');
    }
  },
  fallback: () => ({
    success: false,
    data: null,
    error: 'Service unavailable'
  })
});

const safeApiCall = apiBoundary.wrap(
  async () => {
    const response = await fetch('/api/data');
    if (!response.ok) throw new Error('API error');
    return response.json();
  },
  { type: 'api', endpoint: '/api/data' }
);

const result = await safeApiCall();
```

### **Example 6: Form Validation**
```javascript
const validationBoundary = new ErrorBoundary({
  retry: false,
  fallback: (error) => ({
    valid: false,
    errors: [error.message]
  })
});

const validateForm = validationBoundary.wrap(
  (formData) => {
    if (!formData.email) throw new Error('Email required');
    if (!formData.email.includes('@')) throw new Error('Invalid email');
    if (!formData.password) throw new Error('Password required');
    
    return { valid: true, errors: [] };
  },
  { type: 'validation' }
);

const result = validateForm({ email: '', password: '' });
console.log(result); // { valid: false, errors: ['Email required'] }
```

### **Example 7: Multiple Boundaries**
```javascript
// Critical operations - no retry
const criticalBoundary = new ErrorBoundary({
  retry: false,
  onError: (error) => {
    sendUrgentAlert(error);
  }
});

// Non-critical - retry with fallback
const softBoundary = new ErrorBoundary({
  retry: true,
  maxRetries: 3,
  fallback: () => 'N/A'
});

const criticalOp = criticalBoundary.wrap(savePayment, { type: 'payment' });
const softOp = softBoundary.wrap(loadRecommendations, { type: 'recommendations' });
```

### **Example 8: Error Context Tracking**
```javascript
const boundary = new ErrorBoundary({
  onError: (error, context) => {
    const errorReport = {
      message: error.message,
      stack: error.stack,
      context: {
        type: context.type,
        component: context.component,
        timestamp: new Date().toISOString(),
        userAgent: navigator.userAgent
      }
    };
    
    sendToSentry(errorReport);
  }
});

const trackingWrapper = (fn, componentName) => {
  return boundary.wrap(fn, {
    type: 'component',
    component: componentName,
    created: Date.now()
  });
};
```

### **Example 9: Conditional Retry**
```javascript
const smartBoundary = new ErrorBoundary({
  retry: true,
  maxRetries: 3,
  onError: (error, context) => {
    // Only retry on network errors
    if (error.message.includes('network')) {
      return true; // Continue retry
    }
    
    // Don't retry validation errors
    context.willRetry = false;
    showError(error.message);
  }
});
```

### **Example 10: Global Error Boundary**
```javascript
const globalBoundary = new ErrorBoundary({
  retry: false,
  onError: (error, context) => {
    // Log to console
    console.error('[Global Error]', error);
    
    // Send to tracking
    trackError(error, context);
    
    // Show user notification
    showErrorToast(error.message);
    
    // Update error state
    errorState.error = error;
    errorState.context = context;
  },
  fallback: (error) => {
    return {
      __error: true,
      message: error.message,
      timestamp: Date.now()
    };
  }
});

// Use globally
ReactiveUtils.ErrorBoundary = globalBoundary;
```

---

## **Common Patterns**

### **Pattern 1: Basic Wrap**
```javascript
const boundary = new ErrorBoundary();
const safe = boundary.wrap(riskyFunction, { type: 'computation' });
```

### **Pattern 2: API with Retry**
```javascript
const boundary = new ErrorBoundary({
  retry: true,
  maxRetries: 3,
  retryDelay: 1000
});
const safeAPI = boundary.wrap(apiCall, { type: 'api' });
```

### **Pattern 3: With Fallback**
```javascript
const boundary = new ErrorBoundary({
  fallback: () => defaultValue
});
const safeGet = boundary.wrap(getter, { type: 'getter' });
```

---

## **Integration with ReactiveUtils**

```javascript
// Use built-in safeEffect and safeWatch
import { safeEffect, safeWatch, ErrorBoundary } from 'ReactiveUtils';

// Or create custom boundary
const myBoundary = new ErrorBoundary({
  onError: customErrorHandler
});
```

---

## **Key Takeaways**

1. **Prevents Crashes**: Isolates errors to prevent app-wide failures
2. **Automatic Retry**: Configurable retry logic for transient errors
3. **Fallback Values**: Provide defaults when operations fail
4. **Error Tracking**: Custom error handlers for logging/monitoring
5. **Context Aware**: Track where and why errors occur
6. **Flexible**: Configurable retry, delay, fallback behavior
7. **Reusable**: Create once, wrap multiple functions
8. **Common Use**: Effects, API calls, validation, computation

---

## **Summary**

`ErrorBoundary` is a class for wrapping functions with error handling, automatic retry logic, and fallback values. It prevents one failing operation from crashing your entire application by catching errors, optionally retrying operations, calling custom error handlers, and providing fallback values. Create an error boundary with configuration options, then use the `wrap()` method to protect any function from uncaught errors while maintaining full control over error handling behavior.
