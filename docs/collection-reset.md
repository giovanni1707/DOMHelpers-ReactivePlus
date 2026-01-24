# `collection.reset()` - Reset Collection with New Items

## Quick Start (30 seconds)

```javascript
// Create collection
const todos = collection([
  { id: 1, text: 'Old task 1', done: false },
  { id: 2, text: 'Old task 2', done: true }
]);

console.log(todos.length); // 2

// Reset with new items
todos.reset([
  { id: 1, text: 'New task 1', done: false },
  { id: 2, text: 'New task 2', done: false },
  { id: 3, text: 'New task 3', done: false }
]);

console.log(todos.length); // 3
console.log(todos.items[0].text); // 'New task 1'

// Reset to empty
todos.reset([]);
console.log(todos.length); // 0

// Chainable
todos
  .reset([{ id: 1, text: 'Fresh start', done: false }])
  .add({ id: 2, text: 'Another task', done: false });
```

**That's it.** Replace all items in a collection with a new array. Perfect for resetting state or loading new data.

---

## What is `collection.reset()`?

`collection.reset()` **clears the collection and replaces it with a new array of items**. It's a combination of `clear()` + `push(...newItems)` in one atomic operation.

Think of it as **the replace operation** — swap out all current items for a new set.

**In practical terms:** Use `reset()` when loading new data from an API, resetting to initial state, or completely replacing collection contents.

---

## Syntax

```javascript
// Reset with new items
collection.reset(newItemsArray);

// Reset to empty (same as clear)
collection.reset([]);
collection.reset();  // Empty array by default

// Chainable
collection
  .reset(newItems)
  .add(additionalItem);

// Returns collection
const result = collection.reset(newItems);  // result === collection
```

**Parameters:**
- `newItems` (optional) - Array of new items (default: `[]`)

**Returns:**
- The collection itself (for chaining)

**Important:**
- **Clears** all existing items first
- **Adds** all new items
- Defaults to **empty array** if no parameter
- **Chainable** — returns collection
- Triggers **reactivity** once
- More efficient than clearing and adding manually

---

## reset() vs clear() + push()

| Feature | reset() | clear() + push() |
|---------|---------|------------------|
| Operations | Single atomic operation | Two separate operations |
| Reactivity | Triggers once | Triggers twice |
| Syntax | `reset(newItems)` | `clear(); items.push(...newItems)` |
| Performance | Slightly faster | Slightly slower |
| Chainability | Chainable | clear() chainable, push() not |

---

## Summary

`collection.reset()` **clears the collection and replaces it with new items** in one atomic operation. Chainable and reactive.

**The magic formula:**
```
collection.reset([newItem1, newItem2])
  =
Clear all + Replace with new
──────────────────────────────
Atomic swap, chainable, reactive
```

Think of it as **the replace operation** — completely replace collection contents in one call. Perfect for loading new data, resetting state, or swapping entire datasets efficiently.
