# `collection.add()` - Add Item to Collection

## Quick Start (30 seconds)

```javascript
// Create collection
const todos = collection([
  { id: 1, text: 'Buy milk', done: false }
]);

// Add single item
todos.add({ id: 2, text: 'Walk dog', done: false });

console.log(todos.items.length); // 2

// Chainable - add multiple items
todos
  .add({ id: 3, text: 'Read book', done: false })
  .add({ id: 4, text: 'Write code', done: true });

console.log(todos.items.length); // 4
```

**That's it.** Add items to a collection with a chainable method. Perfect for building fluent APIs.

---

## What is `collection.add()`?

`collection.add()` **adds an item to the end of the collection and returns the collection for chaining**. It's a convenient wrapper around `items.push()` with a more semantic name and chainability.

Think of it as **the fluent push** — add items with a clean, chainable syntax that reads like English.

**In practical terms:** Use `add()` when you want to append items to a collection with a clear, chainable syntax.

---

## Syntax

```javascript
// Add single item
collection.add(item);

// Chainable
collection
  .add(item1)
  .add(item2)
  .add(item3);

// Returns collection for chaining
const result = collection.add(item);  // result === collection
```

**Parameters:**
- `item` - Item to add to the collection

**Returns:**
- The collection itself (for chaining)

**Important:**
- Adds to the **end** of the array (like `push`)
- **Chainable** — returns the collection
- Triggers **reactivity** — effects run
- Can add **any type** of item

---

## Why Does This Exist?

### The Problem Without add()

Adding items requires verbose array access:

```javascript
// ❌ Direct array manipulation - verbose
const todos = collection([]);

todos.items.push({ id: 1, text: 'Task 1', done: false });
todos.items.push({ id: 2, text: 'Task 2', done: false });
todos.items.push({ id: 3, text: 'Task 3', done: false });

// Not chainable, repetitive "items.push"
```

**Problems:**
❌ **Verbose** - Repetitive `items.push()`
❌ **Not chainable** - Can't fluently add multiple items
❌ **Less semantic** - `push` doesn't express intent clearly
❌ **More typing** - Extra characters for each add

### The Solution with `add()`

```javascript
// ✅ Clean, chainable syntax
const todos = collection([]);

todos
  .add({ id: 1, text: 'Task 1', done: false })
  .add({ id: 2, text: 'Task 2', done: false })
  .add({ id: 3, text: 'Task 3', done: false });

// Chainable, semantic, concise
```

**Benefits:**
✅ **Concise** - Shorter syntax
✅ **Chainable** - Fluent API pattern
✅ **Semantic** - "add" is clearer than "push"
✅ **Consistent** - Matches collection API style

---

## Mental Model: Fluent Builder

Think of `add()` like **building a chain**:

**Without add() (Repetitive):**
```
┌─────────────────────────────┐
│  Repetitive Array Push      │
│                             │
│  collection.items.push(1)   │
│  collection.items.push(2)   │
│  collection.items.push(3)   │
│                             │
│  Verbose, not chainable     │
└─────────────────────────────┘
```

**With add() (Fluent Chain):**
```
┌─────────────────────────────┐
│  Fluent Chaining            │
│                             │
│  collection                 │
│    .add(1)                  │
│    .add(2)                  │
│    .add(3)                  │
│                             │
│  Clean, chainable           │
└─────────────────────────────┘
```

`add()` is **the fluent builder** — chain operations naturally.

---

## How Does It Work?

`add()` pushes to the array and returns the collection:

```
Call collection.add(item)
    ↓
Push item to items array
    ↓
this.items.push(item)
    ↓
Trigger reactivity
    ↓
Return this (the collection)
    ↓
Enable chaining
```

**Key behaviors:**
- Uses `push` internally
- Adds to **end** of array
- Returns `this` for **chaining**
- Triggers **reactive updates**
- Works with **any item type**

---

## Basic Usage

### Example 1: Simple Todo List

```javascript
const todos = collection([]);

// Add tasks
todos.add({ id: 1, text: 'Buy groceries', done: false });
todos.add({ id: 2, text: 'Call mom', done: false });

console.log(todos.items.length); // 2
```

---

### Example 2: Chaining

```javascript
const users = collection([]);

// Chain multiple adds
users
  .add({ id: 1, name: 'Alice', role: 'admin' })
  .add({ id: 2, name: 'Bob', role: 'user' })
  .add({ id: 3, name: 'Charlie', role: 'user' });

console.log(users.length); // 3
```

---

### Example 3: Reactive Updates

```javascript
const notifications = collection([]);

// React to changes
effect(() => {
  console.log('Notifications:', notifications.length);
});
// Output: Notifications: 0

notifications.add({ id: 1, message: 'New email' });
// Output: Notifications: 1

notifications.add({ id: 2, message: 'Update available' });
// Output: Notifications: 2
```

---

### Example 4: Dynamic ID Generation

```javascript
const tasks = collection([]);

function addTask(text) {
  tasks.add({
    id: Date.now(),
    text: text,
    done: false,
    createdAt: new Date()
  });
}

addTask('Buy milk');
addTask('Walk dog');

console.log(tasks.items);
// [{ id: 1234..., text: 'Buy milk', ... }, ...]
```

---

### Example 5: Conditional Adding

```javascript
const cart = collection([]);

function addToCart(product) {
  if (!cart.find(item => item.id === product.id)) {
    cart.add(product);
  }
}

addToCart({ id: 1, name: 'Laptop', price: 999 });
addToCart({ id: 1, name: 'Laptop', price: 999 }); // Duplicate, not added

console.log(cart.length); // 1
```

---

### Example 6: Builder Pattern

```javascript
const playlist = collection([])
  .add({ id: 1, title: 'Song 1', duration: 180 })
  .add({ id: 2, title: 'Song 2', duration: 210 })
  .add({ id: 3, title: 'Song 3', duration: 195 });

console.log(playlist.length); // 3
```

---

### Example 7: Event Handler

```javascript
const events = collection([]);

document.getElementById('addBtn').addEventListener('click', () => {
  events.add({
    type: 'click',
    timestamp: Date.now(),
    target: 'addBtn'
  });
});
```

---

### Example 8: API Response

```javascript
const products = collection([]);

async function loadProducts() {
  const response = await fetch('/api/products');
  const data = await response.json();

  data.forEach(product => {
    products.add(product);
  });
}
```

---

### Example 9: Form Submission

```javascript
const submissions = collection([]);

function handleSubmit(formData) {
  submissions.add({
    id: Date.now(),
    data: formData,
    submittedAt: new Date(),
    status: 'pending'
  });
}
```

---

### Example 10: Real-time Updates

```javascript
const messages = collection([]);

// WebSocket connection
socket.on('message', (msg) => {
  messages.add({
    id: msg.id,
    text: msg.text,
    sender: msg.sender,
    timestamp: msg.timestamp
  });
});
```

---

## Advanced Usage: Custom Wrapper

```javascript
const tasks = collection([]);

// Custom add with validation
tasks.addTask = function(text) {
  if (!text || text.trim() === '') {
    throw new Error('Task text cannot be empty');
  }

  return this.add({
    id: crypto.randomUUID(),
    text: text.trim(),
    done: false,
    priority: 'normal',
    createdAt: new Date()
  });
};

// Use custom method
tasks
  .addTask('Buy milk')
  .addTask('Walk dog');
```

---

## Common Patterns

### Pattern 1: Single Add

```javascript
collection.add(item);
```

### Pattern 2: Chained Adds

```javascript
collection
  .add(item1)
  .add(item2)
  .add(item3);
```

### Pattern 3: Conditional Add

```javascript
if (condition) {
  collection.add(item);
}
```

### Pattern 4: Add with ID

```javascript
collection.add({
  id: Date.now(),
  ...data
});
```

### Pattern 5: Add in Loop

```javascript
items.forEach(item => {
  collection.add(processItem(item));
});
```

---

## add() vs push()

| Feature | add() | push() |
|---------|-------|--------|
| Syntax | `collection.add(item)` | `collection.items.push(item)` |
| Chainable | Yes | No |
| Returns | Collection | New length |
| Semantic | More readable | Standard JS |
| Multiple items | One at a time | Can push many |

---

## Key Takeaways

✅ **Chainable** - Returns collection for fluent API
✅ **Semantic** - "add" is clearer than "push"
✅ **Concise** - Shorter syntax than `items.push()`
✅ **Reactive** - Triggers effects automatically
✅ **Fluent** - Enables builder pattern
✅ **Simple** - Just adds to the end

---

## What's Next?

- **`collection.remove()`** - Remove items from collection
- **`collection.update()`** - Update items in collection
- **`collection.clear()`** - Clear all items

---

## Summary

`collection.add()` **adds an item to the collection and returns the collection for chaining**. It's a semantic, chainable wrapper around array push.

**The magic formula:**
```
collection
  .add(item1)
  .add(item2)
  =
Fluent item addition
──────────────────
Chainable, semantic, reactive
```

Think of it as **the fluent push** — add items with a clean, readable syntax that chains naturally. Perfect for builder patterns and expressive code.
