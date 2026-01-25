# `collection.copyWithin()` - Copy Elements Within Array

## Quick Start (30 seconds)

```javascript
const items = collection([1, 2, 3, 4, 5]);

// Copy sequence to another position
items.copyWithin(0, 3, 5);  // Copy items 3-5 to index 0
console.log(items.items); // [4, 5, 3, 4, 5]

// Another example
const arr = collection(['a', 'b', 'c', 'd', 'e']);
arr.copyWithin(2, 0, 2);  // Copy items 0-2 to index 2
console.log(arr.items); // ['a', 'b', 'a', 'b', 'e']
```

**That's it.** Copy a sequence of elements within the collection. Triggers reactivity.

---

## What is `collection.copyWithin()`?

`collection.copyWithin()` **copies a sequence of array elements to another position in the same array and returns the collection for chaining**. It's a wrapper around array `copyWithin()` that triggers reactivity.

**Syntax:**
```javascript
collection.copyWithin(target, start, end);
```

**Parameters:**
- `target` - Index to copy to
- `start` - Start index of copy source
- `end` (optional) - End index, exclusive (default: length)

**Returns:**
- The collection itself (for chaining)

**Key Points:**
- Modifies **in place**
- **Chainable** (returns collection)
- Triggers **reactivity**
- Wrapper around `items.copyWithin()`

---

## Summary

`collection.copyWithin()` **copies elements within the array**. Chainable wrapper with reactivity.

```javascript
collection.copyWithin(0, 3, 5)
```
