# `builder.computed(defs)` - Add Computed Properties

**Quick Start (30 seconds)**
```javascript
const app = ReactiveUtils.reactive({
  firstName: 'John',
  lastName: 'Doe',
  count: 5
})
.computed({
  fullName: function() {
    return `${this.firstName} ${this.lastName}`;
  },
  double: function() {
    return this.count * 2;
  },
  triple: function() {
    return this.count * 3;
  }
})
.build();

console.log(app.fullName); // 'John Doe'
console.log(app.double); // 10
console.log(app.triple); // 15

app.count = 10;
console.log(app.double); // 20 (auto-updated)
```

---

## **What is `builder.computed(defs)`?**

`builder.computed(defs)` is a **builder method** that adds computed properties to the reactive state, creating cached,auto-updating derived values based on other state properties.

**Key characteristics:**
- **Cached**: Computed once, cached until dependencies change
- **Auto-Update**: Automatically recalculate when dependencies change
- **Chainable**: Returns builder for method chaining
- **Multiple**: Add multiple computed properties at once
- **Reactive**: Integrated with reactivity system

---

## **Syntax**

```javascript
builder.computed(definitions)
```

### **Parameters**
- **`definitions`** (Object): Object mapping property names to getter functions

### **Returns**
- **Type**: Builder instance (for chaining)

---

## **How it works**

```javascript
builder.computed(defs) {
  Object.entries(defs).forEach(([key, fn]) => addComputed(state, key, fn));
  return this;
}
```

**What happens:**
1. Iterates through definition object
2. Adds each computed property to state
3. Returns builder for chaining
4. Computed properties track dependencies automatically

---

## **Examples**

### **Example 1: Basic Computed Properties**
```javascript
const app = ReactiveUtils.reactive({
  price: 100,
  quantity: 2,
  taxRate: 0.1
})
.computed({
  subtotal: function() {
    return this.price * this.quantity;
  },
  tax: function() {
    return this.subtotal * this.taxRate;
  },
  total: function() {
    return this.subtotal + this.tax;
  }
})
.build();

console.log(app.total); // 220

app.quantity = 3;
console.log(app.total); // 330 (auto-updated)
```

### **Example 2: String Manipulation**
```javascript
const user = ReactiveUtils.reactive({
  firstName: 'john',
  lastName: 'doe',
  email: 'JOHN@EXAMPLE.COM'
})
.computed({
  fullName: function() {
    const first = this.firstName.charAt(0).toUpperCase() + this.firstName.slice(1);
    const last = this.lastName.charAt(0).toUpperCase() + this.lastName.slice(1);
    return `${first} ${last}`;
  },
  normalizedEmail: function() {
    return this.email.toLowerCase();
  },
  initials: function() {
    return `${this.firstName[0]}${this.lastName[0]}`.toUpperCase();
  }
})
.build();

console.log(user.fullName); // 'John Doe'
console.log(user.normalizedEmail); // 'john@example.com'
console.log(user.initials); // 'JD'
```

### **Example 3: Boolean Logic**
```javascript
const form = ReactiveUtils.reactive({
  email: '',
  password: '',
  agreeToTerms: false
})
.computed({
  hasEmail: function() {
    return this.email.length > 0;
  },
  hasPassword: function() {
    return this.password.length > 0;
  },
  isValid: function() {
    return this.hasEmail && this.hasPassword && this.agreeToTerms;
  },
  canSubmit: function() {
    return this.isValid && this.email.includes('@');
  }
})
.build();

form.email = 'user@example.com';
form.password = 'secret123';
form.agreeToTerms = true;

console.log(form.canSubmit); // true
```

### **Example 4: Array Operations**
```javascript
const todoList = ReactiveUtils.reactive({
  todos: [
    { id: 1, text: 'Learn Vue', done: true },
    { id: 2, text: 'Build app', done: false },
    { id: 3, text: 'Deploy', done: false }
  ]
})
.computed({
  completedTodos: function() {
    return this.todos.filter(t => t.done);
  },
  activeTodos: function() {
    return this.todos.filter(t => !t.done);
  },
  completedCount: function() {
    return this.completedTodos.length;
  },
  progress: function() {
    return (this.completedCount / this.todos.length * 100).toFixed(0) + '%';
  }
})
.build();

console.log(todoList.progress); // '33%'
console.log(todoList.activeTodos.length); // 2
```

### **Example 5: Nested Computed Dependencies**
```javascript
const cart = ReactiveUtils.reactive({
  items: [
    { price: 10, qty: 2 },
    { price: 20, qty: 1 },
    { price: 15, qty: 3 }
  ],
  discount: 0.1,
  shippingFee: 10
})
.computed({
  subtotal: function() {
    return this.items.reduce((sum, item) => sum + (item.price * item.qty), 0);
  },
  discountAmount: function() {
    return this.subtotal * this.discount;
  },
  afterDiscount: function() {
    return this.subtotal - this.discountAmount;
  },
  total: function() {
    return this.afterDiscount + this.shippingFee;
  }
})
.build();

console.log(cart.total); // 94 ((10*2 + 20*1 + 15*3) * 0.9 + 10)
```

### **Example 6: Conditional Computed**
```javascript
const app = ReactiveUtils.reactive({
  mode: 'light',
  customColor: '#FF5733'
})
.computed({
  backgroundColor: function() {
    if (this.mode === 'dark') {
      return '#1a1a1a';
    } else if (this.mode === 'light') {
      return '#ffffff';
    } else {
      return this.customColor;
    }
  },
  textColor: function() {
    return this.mode === 'dark' ? '#ffffff' : '#000000';
  }
})
.build();

console.log(app.backgroundColor); // '#ffffff'
app.mode = 'dark';
console.log(app.backgroundColor); // '#1a1a1a'
app.mode = 'custom';
console.log(app.backgroundColor); // '#FF5733'
```

### **Example 7: Date Formatting**
```javascript
const event = ReactiveUtils.reactive({
  timestamp: Date.now(),
  timeZone: 'UTC'
})
.computed({
  date: function() {
    return new Date(this.timestamp);
  },
  formatted: function() {
    return this.date.toLocaleString('en-US', { timeZone: this.timeZone });
  },
  dayOfWeek: function() {
    return this.date.toLocaleDateString('en-US', { weekday: 'long' });
  },
  relative: function() {
    const now = Date.now();
    const diff = now - this.timestamp;
    const hours = Math.floor(diff / (1000 * 60 * 60));

    if (hours < 1) return 'Just now';
    if (hours === 1) return '1 hour ago';
    return `${hours} hours ago`;
  }
})
.build();

console.log(event.formatted);
console.log(event.dayOfWeek);
console.log(event.relative);
```

### **Example 8: Validation**
```javascript
const registration = ReactiveUtils.reactive({
  username: '',
  email: '',
  password: '',
  confirmPassword: ''
})
.computed({
  usernameError: function() {
    if (this.username.length < 3) return 'Username too short';
    if (!/^[a-zA-Z0-9]+$/.test(this.username)) return 'Invalid characters';
    return null;
  },
  emailError: function() {
    if (!this.email.includes('@')) return 'Invalid email';
    return null;
  },
  passwordError: function() {
    if (this.password.length < 8) return 'Password too short';
    return null;
  },
  passwordMatchError: function() {
    if (this.password !== this.confirmPassword) return 'Passwords do not match';
    return null;
  },
  isValid: function() {
    return !this.usernameError && !this.emailError &&
           !this.passwordError && !this.passwordMatchError;
  }
})
.build();
```

### **Example 9: Statistics**
```javascript
const stats = ReactiveUtils.reactive({
  values: [10, 20, 30, 40, 50]
})
.computed({
  sum: function() {
    return this.values.reduce((a, b) => a + b, 0);
  },
  count: function() {
    return this.values.length;
  },
  average: function() {
    return this.sum / this.count;
  },
  min: function() {
    return Math.min(...this.values);
  },
  max: function() {
    return Math.max(...this.values);
  },
  range: function() {
    return this.max - this.min;
  }
})
.build();

console.log(stats.average); // 30
console.log(stats.range); // 40
```

### **Example 10: Progressive Computation**
```javascript
const builder = ReactiveUtils.reactive({
  radius: 5
});

// Add basic computed
builder.computed({
  diameter: function() {
    return this.radius * 2;
  }
});

// Add more complex computed that depend on previous
builder.computed({
  circumference: function() {
    return Math.PI * this.diameter;
  },
  area: function() {
    return Math.PI * this.radius * this.radius;
  }
});

const circle = builder.build();

console.log(circle.diameter); // 10
console.log(circle.circumference.toFixed(2)); // 31.42
console.log(circle.area.toFixed(2)); // 78.54
```

---

## **Common Patterns**

### **Pattern 1: Simple Computed**
```javascript
.computed({
  double: function() { return this.count * 2; }
})
```

### **Pattern 2: Multiple Computed**
```javascript
.computed({
  prop1: function() { return this.a + this.b; },
  prop2: function() { return this.c * this.d; }
})
```

### **Pattern 3: Dependent Computed**
```javascript
.computed({
  subtotal: function() { return this.price * this.qty; },
  total: function() { return this.subtotal + this.tax; }
})
```

### **Pattern 4: Boolean Computed**
```javascript
.computed({
  isValid: function() { return this.value > 0; }
})
```

### **Pattern 5: Array Filtering**
```javascript
.computed({
  active: function() { return this.items.filter(i => i.active); }
})
```

---

## **Best Practices**

1. **Keep computed functions pure**
   ```javascript
   // Good
   .computed({
     double: function() { return this.count * 2; }
   })

   // Bad (side effects)
   .computed({
     double: function() {
       console.log('Computing...');
       return this.count * 2;
     }
   })
   ```

2. **Use descriptive names**
   ```javascript
   .computed({
     fullName: function() { return `${this.first} ${this.last}`; }
   })
   ```

3. **Chain dependent computations**
   ```javascript
   .computed({
     subtotal: function() { return this.price * this.qty; }
   })
   .computed({
     total: function() { return this.subtotal + this.tax; }
   })
   ```

4. **Avoid expensive operations**
   ```javascript
   // Consider caching separately for very expensive ops
   .computed({
     sorted: function() {
       // This runs on every dependency change
       return this.items.slice().sort();
     }
   })
   ```

5. **Return consistent types**
   ```javascript
   .computed({
     status: function() {
       return this.active ? 'Active' : 'Inactive'; // Always string
     }
   })
   ```

---

## **Key Takeaways**

1. **Cached**: Computed values are cached and reused
2. **Auto-Update**: Recalculate when dependencies change
3. **Chainable**: Returns builder for method chaining
4. **Multiple**: Add many computed properties at once
5. **Dependencies**: Automatically track what's accessed
6. **Pure Functions**: Should be side-effect free
7. **Read-Only**: Computed properties cannot be set directly
8. **Nested**: Can depend on other computed properties
9. **Efficient**: Only recalculate when needed
10. **Reactive**: Fully integrated with reactivity system

---

## **Summary**

`builder.computed(defs)` is a builder method that adds computed properties to reactive state, creating cached, auto-updating derived values. Computed properties are defined as getter functions that automatically track their dependencies and recalculate only when those dependencies change. The method accepts an object mapping property names to getter functions, allowing you to add multiple computed properties at once. Computed properties are ideal for derived data like totals, formatted strings, filtered arrays, and boolean flags. They're cached for performance and only recalculate when their dependencies change. Use computed properties to keep your state minimal and derive complex values on-the-fly. The method returns the builder instance, enabling method chaining to add more configurations. Keep computed functions pure (no side effects) and return consistent types for predictable behavior.
