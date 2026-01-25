# `builder.action(name, fn)` - Add Single Action

**Quick Start (30 seconds)**
```javascript
const counter = ReactiveUtils.reactive({ count: 0 })
  .action('increment', function() {
    this.count++;
  })
  .action('decrement', function() {
    this.count--;
  })
  .action('reset', function() {
    this.count = 0;
  })
  .build();

counter.increment(); // count becomes 1
counter.increment(); // count becomes 2
counter.decrement(); // count becomes 1
counter.reset(); // count becomes 0
```

---

## **What is `builder.action(name, fn)`?**

`builder.action(name, fn)` is a **builder method** that adds a single action (method) to the reactive state.

**Key characteristics:**
- **Method Definition**: Adds methods to state object
- **Context Bound**: `this` refers to state
- **Chainable**: Returns builder for method chaining
- **Multiple**: Can call multiple times
- **State Mutation**: Actions can modify state

---

## **Syntax**

```javascript
builder.action(name, actionFunction)
```

### **Parameters**
- **`name`** (String): Name of the action/method
- **`actionFunction`** (Function): Function to execute (receives state as `this`)

### **Returns**
- **Type**: Builder instance (for chaining)

---

## **How it works**

```javascript
builder.action(name, fn) {
  state[name] = function(...args) { return fn(this, ...args); };
  return this;
}
```

---

## **Examples**

### **Example 1: Counter Actions**
```javascript
const counter = ReactiveUtils.reactive({ count: 0 })
  .action('increment', function() {
    this.count++;
  })
  .action('add', function(amount) {
    this.count += amount;
  })
  .build();

counter.increment(); // 1
counter.add(5); // 6
```

### **Example 2: Todo Actions**
```javascript
const todos = ReactiveUtils.reactive({ items: [] })
  .action('add', function(text) {
    this.items.push({ id: Date.now(), text, done: false });
  })
  .action('toggle', function(id) {
    const todo = this.items.find(t => t.id === id);
    if (todo) todo.done = !todo.done;
  })
  .action('remove', function(id) {
    const index = this.items.findIndex(t => t.id === id);
    if (index !== -1) this.items.splice(index, 1);
  })
  .build();

todos.add('Learn reactive programming');
todos.add('Build something cool');
```

### **Example 3: Form Actions**
```javascript
const form = ReactiveUtils.reactive({
  values: {},
  errors: {}
})
.action('setField', function(name, value) {
  this.values[name] = value;
})
.action('setError', function(name, error) {
  this.errors[name] = error;
})
.action('reset', function() {
  this.values = {};
  this.errors = {};
})
.build();
```

### **Example 4: Async Action**
```javascript
const user = ReactiveUtils.reactive({ data: null, loading: false })
  .action('fetch', async function(id) {
    this.loading = true;
    try {
      const response = await fetch(`/api/users/${id}`);
      this.data = await response.json();
    } finally {
      this.loading = false;
    }
  })
  .build();

await user.fetch(123);
```

### **Example 5: Validation Action**
```javascript
const email = ReactiveUtils.reactive({ value: '', error: null })
  .action('setValue', function(newValue) {
    this.value = newValue;
    this.validate();
  })
  .action('validate', function() {
    if (!this.value.includes('@')) {
      this.error = 'Invalid email';
    } else {
      this.error = null;
    }
  })
  .build();

email.setValue('test'); // Sets error
email.setValue('test@example.com'); // Clears error
```

---

## **Common Patterns**

### **Pattern 1: Simple Mutation**
```javascript
.action('increment', function() {
  this.count++;
})
```

### **Pattern 2: With Parameters**
```javascript
.action('add', function(amount) {
  this.count += amount;
})
```

### **Pattern 3: Async**
```javascript
.action('load', async function() {
  this.data = await fetchData();
})
```

### **Pattern 4: Batch Updates**
```javascript
.action('reset', function() {
  this.count = 0;
  this.name = '';
  this.active = false;
})
```

---

## **Key Takeaways**

1. **Single Action**: Adds one method at a time
2. **Context Bound**: `this` = state object
3. **Chainable**: Returns builder
4. **Parameters**: Actions can accept parameters
5. **Async**: Can be async functions
6. **State Access**: Full access to state via `this`
7. **Mutations**: Can modify state directly
8. **Alternative**: Use `actions()` for multiple

---

## **Summary**

`builder.action(name, fn)` adds a single action method to reactive state. Actions are functions bound to the state object where `this` refers to the state. Use actions to encapsulate state mutations and business logic. Actions can be synchronous or asynchronous and can accept parameters. The method is chainable, allowing multiple actions to be added sequentially. For adding multiple actions at once, use `builder.actions()` instead.
