# `collection.at()` - Get Item at Index

## Quick Start (30 seconds)

```javascript
const items = collection(['a', 'b', 'c', 'd', 'e']);

// Get item at index
console.log(items.at(0)); // 'a'
console.log(items.at(2)); // 'c'

// Negative indices count from end
console.log(items.at(-1)); // 'e' (last item)
console.log(items.at(-2)); // 'd' (second to last)

// Out of bounds returns undefined
console.log(items.at(99)); // undefined
```

**That's it.** Get an item by index, with support for negative indices.

---

## What is `collection.at()`?

`collection.at()` **returns the item at a specific index, supporting negative indices to count from the end**. It's a direct wrapper around the array bracket accessor `items[index]`.

**Syntax:**
```javascript
const item = collection.at(index);
const item = collection.at(-1); // Last item
```

**Parameters:**
- `index` - Numeric index (negative counts from end)

**Returns:**
- Item at that index or `undefined`

**Key Points:**
- Supports **negative indices** (-1 = last, -2 = second to last)
- Returns **undefined** if out of bounds
- **Reactive** - tracks dependency
- Wrapper around `items[index]`

---

## Summary

`collection.at()` **returns the item at a specific index**. Supports negative indices.

```javascript
const item = collection.at(0)     // First
const last = collection.at(-1)    // Last
```
