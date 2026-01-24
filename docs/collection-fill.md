# `collection.fill()` - Fill with Value

## Quick Start (30 seconds)

```javascript
const items = collection([1, 2, 3, 4, 5]);

// Fill all with same value
items.fill(0);
console.log(items.items); // [0, 0, 0, 0, 0]

// Fill range
const arr = collection([1, 2, 3, 4, 5]);
arr.fill(9, 2, 4);  // Fill from index 2 to 4 (exclusive)
console.log(arr.items); // [1, 2, 9, 9, 5]

// Chainable
items
  .fill(1)
  .push(2);
```

**That's it.** Fill all or part of the collection with a value. Triggers reactivity.

---

## What is `collection.fill()`?

`collection.fill()` **fills all or part of the collection with a static value and returns the collection for chaining**. It's a wrapper around array `fill()` that triggers reactivity.

**Syntax:**
```javascript
collection.fill(value, start, end);
```

**Parameters:**
- `value` - Value to fill with
- `start` (optional) - Start index (default: 0)
- `end` (optional) - End index, exclusive (default: length)

**Returns:**
- The collection itself (for chaining)

**Key Points:**
- Fills **in place** (modifies collection)
- **Chainable** (returns collection)
- Triggers **reactivity**
- Wrapper around `items.fill()`

---

## Summary

`collection.fill()` **fills with a value**. Chainable wrapper around array fill with reactivity.

```javascript
collection.fill(0, 2, 5)
```
