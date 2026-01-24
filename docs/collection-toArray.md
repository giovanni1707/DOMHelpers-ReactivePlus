# `collection.toArray()` - Convert to Plain Array

## Quick Start (30 seconds)

```javascript
const items = collection(['a', 'b', 'c']);

// Convert to plain array
const plainArray = items.toArray();
console.log(plainArray); // ['a', 'b', 'c']
console.log(Array.isArray(plainArray)); // true

// It's a shallow copy
plainArray.push('d');
console.log(items.length); // 3 (unchanged)
console.log(plainArray.length); // 4

// Use for serialization
const json = JSON.stringify(items.toArray());
```

**That's it.** Get a shallow copy of the items as a plain JavaScript array.

---

## What is `collection.toArray()`?

`collection.toArray()` **returns a shallow copy of the items as a plain JavaScript array**. It's equivalent to `[...items.items]` or `items.slice()`.

**Syntax:**
```javascript
const array = collection.toArray();
```

**Parameters:** None

**Returns:**
- Plain JavaScript array (shallow copy)

**Key Points:**
- Returns **plain array**, not reactive
- **Shallow copy** (original unchanged)
- Useful for **serialization**
- Equivalent to `[...items.items]`

---

## Summary

`collection.toArray()` **returns a plain array copy**. For serialization and external use.

```javascript
const array = collection.toArray()
```
