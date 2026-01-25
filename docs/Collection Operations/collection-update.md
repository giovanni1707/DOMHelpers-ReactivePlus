# `collection.update()` - Update First Matching Item

## Quick Start (30 seconds)

```javascript
// Create collection
const users = collection([
  { id: 1, name: 'Alice', age: 28, active: true },
  { id: 2, name: 'Bob', age: 35, active: false },
  { id: 3, name: 'Charlie', age: 42, active: true }
]);

// Update by predicate
users.update(
  user => user.id === 2,
  { active: true, lastLogin: new Date() }
);

console.log(users.items[1].active); // true

// Update by value
const alice = users.first;
users.update(alice, { age: 29 });

console.log(users.first.age); // 29

// Chainable
users
  .update(u => u.name === 'Charlie', { age: 43 })
  .update(u => u.id === 1, { active: false });
```

**That's it.** Update the first matching item with new properties. Chainable and reactive.

---

## What is `collection.update()`?

`collection.update()` **finds the first item matching a predicate or value and merges updates into it using Object.assign**. It's a semantic, chainable method for updating items in collections.

Think of it as **targeted patch** — find an item and update specific properties without replacing the whole object.

**In practical terms:** Use `update()` to modify specific properties of collection items by condition or reference.

---

## Syntax

```javascript
// Update by predicate function
collection.update(
  item => item.id === 5,
  { property: newValue }
);

// Update by direct value/reference
collection.update(itemReference, updates);

// Chainable
collection
  .update(predicate1, updates1)
  .update(predicate2, updates2);

// Returns collection
const result = collection.update(predicate, updates);  // result === collection
```

**Parameters:**
- `predicate` - Either:
  - **Function**: `(item) => boolean` - Returns true for item to update
  - **Value**: Direct item reference to update
- `updates` - Object with properties to merge into the item

**Returns:**
- The collection itself (for chaining)

**Important:**
- Updates only the **first match**
- Uses **Object.assign** to merge properties
- Does **nothing** if no match found
- Triggers **reactivity** on update
- Use `updateWhere()` to update **all matches**

---

## Why Does This Exist?

### The Problem Without update()

Updating items requires finding and manually assigning:

```javascript
// ❌ Manual update - verbose
const users = collection([
  { id: 1, name: 'Alice', age: 28 }
]);

// Find item manually
const index = users.items.findIndex(u => u.id === 1);
if (index !== -1) {
  Object.assign(users.items[index], { age: 29 });
}

// Or even more verbose
const user = users.items.find(u => u.id === 1);
if (user) {
  user.age = 29;
  user.lastUpdated = new Date();
}
```

**Problems:**
❌ **Verbose** - Multi-step process
❌ **Error-prone** - Must check if found
❌ **Not chainable** - Can't fluently combine
❌ **Manual** - Have to find + assign yourself

### The Solution with `update()`

```javascript
// ✅ Clean, one-line update
const users = collection([
  { id: 1, name: 'Alice', age: 28 }
]);

users.update(
  u => u.id === 1,
  { age: 29, lastUpdated: new Date() }
);

// Chainable
users
  .update(u => u.id === 1, { active: true })
  .update(u => u.id === 2, { active: false });
```

**Benefits:**
✅ **Concise** - Single method call
✅ **Safe** - Handles missing items gracefully
✅ **Chainable** - Fluent API support
✅ **Semantic** - Intent is clear
✅ **Automatic merge** - Object.assign built-in

---

## Mental Model: Find and Patch

Think of `update()` like **search and patch (first match only)**:

**Without update() (Manual):**
```
┌─────────────────────────────┐
│  Manual Find and Update     │
│                             │
│  1. Find item               │
│     find(predicate)         │
│  2. Check if found          │
│     if (item)               │
│  3. Update properties       │
│     Object.assign(item, {}) │
│                             │
│  Multi-step, error-prone    │
└─────────────────────────────┘
```

**With update() (Automatic):**
```
┌─────────────────────────────┐
│  Automatic Find and Update  │
│                             │
│  update(predicate, updates) │
│  ↓                          │
│  1. Find first match        │
│  2. Merge properties        │
│  3. Return collection       │
│                             │
│  One step, safe, chainable  │
└─────────────────────────────┘
```

`update()` is **find and patch** — automatic, safe, chainable.

---

## How Does It Work?

`update()` finds and merges properties into the first match:

```
Call collection.update(predicate, updates)
    ↓
Is predicate a function?
    ↓
   Yes → findIndex(predicate)
    No → indexOf(value)
    ↓
Found (index !== -1)?
    ↓
   Yes → Object.assign(items[index], updates)
    No → Do nothing
    ↓
Trigger reactivity
    ↓
Return this (collection)
```

**Key behaviors:**
- Accepts **function or value**
- Updates **first match** only
- Uses **Object.assign** (shallow merge)
- Safe if **not found** (no error)
- Returns **collection** for chaining
- Triggers **reactive updates**

---

## Basic Usage

### Example 1: Update by ID

```javascript
const tasks = collection([
  { id: 1, text: 'Buy milk', done: false },
  { id: 2, text: 'Walk dog', done: false }
]);

// Mark task as done
tasks.update(
  task => task.id === 1,
  { done: true, completedAt: new Date() }
);

console.log(tasks.items[0].done); // true
```

---

### Example 2: Update by Reference

```javascript
const users = collection([
  { id: 1, name: 'Alice', score: 100 },
  { id: 2, name: 'Bob', score: 150 }
]);

const alice = users.first;
users.update(alice, { score: 120 });

console.log(alice.score); // 120
```

---

### Example 3: Partial Update

```javascript
const products = collection([
  { id: 1, name: 'Laptop', price: 999, stock: 5 }
]);

// Update only price, keep other properties
products.update(
  p => p.id === 1,
  { price: 899 }
);

console.log(products.first);
// { id: 1, name: 'Laptop', price: 899, stock: 5 }
```

---

### Example 4: Chainable Updates

```javascript
const inventory = collection([
  { sku: 'A001', stock: 10, reserved: 0 },
  { sku: 'A002', stock: 5, reserved: 0 }
]);

// Chain multiple updates
inventory
  .update(item => item.sku === 'A001', { reserved: 2 })
  .update(item => item.sku === 'A002', { stock: 8 });
```

---

### Example 5: Update with Calculated Values

```javascript
const cart = collection([
  { id: 1, name: 'Laptop', price: 999, qty: 1 }
]);

// Update quantity and recalculate total
const item = cart.first;
const newQty = item.qty + 1;

cart.update(item, {
  qty: newQty,
  total: item.price * newQty
});
```

---

### Example 6: Toggle Property

```javascript
const settings = collection([
  { id: 'notifications', enabled: true },
  { id: 'darkMode', enabled: false }
]);

const darkMode = settings.find(s => s.id === 'darkMode');

// Toggle enabled
settings.update(darkMode, { enabled: !darkMode.enabled });

console.log(darkMode.enabled); // true
```

---

### Example 7: Update from Form

```javascript
const profiles = collection([
  { id: 1, name: 'Alice', email: 'alice@example.com' }
]);

function updateProfile(userId, formData) {
  profiles.update(
    p => p.id === userId,
    {
      name: formData.name,
      email: formData.email,
      updatedAt: new Date()
    }
  );
}

updateProfile(1, {
  name: 'Alice Smith',
  email: 'alice.smith@example.com'
});
```

---

### Example 8: Increment Counter

```javascript
const posts = collection([
  { id: 1, title: 'Post 1', likes: 10, views: 100 }
]);

function likePost(postId) {
  const post = posts.find(p => p.id === postId);
  if (post) {
    posts.update(post, { likes: post.likes + 1 });
  }
}

likePost(1);
console.log(posts.first.likes); // 11
```

---

### Example 9: Update First Match Only

```javascript
const tasks = collection([
  { id: 1, status: 'pending', priority: 'high' },
  { id: 2, status: 'pending', priority: 'low' },
  { id: 3, status: 'pending', priority: 'medium' }
]);

// Update first pending task only
tasks.update(
  t => t.status === 'pending',
  { status: 'in-progress', startedAt: new Date() }
);

console.log(tasks.items[0].status); // 'in-progress'
console.log(tasks.items[1].status); // 'pending' (unchanged)
```

---

### Example 10: Safe Update (No Match)

```javascript
const items = collection([
  { id: 1, value: 'A' },
  { id: 2, value: 'B' }
]);

// Try to update non-existent item
items.update(
  item => item.id === 999,
  { value: 'C' }
);

// No error, no change
console.log(items.length); // 2
```

---

## Advanced Usage: Complex Updates

```javascript
const orders = collection([]);

// Custom update with validation
orders.updateOrder = function(orderId, updates) {
  const order = this.find(o => o.id === orderId);

  if (!order) {
    throw new Error(`Order ${orderId} not found`);
  }

  if (order.status === 'shipped') {
    throw new Error('Cannot update shipped order');
  }

  // Validate updates
  if (updates.total && updates.total < 0) {
    throw new Error('Total cannot be negative');
  }

  return this.update(
    o => o.id === orderId,
    {
      ...updates,
      updatedAt: new Date(),
      version: order.version + 1
    }
  );
};

// Use custom method
orders.add({ id: 1, total: 100, status: 'pending', version: 1 });
orders.updateOrder(1, { total: 150 });
```

---

## Common Patterns

### Pattern 1: Update by ID

```javascript
collection.update(
  item => item.id === targetId,
  { property: value }
);
```

### Pattern 2: Update by Reference

```javascript
const item = collection.find(predicate);
collection.update(item, updates);
```

### Pattern 3: Partial Update

```javascript
collection.update(predicate, {
  field1: value1,
  field2: value2
});
```

### Pattern 4: Update with Timestamp

```javascript
collection.update(predicate, {
  ...updates,
  updatedAt: new Date()
});
```

### Pattern 5: Chain Updates

```javascript
collection
  .update(pred1, updates1)
  .update(pred2, updates2);
```

---

## update() vs updateWhere()

| Feature | update() | updateWhere() |
|---------|----------|---------------|
| Matches | First only | All matches |
| Predicate | Function or value | Function only |
| Use case | Update specific item | Update all matching |
| Performance | Stops after first | Checks all items |

---

## Key Takeaways

✅ **First match** - Updates only first matching item
✅ **Shallow merge** - Uses Object.assign
✅ **Safe** - No error if not found
✅ **Flexible** - Accepts function or value
✅ **Chainable** - Returns collection
✅ **Reactive** - Triggers effects

---

## What's Next?

- **`collection.updateWhere()`** - Update all matching items
- **`collection.toggle()`** - Toggle boolean property
- **`collection.remove()`** - Remove items from collection

---

## Summary

`collection.update()` **finds the first item matching a predicate or value and merges updates into it**. Safe, semantic, and reactive.

**The magic formula:**
```
collection.update(
  item => item.id === 5,
  { status: 'active' }
)
  =
Find first match, merge properties
──────────────────────────────────
Safe, chainable, reactive
```

Think of it as **targeted patch** — find and update specific properties of the first match without manual array manipulation. Perfect for updating individual items with clean syntax.
