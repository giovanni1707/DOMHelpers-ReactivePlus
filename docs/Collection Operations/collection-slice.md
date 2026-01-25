# `collection.slice()` - Get Slice of Items

## Quick Start (30 seconds)

```javascript
const items = collection(['a', 'b', 'c', 'd', 'e']);

// Get slice from index 1 to 3 (exclusive)
const slice = items.slice(1, 3);
console.log(slice); // ['b', 'c']

// Original unchanged
console.log(items.items); // ['a', 'b', 'c', 'd', 'e']

// Negative indices
const last2 = items.slice(-2);
console.log(last2); // ['d', 'e']

// Copy entire array
const copy = items.slice();
console.log(copy); // ['a', 'b', 'c', 'd', 'e']
```

**That's it.** Extract a portion of the collection to a new array. Original collection unchanged.

---

## What is `collection.slice()`?

`collection.slice()` **returns a shallow copy of a portion of the collection as a new array**. It's a direct wrapper around the standard array `slice()` method.

**Syntax:**
```javascript
const newArray = collection.slice(start, end);
```

**Parameters:**
- `start` (optional) - Start index (default: 0)
- `end` (optional) - End index, exclusive (default: length)

**Returns:**
- New **array** (not a collection)

**Key Points:**
- Returns **new array**, not collection
- Original collection **unchanged**
- Supports **negative indices**
- Wrapper around `items.slice()`

---

## Summary

`collection.slice()` **returns a slice as a new array**. Standard array slice.

```javascript
const slice = collection.slice(1, 3)
```
