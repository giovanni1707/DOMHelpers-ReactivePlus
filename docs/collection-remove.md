# `collection.remove()` - Remove First Matching Item

## Quick Start (30 seconds)

```javascript
// Create collection
const todos = collection([
  { id: 1, text: 'Buy milk', done: false },
  { id: 2, text: 'Walk dog', done: true },
  { id: 3, text: 'Read book', done: false }
]);

// Remove by predicate function
todos.remove(item => item.id === 2);

console.log(todos.length); // 2

// Remove by direct value
const item = todos.first;
todos.remove(item);

console.log(todos.length); // 1

// Chainable
todos
  .add({ id: 4, text: 'Task 4', done: false })
  .remove(item => item.done);
```

**That's it.** Remove the first item matching a predicate or value. Chainable and reactive.

---

## What is `collection.remove()`?

`collection.remove()` **removes the first item that matches a predicate function or exact value**. It's a semantic, chainable method for removing single items from collections.

Think of it as **targeted removal** — find the first match and remove it, with a clean syntax.

**In practical terms:** Use `remove()` to delete specific items by condition or reference, one at a time.

---

## Syntax

```javascript
// Remove by predicate function
collection.remove(item => item.id === 5);

// Remove by direct value
collection.remove(itemReference);

// Chainable
collection
  .remove(predicate1)
  .remove(predicate2);

// Returns collection
const result = collection.remove(predicate);  // result === collection
```

**Parameters:**
- `predicate` - Either:
  - **Function**: `(item) => boolean` - Returns true for item to remove
  - **Value**: Direct item reference to remove

**Returns:**
- The collection itself (for chaining)

**Important:**
- Removes only the **first match**
- Returns **collection** for chaining
- Does **nothing** if no match found
- Triggers **reactivity** on removal
- Use `removeWhere()` to remove **all matches**

---

## Why Does This Exist?

### The Problem Without remove()

Removing items requires manual array manipulation:

```javascript
// ❌ Manual array manipulation - complex
const todos = collection([
  { id: 1, text: 'Task 1', done: false },
  { id: 2, text: 'Task 2', done: true }
]);

// Find index manually
const index = todos.items.findIndex(item => item.id === 1);
if (index !== -1) {
  todos.items.splice(index, 1);
}

// Verbose, error-prone (what if index is -1?)
```

**Problems:**
❌ **Verbose** - Multi-step process
❌ **Error-prone** - Must check if index exists
❌ **Not chainable** - Can't fluently combine operations
❌ **Manual** - Have to findIndex + splice yourself

### The Solution with `remove()`

```javascript
// ✅ Clean, one-line removal
const todos = collection([
  { id: 1, text: 'Task 1', done: false },
  { id: 2, text: 'Task 2', done: true }
]);

// Simple removal
todos.remove(item => item.id === 1);

// Chainable
todos
  .add({ id: 3, text: 'Task 3', done: false })
  .remove(item => item.done);
```

**Benefits:**
✅ **Concise** - Single method call
✅ **Safe** - Handles missing items gracefully
✅ **Chainable** - Fluent API support
✅ **Semantic** - Intent is clear
✅ **Flexible** - Predicate or value

---

## Mental Model: Find and Delete

Think of `remove()` like **search and destroy (first match only)**:

**Without remove() (Manual):**
```
┌─────────────────────────────┐
│  Manual Find and Remove     │
│                             │
│  1. Find index              │
│     findIndex(predicate)    │
│  2. Check if found          │
│     if (index !== -1)       │
│  3. Remove item             │
│     splice(index, 1)        │
│                             │
│  Multi-step, error-prone    │
└─────────────────────────────┘
```

**With remove() (Automatic):**
```
┌─────────────────────────────┐
│  Automatic Find and Remove  │
│                             │
│  remove(predicate)          │
│  ↓                          │
│  1. Find first match        │
│  2. Remove if found         │
│  3. Return collection       │
│                             │
│  One step, safe, chainable  │
└─────────────────────────────┘
```

`remove()` is **find and delete** — automatic, safe, chainable.

---

## How Does It Work?

`remove()` finds and removes the first match:

```
Call collection.remove(predicate)
    ↓
Is predicate a function?
    ↓
   Yes → findIndex(predicate)
    No → indexOf(value)
    ↓
Found (index !== -1)?
    ↓
   Yes → splice(index, 1)
    No → Do nothing
    ↓
Trigger reactivity
    ↓
Return this (collection)
```

**Key behaviors:**
- Accepts **function or value**
- Finds **first match** only
- Safe if **not found** (no error)
- Returns **collection** for chaining
- Triggers **reactive updates**

---

## Basic Usage

### Example 1: Remove by ID

```javascript
const users = collection([
  { id: 1, name: 'Alice' },
  { id: 2, name: 'Bob' },
  { id: 3, name: 'Charlie' }
]);

// Remove specific user
users.remove(user => user.id === 2);

console.log(users.length); // 2
console.log(users.items.map(u => u.name)); // ['Alice', 'Charlie']
```

---

### Example 2: Remove by Value

```javascript
const items = collection([1, 2, 3, 4, 5]);

// Remove direct value
items.remove(3);

console.log(items.items); // [1, 2, 4, 5]
```

---

### Example 3: Remove by Reference

```javascript
const todos = collection([
  { id: 1, text: 'Task 1' },
  { id: 2, text: 'Task 2' }
]);

const firstTodo = todos.first;
todos.remove(firstTodo);

console.log(todos.length); // 1
```

---

### Example 4: Conditional Remove

```javascript
const tasks = collection([
  { id: 1, done: false },
  { id: 2, done: true },
  { id: 3, done: false }
]);

// Remove first completed task
tasks.remove(task => task.done);

console.log(tasks.length); // 2
// Only first done task removed
```

---

### Example 5: Chainable Remove

```javascript
const notifications = collection([
  { id: 1, read: false },
  { id: 2, read: true },
  { id: 3, read: false },
  { id: 4, read: true }
]);

// Remove first read, add new
notifications
  .remove(n => n.read)
  .add({ id: 5, read: false });

console.log(notifications.length); // 4
```

---

### Example 6: Safe Removal (No Match)

```javascript
const products = collection([
  { id: 1, name: 'Laptop' },
  { id: 2, name: 'Mouse' }
]);

// Try to remove non-existent item
products.remove(p => p.id === 999);

// No error, no change
console.log(products.length); // 2
```

---

### Example 7: Remove from UI Event

```javascript
const cart = collection([
  { id: 1, name: 'Laptop', price: 999 },
  { id: 2, name: 'Mouse', price: 29 }
]);

function removeFromCart(productId) {
  cart.remove(item => item.id === productId);
}

// Button click
document.getElementById('remove-1').addEventListener('click', () => {
  removeFromCart(1);
});
```

---

### Example 8: Remove First Match Only

```javascript
const numbers = collection([1, 2, 3, 2, 4]);

// Remove first occurrence of 2
numbers.remove(n => n === 2);

console.log(numbers.items); // [1, 3, 2, 4]
// Only first 2 removed, second 2 remains
```

---

### Example 9: Complex Predicate

```javascript
const orders = collection([
  { id: 1, status: 'pending', total: 100 },
  { id: 2, status: 'completed', total: 200 },
  { id: 3, status: 'pending', total: 50 }
]);

// Remove first pending order over $75
orders.remove(order =>
  order.status === 'pending' && order.total > 75
);

console.log(orders.length); // 2
```

---

### Example 10: Remove with Effect

```javascript
const messages = collection([
  { id: 1, text: 'Hello' },
  { id: 2, text: 'World' }
]);

// Track changes
effect(() => {
  console.log('Messages:', messages.length);
});
// Output: Messages: 2

messages.remove(m => m.id === 1);
// Output: Messages: 1
```

---

## Advanced Usage: Custom Remove Logic

```javascript
const tasks = collection([]);

// Custom remove with validation
tasks.removeTask = function(taskId) {
  const task = this.find(t => t.id === taskId);

  if (!task) {
    console.warn(`Task ${taskId} not found`);
    return this;
  }

  if (task.locked) {
    throw new Error('Cannot remove locked task');
  }

  return this.remove(t => t.id === taskId);
};

// Use custom method
tasks.add({ id: 1, text: 'Task 1', locked: false });
tasks.add({ id: 2, text: 'Task 2', locked: true });

tasks.removeTask(1); // Success
// tasks.removeTask(2); // Throws error
```

---

## Common Patterns

### Pattern 1: Remove by ID

```javascript
collection.remove(item => item.id === targetId);
```

### Pattern 2: Remove by Value

```javascript
collection.remove(value);
```

### Pattern 3: Remove by Reference

```javascript
const item = collection.first;
collection.remove(item);
```

### Pattern 4: Remove First Match

```javascript
collection.remove(item => condition);
```

### Pattern 5: Chain with Add

```javascript
collection
  .remove(predicate)
  .add(newItem);
```

---

## remove() vs removeWhere()

| Feature | remove() | removeWhere() |
|---------|----------|---------------|
| Matches | First only | All matches |
| Predicate | Function or value | Function only |
| Use case | Remove specific item | Remove all matching |
| Performance | Stops after first | Checks all items |

---

## Key Takeaways

✅ **First match** - Removes only first matching item
✅ **Safe** - No error if not found
✅ **Flexible** - Accepts function or value
✅ **Chainable** - Returns collection
✅ **Reactive** - Triggers effects
✅ **Semantic** - Clear intent

---

## What's Next?

- **`collection.removeWhere()`** - Remove all matching items
- **`collection.update()`** - Update items in collection
- **`collection.clear()`** - Clear all items

---

## Summary

`collection.remove()` **removes the first item matching a predicate or value** and returns the collection for chaining. Safe, semantic, and reactive.

**The magic formula:**
```
collection.remove(item => item.id === 5)
  =
Find first match, remove it
─────────────────────────
Safe, chainable, reactive
```

Think of it as **targeted removal** — find and delete the first match with a clean, safe syntax. Perfect for removing specific items without manual array manipulation.
