# `collection.push()` - Add Items to End

## Quick Start (30 seconds)

```javascript
const items = collection(['a', 'b']);

// Add single item
items.push('c');
console.log(items.items); // ['a', 'b', 'c']

// Add multiple items
items.push('d', 'e', 'f');
console.log(items.items); // ['a', 'b', 'c', 'd', 'e', 'f']

// Chainable - returns collection
items
  .push('g')
  .push('h');

console.log(items.length); // 8
```

**That's it.** Add one or more items to the end of the collection. Triggers reactivity and is chainable.

---

## What is `collection.push()`?

`collection.push()` **adds one or more items to the end of the collection and returns the collection for chaining**. Unlike array `push()` which returns new length, this returns the collection.

**Syntax:**
```javascript
collection.push(item1, item2, ...itemN);
```

**Parameters:**
- `...items` - One or more items to add

**Returns:**
- The collection itself (for chaining)

**Key Points:**
- Adds to **end** of array
- **Chainable** (returns collection, not length)
- Triggers **reactivity**
- Can add multiple items at once

---

## Summary

`collection.push()` **adds items to the end**. Chainable wrapper around array push.

```javascript
collection.push(item1, item2, item3)
```
