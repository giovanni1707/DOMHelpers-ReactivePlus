# `set()` - Functional State Updates

## Quick Start (30 seconds)

```javascript
// Create state
const counter = state({
  count: 0,
  multiplier: 2
});

// Update with functions - access previous values
set(counter, {
  count: (prev) => prev + 1,
  multiplier: (prev) => prev * 2
});

console.log(counter.count);      // 1
console.log(counter.multiplier); // 4

// Mix functions and direct values
set(counter, {
  count: (prev) => prev + 10,  // Function
  multiplier: 1                 // Direct value
});
```

**That's it.** Update state properties using functions that receive previous values. Perfect for updates based on current state.

---

## What is `set()`?

`set()` **updates state properties using functional updates**. When you provide a function as a value, it receives the current value and returns the new value. This ensures updates are based on the most recent state.

Think of it as **React's setState with functions** — instead of setting values directly, you define how to transform current values into new ones.

**In practical terms:** Use `set()` when your update depends on the current value (incrementing, toggling, transforming), ensuring you always work with the latest state.

---

## Syntax

```javascript
// Update state with functions
set(stateObject, updates);

// Updates object
const updates = {
  // Function receives previous value
  propertyName: (previousValue) => newValue,

  // Or use direct values
  otherProperty: directValue
};

// Example
const user = state({
  score: 100,
  level: 1,
  name: 'Alice'
});

set(user, {
  score: (prev) => prev + 50,  // Add to current
  level: (prev) => prev + 1,   // Increment
  name: 'Bob'                  // Direct assignment
});
```

**Parameters:**
- `stateObject` - Reactive state object to update
- `updates` - Object with property names as keys and functions/values as values

**Returns:**
- The state object (for chaining)

**Important:**
- Functions receive the **current value** as the first argument
- Can mix functional updates and direct values
- All updates are **batched automatically**
- Supports **nested property paths** using dot notation

---

## Why Does This Exist?

### The Problem Without set()

Updating state based on current values can have timing issues:

```javascript
// ❌ Direct updates - can miss intermediate changes
const counter = state({ count: 0 });

// Imagine these happen in quick succession
counter.count = counter.count + 1;  // 0 + 1 = 1
counter.count = counter.count + 1;  // Should be 2, but might read stale 0

// Problem: If updates happen too fast, you might read
// the old value before the reactive update propagates
```

**Problems:**
❌ **Race conditions** - Fast updates might read stale values
❌ **No guarantee** - Updates might not stack correctly
❌ **Manual tracking** - Must carefully manage current values
❌ **Error-prone** - Easy to use outdated values in calculations

### The Solution with `set()`

```javascript
// ✅ DOM Helpers + set() - guaranteed current values
const counter = state({ count: 0 });

// These will correctly stack, even if called rapidly
set(counter, {
  count: (prev) => prev + 1  // Always uses latest value
});
set(counter, {
  count: (prev) => prev + 1  // Guaranteed to see previous update
});

// Result: count is definitely 2
```

**Benefits:**
✅ **No race conditions** - Functions always receive current values
✅ **Guaranteed order** - Updates apply in sequence
✅ **Batched efficiently** - All updates optimized automatically
✅ **Cleaner code** - Express intent clearly with functions
✅ **Nested path support** - Update deeply nested properties safely

---

## Mental Model: Transform Current Value

Think of `set()` like **a transformation pipeline**:

**Without set() (Direct Assignment):**
```
┌─────────────────────────────┐
│  Direct Assignment          │
│                             │
│  counter.count = 10         │
│  ↓                          │
│  Replace with 10            │
│                             │
│  Problem: What if current   │
│  value matters?             │
│                             │
│  Must read first:           │
│  counter.count =            │
│    counter.count + 1        │
│  (might be stale)           │
└─────────────────────────────┘
```

**With set() (Functional Update):**
```
┌─────────────────────────────┐
│  Functional Transform       │
│                             │
│  set(counter, {             │
│    count: (prev) => prev+1  │
│  })                         │
│  ↓                          │
│  Read current: 10           │
│  ↓                          │
│  Transform: 10 + 1 = 11     │
│  ↓                          │
│  Update: 11                 │
│                             │
│  Always uses latest value!  │
└─────────────────────────────┘
```

`set()` is **a transformation function** — it receives current values and returns new ones.

---

## How Does It Work?

`set()` processes each update with the current value:

```
Call set(state, updates)
    ↓
Batch updates begin
    ↓
For each property:
  ┌─────────────────────┐
  │ Is value function?  │
  └─────────────────────┘
         ↓          ↓
        Yes         No
         ↓          ↓
    Get current   Use value
    value (prev)  directly
         ↓          ↓
    Call fn(prev)  │
         ↓          ↓
    Get result     │
         ↓          ↓
         └────┬─────┘
              ↓
       Set new value
              ↓
    Trigger reactivity
              ↓
Batch updates complete
    ↓
Return state object
```

**Key behaviors:**
- Functions are **called with current value**
- Direct values are **assigned as-is**
- Updates are **batched** for performance
- Supports **dot notation** for nested paths
- All changes **trigger reactivity**

---

## Basic Usage

### Example 1: Counter Operations

```javascript
const counter = state({
  count: 0,
  clicks: 0
});

// Increment
function increment() {
  set(counter, {
    count: (prev) => prev + 1,
    clicks: (prev) => prev + 1
  });
}

// Decrement
function decrement() {
  set(counter, {
    count: (prev) => prev - 1,
    clicks: (prev) => prev + 1
  });
}

// Reset count but keep clicks
function reset() {
  set(counter, {
    count: 0,           // Direct value
    clicks: (prev) => prev + 1  // Function
  });
}
```

---

### Example 2: Toggle Boolean

```javascript
const app = state({
  darkMode: false,
  sidebarOpen: true
});

// Toggle dark mode
function toggleDarkMode() {
  set(app, {
    darkMode: (prev) => !prev
  });
}

// Toggle sidebar
function toggleSidebar() {
  set(app, {
    sidebarOpen: (prev) => !prev
  });
}

// Toggle both
function toggleAll() {
  set(app, {
    darkMode: (prev) => !prev,
    sidebarOpen: (prev) => !prev
  });
}
```

---

### Example 3: Array Operations

```javascript
const todos = state({
  items: ['Task 1', 'Task 2'],
  completedCount: 0
});

// Add item
function addTodo(text) {
  set(todos, {
    items: (prev) => [...prev, text]
  });
}

// Remove item
function removeTodo(index) {
  set(todos, {
    items: (prev) => prev.filter((_, i) => i !== index),
    completedCount: (prev) => Math.max(0, prev - 1)
  });
}

// Clear all
function clearAll() {
  set(todos, {
    items: (prev) => [],
    completedCount: 0
  });
}
```

---

### Example 4: Object Merging

```javascript
const user = state({
  profile: {
    name: 'Alice',
    age: 30
  },
  settings: {
    theme: 'light',
    notifications: true
  }
});

// Update profile
function updateProfile(updates) {
  set(user, {
    profile: (prev) => ({ ...prev, ...updates })
  });
}

// Toggle notification
function toggleNotifications() {
  set(user, {
    settings: (prev) => ({
      ...prev,
      notifications: !prev.notifications
    })
  });
}
```

---

### Example 5: Calculations Based on State

```javascript
const cart = state({
  items: [
    { name: 'Laptop', price: 999, qty: 1 }
  ],
  discountPercent: 0,
  tax: 0
});

// Add item and recalculate
function addItem(item) {
  set(cart, {
    items: (prev) => [...prev, item],
    tax: (prev) => {
      // Recalculate tax based on new total
      const newItems = [...prev.items, item];
      const subtotal = newItems.reduce((s, i) => s + i.price * i.qty, 0);
      return subtotal * 0.08;
    }
  });
}

// Apply discount
function applyDiscount(percent) {
  set(cart, {
    discountPercent: percent
  });
}
```

---

### Example 6: Nested Property Updates

```javascript
const app = state({
  user: {
    profile: {
      name: 'Alice',
      email: 'alice@example.com'
    },
    preferences: {
      theme: 'light'
    }
  }
});

// Update nested property using dot notation
function updateEmail(newEmail) {
  set(app, {
    'user.profile.email': newEmail
  });
}

// Update with function
function toggleTheme() {
  set(app, {
    'user.preferences.theme': (prev) => prev === 'light' ? 'dark' : 'light'
  });
}
```

---

### Example 7: Multiple Related Updates

```javascript
const game = state({
  score: 0,
  level: 1,
  health: 100,
  powerUps: 0
});

// Collect power-up
function collectPowerUp() {
  set(game, {
    score: (prev) => prev + 100,
    powerUps: (prev) => prev + 1,
    health: (prev) => Math.min(100, prev + 20)  // Cap at 100
  });
}

// Take damage
function takeDamage(amount) {
  set(game, {
    health: (prev) => Math.max(0, prev - amount),  // Floor at 0
    score: (prev) => Math.max(0, prev - 10)        // Penalty
  });
}

// Level up
function levelUp() {
  set(game, {
    level: (prev) => prev + 1,
    health: 100,              // Reset to full
    score: (prev) => prev + (game.level * 500)  // Bonus based on level
  });
}
```

---

### Example 8: Form Field Updates

```javascript
const form = state({
  values: {
    email: '',
    password: ''
  },
  errors: {},
  touched: {}
});

// Update field with validation
function updateField(field, value) {
  set(form, {
    [`values.${field}`]: value,
    errors: (prev) => {
      const newErrors = { ...prev };
      if (!validateField(field, value)) {
        newErrors[field] = 'Invalid value';
      } else {
        delete newErrors[field];
      }
      return newErrors;
    },
    touched: (prev) => ({ ...prev, [field]: true })
  });
}
```

---

### Example 9: Timer/Interval Updates

```javascript
const timer = state({
  seconds: 0,
  isRunning: false,
  laps: []
});

// Tick
function tick() {
  set(timer, {
    seconds: (prev) => prev + 1
  });
}

// Record lap
function recordLap() {
  set(timer, {
    laps: (prev) => [...prev, timer.seconds]
  });
}

// Reset
function reset() {
  set(timer, {
    seconds: 0,
    isRunning: false,
    laps: []
  });
}
```

---

### Example 10: Conditional Updates

```javascript
const inventory = state({
  stock: 50,
  reorderPoint: 10,
  needsReorder: false,
  orders: 0
});

// Sell item
function sellItem(quantity) {
  set(inventory, {
    stock: (prev) => prev - quantity,
    needsReorder: (prev) => {
      const newStock = inventory.stock - quantity;
      return newStock < inventory.reorderPoint;
    },
    orders: (prev) => {
      const newStock = inventory.stock - quantity;
      // Auto-order if below reorder point
      return newStock < inventory.reorderPoint ? prev + 1 : prev;
    }
  });
}

// Restock
function restock(quantity) {
  set(inventory, {
    stock: (prev) => prev + quantity,
    needsReorder: false,
    orders: 0
  });
}
```

---

## Advanced Usage: Complex Transformations

```javascript
const app = state({
  users: [
    { id: 1, name: 'Alice', active: true },
    { id: 2, name: 'Bob', active: false }
  ],
  stats: {
    total: 2,
    active: 1
  }
});

// Update user and recalculate stats
function updateUser(id, updates) {
  set(app, {
    users: (prev) => prev.map(user =>
      user.id === id ? { ...user, ...updates } : user
    ),
    stats: (prev) => {
      // Recalculate based on new users array
      const updatedUsers = app.users.map(user =>
        user.id === id ? { ...user, ...updates } : user
      );
      return {
        total: updatedUsers.length,
        active: updatedUsers.filter(u => u.active).length
      };
    }
  });
}
```

---

## Common Patterns

### Pattern 1: Increment/Decrement

```javascript
set(state, {
  count: (prev) => prev + 1,
  total: (prev) => prev + amount
});
```

### Pattern 2: Toggle Boolean

```javascript
set(state, {
  enabled: (prev) => !prev,
  visible: (prev) => !prev
});
```

### Pattern 3: Array Add/Remove

```javascript
// Add
set(state, {
  items: (prev) => [...prev, newItem]
});

// Remove
set(state, {
  items: (prev) => prev.filter(item => item.id !== id)
});
```

### Pattern 4: Object Merge

```javascript
set(state, {
  user: (prev) => ({ ...prev, ...updates }),
  settings: (prev) => ({ ...prev, theme: 'dark' })
});
```

### Pattern 5: Min/Max Bounds

```javascript
set(state, {
  volume: (prev) => Math.max(0, Math.min(100, prev + delta))
});
```

---

## Key Takeaways

✅ **Functional updates** - Functions receive current values
✅ **No race conditions** - Always work with latest state
✅ **Batched automatically** - Efficient updates
✅ **Mix and match** - Combine functions and direct values
✅ **Nested paths** - Use dot notation for deep updates
✅ **Reactive** - All updates trigger effects

---

## What's Next?

- **`updateAll()`** - Unified state and DOM updates
- **`batch()`** - Manual batching control
- **`notify()`** - Manual dependency notifications

---

## Summary

`set()` **updates state properties using functional updates**. Functions receive current values and return new ones, ensuring updates are always based on the latest state.

**The magic formula:**
```
set(state, {
  prop: (prev) => newValue
}) =
  Transform current value to new value
────────────────────────────────────
No race conditions, guaranteed order
```

Think of it as **React's setState with functions** — define transformations instead of direct assignments. Perfect for increments, toggles, array operations, and any update that depends on current state values.
