# `builder.destroy()` - Clean Up Before Build

**Quick Start (30 seconds)**
```javascript
const builder = ReactiveUtils.reactive({ count: 0 })
  .effect(() => {
    console.log('Effect running:', builder.state.count);
  })
  .watch({
    count: (newVal) => console.log('Count:', newVal)
  });

// Trigger effect and watcher
builder.state.count = 5;
// Logs: "Effect running: 5"
// Logs: "Count: 5"

// Destroy BEFORE building
builder.destroy();

// Changes no longer trigger effects
builder.state.count = 10;
// No logs

// Can still build (but effects are gone)
const app = builder.build();
```

---

## **What is `builder.destroy()`?**

`builder.destroy()` is a **builder method** that cleans up all effects, watchers, and bindings registered on the builder BEFORE calling `build()`.

**Key characteristics:**
- **Pre-Build Cleanup**: Cleans up before building
- **Stops Effects**: Stops all effects and watchers
- **Not Common**: Rarely used in normal scenarios
- **Different**: Not the same as `state.destroy()`
- **Builder Method**: Called on builder, not built state

---

## **Syntax**

```javascript
builder.destroy()
```

### **Parameters**
- None

### **Returns**
- **Type**: `void`

---

## **How it works**

```javascript
builder.destroy() {
  cleanups.forEach(c => c());
}
```

**What happens:**
1. Executes all cleanup functions
2. Stops all effects, watchers, bindings
3. State remains reactive
4. Can still call `build()` later

---

## **When to Use**

| Scenario | Use builder.destroy() |
|----------|----------------------|
| Normal component lifecycle | ✗ No (use state.destroy()) |
| Before discarding builder | ✓ Maybe |
| Rebuilding component | ✓ Maybe |
| Testing/debugging | ✓ Yes |
| Error recovery | ✓ Maybe |

---

## **Examples**

### **Example 1: Basic Cleanup**
```javascript
const builder = ReactiveUtils.reactive({ count: 0 })
  .effect(() => {
    console.log('Count:', builder.state.count);
  });

builder.state.count = 5; // Logs

builder.destroy(); // Stop effect

builder.state.count = 10; // Doesn't log
```

### **Example 2: Conditional Building**
```javascript
const builder = ReactiveUtils.reactive({ debug: false })
  .effect(() => {
    if (builder.state.debug) {
      console.log('Debug mode');
    }
  });

// Decide not to use this builder
if (someCondition) {
  builder.destroy(); // Clean up
  // Don't build
} else {
  const app = builder.build();
}
```

### **Example 3: Error Recovery**
```javascript
const builder = ReactiveUtils.reactive({ value: 0 });

try {
  builder.effect(() => {
    if (builder.state.value < 0) {
      throw new Error('Value must be positive');
    }
  });

  builder.state.value = -5; // Throws error
} catch (error) {
  console.error('Setup failed:', error);
  builder.destroy(); // Clean up failed setup
}
```

### **Example 4: Testing Setup**
```javascript
function setupTest() {
  const builder = ReactiveUtils.reactive({ count: 0 })
    .effect(() => {
      console.log('Test effect');
    });

  // Test something

  // Clean up test setup
  builder.destroy();

  return builder; // Can still build if needed
}
```

### **Example 5: Rebuilding Component**
```javascript
let builder = ReactiveUtils.reactive({ count: 0 })
  .effect(() => {
    console.log('Effect 1');
  });

// Destroy and recreate
function rebuild() {
  builder.destroy();

  builder = ReactiveUtils.reactive({ count: 0 })
    .effect(() => {
      console.log('Effect 2');
    });

  return builder.build();
}
```

---

## **builder.destroy() vs state.destroy()**

| Aspect | builder.destroy() | state.destroy() |
|--------|------------------|-----------------|
| When | Before build() | After build() |
| Target | Builder's effects | Built component's effects |
| Usage | Rare | Common |
| Purpose | Cancel setup | Unmount component |
| Typical | Testing/errors | Production |

```javascript
// builder.destroy() - Before build
const builder = ReactiveUtils.reactive({ count: 0 })
  .effect(() => console.log('Effect'));

builder.destroy(); // Clean up builder

// state.destroy() - After build
const app = ReactiveUtils.reactive({ count: 0 })
  .effect(() => console.log('Effect'))
  .build();

app.destroy(); // Clean up component
```

---

## **Common Patterns**

### **Pattern 1: Error Recovery**
```javascript
try {
  builder.effect(/* setup */);
} catch (error) {
  builder.destroy();
  throw error;
}
```

### **Pattern 2: Conditional Setup**
```javascript
if (!shouldBuild) {
  builder.destroy();
  return null;
}

return builder.build();
```

### **Pattern 3: Test Cleanup**
```javascript
afterEach(() => {
  builder.destroy();
});
```

---

## **Best Practices**

1. **Prefer state.destroy()**
   ```javascript
   // Normal use - after build
   const app = builder.build();
   app.destroy();

   // Not builder.destroy()
   ```

2. **Use for setup errors**
   ```javascript
   try {
     builder.effect(/* ... */);
   } catch (error) {
     builder.destroy();
   }
   ```

3. **Don't call after build**
   ```javascript
   const app = builder.build();
   // builder.destroy(); // Don't do this
   app.destroy(); // Do this instead
   ```

4. **Understand the difference**
   ```javascript
   // Before build - cleanup builder
   builder.destroy();

   // After build - cleanup component
   app.destroy();
   ```

---

## **Key Takeaways**

1. **Pre-Build**: Cleans up before calling `build()`
2. **Stops Effects**: All effects and watchers stopped
3. **Rare**: Rarely used in normal code
4. **Not Same**: Different from `state.destroy()`
5. **Builder Method**: Called on builder
6. **State Remains**: State object still reactive
7. **Can Build**: Can still call `build()` after
8. **Use Cases**: Errors, testing, conditional setup
9. **Prefer state.destroy()**: For normal cleanup
10. **Understand When**: Know when to use each

---

## **Summary**

`builder.destroy()` is a builder method that cleans up all effects, watchers, and bindings registered on the builder BEFORE the component is built. It stops all reactive behaviors but leaves the state object intact and reactive. This method is rarely used in normal application code - most cleanup should use `state.destroy()` after building. Use `builder.destroy()` for error recovery during setup, conditional component creation, or test cleanup. The key difference is timing: `builder.destroy()` is for pre-build cleanup while `state.destroy()` (returned by `build()`) is for post-build component unmounting. In most cases, you should prefer building the component and using `state.destroy()` when cleanup is needed.
