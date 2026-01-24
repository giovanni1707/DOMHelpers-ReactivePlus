# `collection.clear()` - Clear All Items

## Quick Start (30 seconds)

```javascript
// Create collection
const todos = collection([
  { id: 1, text: 'Buy milk', done: false },
  { id: 2, text: 'Walk dog', done: true },
  { id: 3, text: 'Read book', done: false }
]);

console.log(todos.length); // 3

// Clear all items
todos.clear();

console.log(todos.length); // 0
console.log(todos.items); // []

// Chainable
todos
  .add({ id: 1, text: 'New task', done: false })
  .clear()
  .add({ id: 2, text: 'Another task', done: false });

console.log(todos.length); // 1
```

**That's it.** Remove all items from a collection in one call. Chainable and reactive.

---

## What is `collection.clear()`?

`collection.clear()` **removes all items from the collection by setting the array length to 0**. It's a semantic, chainable method for emptying collections.

Think of it as **the reset button** — instantly empty the collection while maintaining reactivity.

**In practical terms:** Use `clear()` when you need to remove all items at once, like resetting a list or clearing a cart.

---

## Syntax

```javascript
// Clear all items
collection.clear();

// Chainable
collection
  .clear()
  .add(newItem);

// Returns collection
const result = collection.clear();  // result === collection
```

**Parameters:**
- None

**Returns:**
- The collection itself (for chaining)

**Important:**
- Removes **all items**
- Sets `items.length = 0`
- **Chainable** — returns collection
- Triggers **reactivity**
- Faster than removing items one by one

---

## Summary

`collection.clear()` **removes all items from the collection** by setting the array length to 0. Fast, chainable, and reactive.

**The magic formula:**
```
collection.clear()
  =
Empty the collection
──────────────────
Fast, chainable, reactive
```

Think of it as **the reset button** — instantly remove all items with one clean call. Perfect for clearing lists, resetting state, or starting fresh.
