# `collection.indexOf()` - Get Index of Item

## Quick Start (30 seconds)

```javascript
const items = collection(['apple', 'banana', 'cherry']);

// Find index of 'banana'
const index = items.indexOf('banana');
console.log(index); // 1

// Not found returns -1
const notFound = items.indexOf('orange');
console.log(notFound); // -1

// Works with objects too
const users = collection([
  { id: 1, name: 'Alice' },
  { id: 2, name: 'Bob' }
]);

const alice = users.items[0];
const index2 = users.indexOf(alice);
console.log(index2); // 0
```

**That's it.** Get the index of an item in the collection. Returns -1 if not found.

---

## What is `collection.indexOf()`?

`collection.indexOf()` **returns the index of an item in the collection, or -1 if not found**. It's a direct wrapper around the standard array `indexOf()` method.

**Syntax:**
```javascript
const index = collection.indexOf(item);
```

**Parameters:**
- `item` - Item to find (uses strict equality ===)

**Returns:**
- Index number (0-based) or `-1` if not found

**Key Points:**
- Uses **strict equality** (===)
- Returns **-1** if not found
- **Reactive** - tracks dependency
- Wrapper around `items.indexOf()`

---

## Summary

`collection.indexOf()` **returns the index of an item or -1 if not found**. Standard array indexOf with reactivity.

```javascript
const index = collection.indexOf(item)
```
