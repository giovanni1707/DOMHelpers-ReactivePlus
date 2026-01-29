# `builder.actions(defs)` - Add Multiple Actions

**Quick Start (30 seconds)**
```javascript
const counter = ReactiveUtils.reactive({ count: 0 })
  .actions({
    increment: function() {
      this.count++;
    },
    decrement: function() {
      this.count--;
    },
    add: function(amount) {
      this.count += amount;
    },
    reset: function() {
      this.count = 0;
    }
  })
  .build();

counter.increment(); // 1
counter.add(5); // 6
counter.decrement(); // 5
counter.reset(); // 0
```

---

## **What is `builder.actions(defs)`?**

`builder.actions(defs)` is a **builder method** that adds multiple actions (methods) to the reactive state at once.

**Key characteristics:**
- **Batch Definition**: Add many methods at once
- **Context Bound**: `this` refers to state in all actions
- **Chainable**: Returns builder for method chaining
- **Organized**: Keep related actions together
- **State Mutation**: Actions can modify state

---

## **Syntax**

```javascript
builder.actions(definitions)
```

### **Parameters**
- **`definitions`** (Object): Object mapping action names to functions

### **Returns**
- **Type**: Builder instance (for chaining)

---

## **How it works**

```javascript
builder.actions(defs) {
  Object.entries(defs).forEach(([name, fn]) => this.action(name, fn));
  return this;
}
```

---

## **Examples**

### **Example 1: Counter Actions**
```javascript
const counter = ReactiveUtils.reactive({ count: 0, step: 1 })
  .actions({
    increment: function() {
      this.count += this.step;
    },
    decrement: function() {
      this.count -= this.step;
    },
    setStep: function(newStep) {
      this.step = newStep;
    },
    reset: function() {
      this.count = 0;
      this.step = 1;
    }
  })
  .build();

counter.increment(); // 1
counter.setStep(5);
counter.increment(); // 6
```

### **Example 2: Todo List**
```javascript
const todos = ReactiveUtils.reactive({ items: [] })
  .actions({
    add: function(text) {
      this.items.push({
        id: Date.now(),
        text,
        done: false,
        createdAt: new Date()
      });
    },

    toggle: function(id) {
      const todo = this.items.find(t => t.id === id);
      if (todo) todo.done = !todo.done;
    },

    remove: function(id) {
      this.items = this.items.filter(t => t.id !== id);
    },

    clear: function() {
      this.items = [];
    },

    clearCompleted: function() {
      this.items = this.items.filter(t => !t.done);
    }
  })
  .build();
```

### **Example 3: Shopping Cart**
```javascript
const cart = ReactiveUtils.reactive({
  items: [],
  discount: 0
})
.actions({
  addItem: function(product, quantity = 1) {
    const existing = this.items.find(i => i.product.id === product.id);

    if (existing) {
      existing.quantity += quantity;
    } else {
      this.items.push({ product, quantity });
    }
  },

  removeItem: function(productId) {
    this.items = this.items.filter(i => i.product.id !== productId);
  },

  updateQuantity: function(productId, quantity) {
    const item = this.items.find(i => i.product.id === productId);
    if (item) item.quantity = quantity;
  },

  applyDiscount: function(percent) {
    this.discount = percent;
  },

  clear: function() {
    this.items = [];
    this.discount = 0;
  }
})
.computed({
  subtotal: function() {
    return this.items.reduce((sum, item) =>
      sum + (item.product.price * item.quantity), 0
    );
  },
  total: function() {
    return this.subtotal * (1 - this.discount);
  }
})
.build();
```

### **Example 4: User Authentication**
```javascript
const auth = ReactiveUtils.reactive({
  user: null,
  loading: false,
  error: null
})
.actions({
  async login(credentials) {
    this.loading = true;
    this.error = null;

    try {
      const response = await fetch('/api/login', {
        method: 'POST',
        body: JSON.stringify(credentials)
      });

      if (!response.ok) throw new Error('Login failed');

      this.user = await response.json();
    } catch (error) {
      this.error = error.message;
    } finally {
      this.loading = false;
    }
  },

  async logout() {
    await fetch('/api/logout', { method: 'POST' });
    this.user = null;
    this.error = null;
  },

  clearError() {
    this.error = null;
  }
})
.build();
```

### **Example 5: Form Management**
```javascript
const form = ReactiveUtils.reactive({
  values: {},
  errors: {},
  touched: {},
  isSubmitting: false
})
.actions({
  setField(name, value) {
    this.values[name] = value;
    this.touched[name] = true;
  },

  setError(name, error) {
    this.errors[name] = error;
  },

  clearError(name) {
    delete this.errors[name];
  },

  async submit(onSubmit) {
    this.isSubmitting = true;

    try {
      await onSubmit(this.values);
      this.reset();
    } catch (error) {
      this.errors._form = error.message;
    } finally {
      this.isSubmitting = false;
    }
  },

  reset() {
    this.values = {};
    this.errors = {};
    this.touched = {};
  }
})
.build();
```

---

## **Common Patterns**

### **Pattern 1: CRUD Operations**
```javascript
.actions({
  create: function(data) { /* ... */ },
  read: function(id) { /* ... */ },
  update: function(id, data) { /* ... */ },
  delete: function(id) { /* ... */ }
})
```

### **Pattern 2: State Management**
```javascript
.actions({
  set: function(key, value) { this[key] = value; },
  reset: function() { Object.assign(this, initialState); }
})
```

### **Pattern 3: Async Operations**
```javascript
.actions({
  async fetch() { /* ... */ },
  async save() { /* ... */ },
  async delete() { /* ... */ }
})
```

---

## **Best Practices**

1. **Group related actions**
   ```javascript
   .actions({
     // User actions
     login: function() {},
     logout: function() {},
     updateProfile: function() {}
   })
   ```

2. **Use descriptive names**
   ```javascript
   .actions({
     addTodo: function() {}, // Clear
     toggle: function() {}    // Less clear
   })
   ```

3. **Keep actions focused**
   ```javascript
   // Good
   .actions({
     increment: function() { this.count++; }
   })

   // Bad (doing too much)
   .actions({
     increment: function() {
       this.count++;
       this.updateUI();
       this.syncToServer();
     }
   })
   ```

4. **Use for mutations only**
   ```javascript
   .actions({
     setUser: function(user) {
       this.user = user;
     }
   })
   ```

---

## **Key Takeaways**

1. **Batch Definition**: Add multiple methods at once
2. **Organized**: Keep related actions together
3. **Context Bound**: `this` = state in all actions
4. **Chainable**: Returns builder
5. **Async Support**: Actions can be async
6. **State Access**: Full access via `this`
7. **Alternative**: Use `action()` for single actions
8. **Clean Code**: Better organization than individual calls

---

## **Summary**

`builder.actions(defs)` adds multiple action methods to reactive state at once. It accepts an object mapping action names to functions, where each function receives the state as `this`. Actions encapsulate state mutations and business logic, keeping your code organized. The method is equivalent to calling `action()` multiple times but more concise. Actions can be synchronous or asynchronous and can accept parameters. Use actions to create a clean API for manipulating state.
