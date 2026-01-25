# `builder.effect(fn)` - Add Effect

**Quick Start (30 seconds)**
```javascript
const app = ReactiveUtils.reactive({ count: 0, name: 'John' })
  .effect(() => {
    console.log('Effect runs! Count:', app.state.count);
    console.log('Name:', app.state.name);
  })
  .build();

// Effect runs immediately
// Logs: "Effect runs! Count: 0"
// Logs: "Name: John"

app.count = 5;
// Effect runs again
// Logs: "Effect runs! Count: 5"
// Logs: "Name: John"

app.name = 'Jane';
// Effect runs again
// Logs: "Effect runs! Count: 5"
// Logs: "Name: Jane"
```

---

## **What is `builder.effect(fn)`?**

`builder.effect(fn)` is a **builder method** that adds a reactive effect to the component, automatically re-running the effect function whenever its dependencies (accessed state properties) change.

**Key characteristics:**
- **Auto-Tracking**: Automatically tracks accessed properties
- **Auto-Rerun**: Re-runs when dependencies change
- **Immediate**: Runs immediately when added
- **Chainable**: Returns builder for method chaining
- **Cleanup**: Effects cleaned up on destroy

---

## **Syntax**

```javascript
builder.effect(effectFunction)
```

### **Parameters**
- **`effectFunction`** (Function): Function to run reactively

**Effect function can return cleanup:**
```javascript
.effect(() => {
  // Effect logic

  return () => {
    // Cleanup logic
  };
})
```

### **Returns**
- **Type**: Builder instance (for chaining)

---

## **How it works**

```javascript
builder.effect(fn) {
  cleanups.push(effect(fn));
  return this;
}
```

**What happens:**
1. Adds effect to reactivity system
2. Effect runs immediately
3. Tracks all accessed properties
4. Re-runs when dependencies change
5. Stores cleanup function
6. Returns builder for chaining

---

## **Examples**

### **Example 1: DOM Updates**
```javascript
const counter = ReactiveUtils.reactive({ count: 0 })
  .effect(() => {
    document.getElementById('count').textContent = counter.state.count;
  })
  .build();

counter.count = 5; // DOM automatically updated
```

### **Example 2: Multiple Dependencies**
```javascript
const app = ReactiveUtils.reactive({
  firstName: 'John',
  lastName: 'Doe',
  age: 25
})
.effect(() => {
  const fullName = `${app.state.firstName} ${app.state.lastName}`;
  const info = `${fullName}, ${app.state.age} years old`;
  console.log(info);
})
.build();

// Effect runs on any change
app.firstName = 'Jane'; // Re-runs
app.age = 26; // Re-runs
```

### **Example 3: With Cleanup**
```javascript
const timer = ReactiveUtils.reactive({ interval: 1000 })
  .effect(() => {
    const id = setInterval(() => {
      console.log('Tick');
    }, timer.state.interval);

    // Cleanup function
    return () => clearInterval(id);
  })
  .build();

timer.interval = 2000; // Clears old interval, starts new one
```

### **Example 4: API Sync**
```javascript
const user = ReactiveUtils.reactive({ id: 123 })
  .effect(() => {
    fetch(`/api/users/${user.state.id}`)
      .then(r => r.json())
      .then(data => {
        console.log('User data:', data);
      });
  })
  .build();

user.id = 456; // Fetches new user
```

### **Example 5: Logging**
```javascript
const state = ReactiveUtils.reactive({
  action: 'idle',
  count: 0
})
.effect(() => {
  console.log(`[${new Date().toISOString()}] Action: ${state.state.action}, Count: ${state.state.count}`);
})
.build();

state.action = 'increment';
state.count = 1;
// Logs twice with timestamps
```

### **Example 6: Conditional Effect**
```javascript
const app = ReactiveUtils.reactive({
  debug: true,
  value: 0
})
.effect(() => {
  if (app.state.debug) {
    console.log('Debug: value =', app.state.value);
  }
})
.build();

app.value = 5; // Logs if debug is true
app.debug = false;
app.value = 10; // Doesn't log
```

### **Example 7: Local Storage Sync**
```javascript
const settings = ReactiveUtils.reactive({
  theme: 'light',
  fontSize: 14
})
.effect(() => {
  localStorage.setItem('settings', JSON.stringify({
    theme: settings.state.theme,
    fontSize: settings.state.fontSize
  }));
  console.log('Settings saved');
})
.build();

settings.theme = 'dark'; // Auto-saves
```

### **Example 8: Event Listeners**
```javascript
const app = ReactiveUtils.reactive({ active: true })
  .effect(() => {
    if (!app.state.active) return;

    const handler = () => console.log('Clicked');
    document.addEventListener('click', handler);

    return () => document.removeEventListener('click', handler);
  })
  .build();

app.active = false; // Removes listener
app.active = true; // Adds listener again
```

### **Example 9: Multiple Effects**
```javascript
const app = ReactiveUtils.reactive({ count: 0 })
  .effect(() => {
    console.log('Effect 1:', app.state.count);
  })
  .effect(() => {
    console.log('Effect 2:', app.state.count * 2);
  })
  .effect(() => {
    document.title = `Count: ${app.state.count}`;
  })
  .build();

app.count = 5;
// All three effects run
```

### **Example 10: Async Effect**
```javascript
const search = ReactiveUtils.reactive({
  query: '',
  results: []
})
.effect(async () => {
  if (search.state.query.length < 3) return;

  const response = await fetch(`/api/search?q=${search.state.query}`);
  search.state.results = await response.json();
})
.build();

search.query = 'react'; // Triggers search
```

---

## **Common Patterns**

### **Pattern 1: Simple Effect**
```javascript
.effect(() => {
  console.log(state.count);
})
```

### **Pattern 2: DOM Update**
```javascript
.effect(() => {
  element.textContent = state.value;
})
```

### **Pattern 3: With Cleanup**
```javascript
.effect(() => {
  const id = setInterval(...);
  return () => clearInterval(id);
})
```

### **Pattern 4: Conditional**
```javascript
.effect(() => {
  if (state.condition) {
    // Do something
  }
})
```

### **Pattern 5: Multiple Effects**
```javascript
.effect(() => { /* Effect 1 */ })
.effect(() => { /* Effect 2 */ })
```

---

## **Best Practices**

1. **Keep effects focused**
   ```javascript
   .effect(() => {
     updateDOM();
   })
   ```

2. **Return cleanup functions**
   ```javascript
   .effect(() => {
     const subscription = subscribe();
     return () => subscription.unsubscribe();
   })
   ```

3. **Avoid infinite loops**
   ```javascript
   // Bad
   .effect(() => {
     state.count = state.count + 1; // Infinite loop!
   })
   ```

4. **Use for side effects only**
   ```javascript
   .effect(() => {
     // Good: DOM updates, API calls, logging
     updateDOM(state.value);
   })
   ```

5. **Debounce expensive operations**
   ```javascript
   let timeout;
   .effect(() => {
     clearTimeout(timeout);
     timeout = setTimeout(() => expensiveOp(state.value), 300);
   })
   ```

---

## **Key Takeaways**

1. **Auto-Tracking**: Automatically tracks accessed properties
2. **Reactive**: Re-runs when dependencies change
3. **Immediate**: Runs immediately when added
4. **Cleanup**: Can return cleanup function
5. **Chainable**: Returns builder for chaining
6. **Multiple**: Can add multiple effects
7. **Async**: Can be async functions
8. **Side Effects**: For DOM updates, API calls, etc.
9. **Auto-Cleanup**: Cleaned up on destroy
10. **Flexible**: Tracks any accessed state

---

## **Summary**

`builder.effect(fn)` is a builder method that adds a reactive effect to the component, automatically re-running the effect function whenever its dependencies change. Effects run immediately when added and automatically track all state properties accessed during execution. When those properties change, the effect re-runs with the latest values. Effects can return cleanup functions for managing subscriptions, timers, and event listeners. Use effects for side effects like DOM updates, API calls, logging, and event listener management. The method is chainable and allows multiple effects to be added. Effects are automatically cleaned up when the component is destroyed. Keep effects focused and avoid modifying state within effects to prevent infinite loops.
