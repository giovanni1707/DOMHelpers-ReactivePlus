# `DevTools.enable()` - Enable Development Tools

**Quick Start (30 seconds)**
```javascript
// Enable DevTools
ReactiveUtils.DevTools.enable();

console.log('DevTools enabled:', ReactiveUtils.DevTools.enabled); // true

// Now available globally
console.log(window.__REACTIVE_DEVTOOLS__); // DevTools object

// Start tracking
const state = ReactiveUtils.state({ count: 0 });
ReactiveUtils.DevTools.trackState(state, 'Counter');
```

---

## **What is `DevTools.enable()`?**

`DevTools.enable()` is a **method** that enables the development tools and exposes them globally via `window.__REACTIVE_DEVTOOLS__`.

**Key characteristics:**
- **Enables Tracking**: Turns on state and effect tracking
- **Global Exposure**: Creates `window.__REACTIVE_DEVTOOLS__`
- **Console Access**: Allows debugging from browser console
- **Development Mode**: For debugging during development
- **Auto-Enabled**: Already enabled on localhost

---

## **Syntax**

```javascript
ReactiveUtils.DevTools.enable()
```

### **Parameters**
- None

### **Returns**
- **Type**: `void`

---

## **How it works**

```javascript
enable() {
  this.enabled = true;
  window.__REACTIVE_DEVTOOLS__ = this;
  console.log('[DevTools] Enabled - inspect with window.__REACTIVE_DEVTOOLS__');
}
```

**What happens:**
1. Sets `enabled` flag to `true`
2. Exposes DevTools object globally
3. Logs confirmation message
4. State/effect tracking begins working

---

## **Examples**

### **Example 1: Basic Enable**
```javascript
// Enable DevTools
ReactiveUtils.DevTools.enable();

// Check if enabled
console.log(ReactiveUtils.DevTools.enabled); // true

// Access globally
window.__REACTIVE_DEVTOOLS__.getHistory();
```

### **Example 2: Conditional Enable**
```javascript
if (process.env.NODE_ENV === 'development') {
  ReactiveUtils.DevTools.enable();
  console.log('DevTools enabled for development');
}
```

### **Example 3: Enable and Track**
```javascript
// Enable first
ReactiveUtils.DevTools.enable();

// Then create and track states
const state = ReactiveUtils.state({ count: 0 });
ReactiveUtils.DevTools.trackState(state, 'Counter');

state.count++; // Logged to DevTools
```

### **Example 4: Re-Enable After Disable**
```javascript
ReactiveUtils.DevTools.enable();
console.log('Enabled:', ReactiveUtils.DevTools.enabled); // true

ReactiveUtils.DevTools.disable();
console.log('Enabled:', ReactiveUtils.DevTools.enabled); // false

ReactiveUtils.DevTools.enable();
console.log('Enabled:', ReactiveUtils.DevTools.enabled); // true
```

### **Example 5: Environment-Based**
```javascript
// Enable based on environment
const isDev = window.location.hostname === 'localhost';

if (isDev) {
  ReactiveUtils.DevTools.enable();
}
```

### **Example 6: Enable with Logging**
```javascript
function enableDevTools() {
  ReactiveUtils.DevTools.enable();
  
  console.log('✓ DevTools enabled');
  console.log('Access via: window.__REACTIVE_DEVTOOLS__');
  console.log('Methods: getStates(), getHistory(), clearHistory()');
}

enableDevTools();
```

### **Example 7: Enable Early**
```javascript
// Enable before creating any states
ReactiveUtils.DevTools.enable();

// All states created after this will be trackable
const state1 = ReactiveUtils.state({ a: 1 });
const state2 = ReactiveUtils.state({ b: 2 });

ReactiveUtils.DevTools.trackState(state1, 'State1');
ReactiveUtils.DevTools.trackState(state2, 'State2');
```

### **Example 8: Enable with Custom Config**
```javascript
ReactiveUtils.DevTools.enable();

// Customize after enabling
ReactiveUtils.DevTools.maxHistory = 100; // Increase history limit

console.log('DevTools enabled with max history:', 
            ReactiveUtils.DevTools.maxHistory);
```

### **Example 9: Feature Flag**
```javascript
class App {
  constructor() {
    this.features = {
      devTools: true,
      analytics: false
    };
    
    if (this.features.devTools) {
      ReactiveUtils.DevTools.enable();
    }
  }
}
```

### **Example 10: Enable with Persistence**
```javascript
// Save enable state to localStorage
function enableDevToolsPersistent() {
  ReactiveUtils.DevTools.enable();
  localStorage.setItem('devtools_enabled', 'true');
}

// Restore on page load
if (localStorage.getItem('devtools_enabled') === 'true') {
  ReactiveUtils.DevTools.enable();
}
```

---

## **Common Patterns**

### **Pattern 1: Development Mode**
```javascript
if (process.env.NODE_ENV === 'development') {
  ReactiveUtils.DevTools.enable();
}
```

### **Pattern 2: Localhost Only**
```javascript
if (window.location.hostname === 'localhost') {
  ReactiveUtils.DevTools.enable();
}
```

### **Pattern 3: Enable and Track**
```javascript
ReactiveUtils.DevTools.enable();
ReactiveUtils.DevTools.trackState(state, 'MyState');
```

---

## **Auto-Enable**

DevTools automatically enables on localhost:

```javascript
// Already enabled by default on localhost!
if (typeof window !== 'undefined' && 
    (window.location.hostname === 'localhost' || 
     window.location.hostname === '127.0.0.1')) {
  DevTools.enable();
}
```

So you may not need to call `enable()` during local development.

---

## **Before vs After Enable**

| Before Enable | After Enable |
|---------------|--------------|
| `enabled: false` | `enabled: true` |
| `window.__REACTIVE_DEVTOOLS__` is `undefined` | `window.__REACTIVE_DEVTOOLS__` exists |
| Tracking methods do nothing | Tracking methods work |
| No history logged | History is logged |

---

## **Console Access**

After enabling, you can use DevTools from browser console:

```javascript
// Enable
ReactiveUtils.DevTools.enable();

// Now in browser console:
__REACTIVE_DEVTOOLS__.getStates()
__REACTIVE_DEVTOOLS__.getHistory()
__REACTIVE_DEVTOOLS__.clearHistory()
```

---

## **Key Takeaways**

1. **Enables DevTools**: Turns on development tools
2. **Global Exposure**: Creates `window.__REACTIVE_DEVTOOLS__`
3. **Console Access**: Enables debugging from console
4. **Auto-Enabled**: Already enabled on localhost
5. **No Parameters**: Simple method call
6. **Idempotent**: Safe to call multiple times
7. **Development Only**: Use in development, disable in production

---

## **Summary**

`DevTools.enable()` enables the development tools and exposes them globally via `window.__REACTIVE_DEVTOOLS__`, allowing you to debug reactive state from the browser console. It's automatically enabled on localhost, but you can manually enable it in other environments during development. After enabling, all tracking methods become active, and you can access DevTools features through the global object. Always disable DevTools in production for optimal performance.
