# `collection.splice()` - Add/Remove Items at Position

## Quick Start (30 seconds)

```javascript
const items = collection(['a', 'b', 'c', 'd', 'e']);

// Remove 2 items starting at index 1
items.splice(1, 2);
console.log(items.items); // ['a', 'd', 'e']

// Insert items at index 1
items.splice(1, 0, 'x', 'y', 'z');
console.log(items.items); // ['a', 'x', 'y', 'z', 'd', 'e']

// Replace items (remove 2, add 1)
items.splice(1, 2, 'b');
console.log(items.items); // ['a', 'b', 'z', 'd', 'e']
```

**That's it.** Add or remove items at any position. Triggers reactivity and is chainable.

---

## What is `collection.splice()`?

`collection.splice()` **adds and/or removes items at a specific position in the collection and returns the collection for chaining**. Unlike array `splice()` which returns removed items, this returns the collection.

**Syntax:**
```javascript
collection.splice(start, deleteCount, ...itemsToAdd);
```

**Parameters:**
- `start` - Index to start at
- `deleteCount` - Number of items to remove
- `...items` - Items to insert

**Returns:**
- The collection itself (for chaining)

**Key Points:**
- Can **add and remove** in one call
- **Chainable** (returns collection, not removed items)
- Triggers **reactivity**
- Wrapper around `items.splice()`

---

## Summary

`collection.splice()` **adds/removes items at a position**. Chainable wrapper around array splice.

```javascript
collection.splice(1, 2, 'x', 'y')
```
