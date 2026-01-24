# `collection.removeWhere()` - Remove ALL Matching Items

## Quick Start (30 seconds)

```javascript
// Create collection
const tasks = collection([
  { id: 1, done: false },
  { id: 2, done: true },
  { id: 3, done: false },
  { id: 4, done: true },
  { id: 5, done: true }
]);

// Remove ALL completed tasks
tasks.removeWhere(task => task.done);

console.log(tasks.length); // 2 (only incomplete tasks remain)

// Remove multiple items by condition
const numbers = collection([1, 2, 3, 4, 5, 6, 7, 8, 9, 10]);
numbers.removeWhere(n => n % 2 === 0);  // Remove all even numbers

console.log(numbers.items); // [1, 3, 5, 7, 9]
```

**That's it.** Remove all items matching a predicate. Unlike `remove()` which removes only the first match, `removeWhere()` removes **all** matching items.

---

## What is `collection.removeWhere()`?

`collection.removeWhere()` **removes ALL items that match a predicate function**. It iterates through the collection and removes every item for which the predicate returns true.

Think of it as **bulk removal** — find all matches and remove them all at once.

**In practical terms:** Use `removeWhere()` to filter out multiple items by condition, like clearing all completed tasks or removing all items below a threshold.

---

## Syntax

```javascript
// Remove all matching items
collection.removeWhere(item => condition);

// Example
collection.removeWhere(item => item.done);
collection.removeWhere((item, index) => index > 5);

// Returns collection (chainable)
const result = collection.removeWhere(predicate);  // result === collection
```

**Parameters:**
- `predicate` - Function `(item, index) => boolean` - Returns true for items to remove

**Returns:**
- The collection itself (for chaining)

**Important:**
- Removes **ALL matches** (not just first)
- Iterates **backwards** to avoid index issues
- **Chainable** — returns collection
- Triggers **reactivity**
- Predicate must be a **function** (no direct values)

---

## removeWhere() vs remove()

| Feature | remove() | removeWhere() |
|---------|----------|---------------|
| Matches | First only | All matches |
| Predicate | Function or value | Function only |
| Use case | Remove specific item | Remove all matching |
| Performance | Stops after first | Checks all items |

---

## Summary

`collection.removeWhere()` **removes ALL items matching a predicate**. Perfect for bulk removal operations.

**The magic formula:**
```
collection.removeWhere(item => item.done)
  =
Remove all matching items
──────────────────────────
Bulk removal, chainable, reactive
```

Think of it as **bulk removal** — filter out all items matching a condition in one call. Perfect for clearing categories, removing old items, or filtering by criteria.
