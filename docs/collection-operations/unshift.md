# `collection.unshift()` - Add Items to Start

## Quick Start (30 seconds)

```javascript
const items = collection(['c', 'd']);

// Add single item to start
items.unshift('b');
console.log(items.items); // ['b', 'c', 'd']

// Add multiple items to start
items.unshift('a', 'z');
console.log(items.items); // ['a', 'z', 'b', 'c', 'd']

// Chainable - returns collection
items
  .unshift('x')
  .add('e');

console.log(items.first); // 'x'
```

**That's it.** Add one or more items to the start of the collection. Triggers reactivity and is chainable.

---

## What is `collection.unshift()`?

`collection.unshift()` **adds one or more items to the start of the collection and returns the collection for chaining**. Unlike array `unshift()` which returns new length, this returns the collection.

**Syntax:**
```javascript
collection.unshift(item1, item2, ...itemN);
```

**Parameters:**
- `...items` - One or more items to add

**Returns:**
- The collection itself (for chaining)

**Key Points:**
- Adds to **start** of array
- **Chainable** (returns collection, not length)
- Triggers **reactivity**
- Can add multiple items at once

---

## Summary

`collection.unshift()` **adds items to the start**. Chainable wrapper around array unshift.

```javascript
collection.unshift(item1, item2)
```
