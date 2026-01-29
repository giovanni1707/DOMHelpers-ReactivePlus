# `collection.includes()` - Check if Item Exists

## Quick Start (30 seconds)

```javascript
const items = collection(['apple', 'banana', 'cherry']);

// Check if item exists
console.log(items.includes('banana')); // true
console.log(items.includes('orange')); // false

// Works with objects (by reference)
const users = collection([
  { id: 1, name: 'Alice' },
  { id: 2, name: 'Bob' }
]);

const alice = users.items[0];
console.log(users.includes(alice)); // true

const stranger = { id: 3, name: 'Charlie' };
console.log(users.includes(stranger)); // false
```

**That's it.** Check if a collection contains an item. Returns boolean.

---

## What is `collection.includes()`?

`collection.includes()` **returns true if the collection contains the item, false otherwise**. It's a direct wrapper around the standard array `includes()` method.

**Syntax:**
```javascript
const exists = collection.includes(item);
```

**Parameters:**
- `item` - Item to check (uses strict equality ===)

**Returns:**
- `true` if found, `false` otherwise

**Key Points:**
- Uses **strict equality** (===)
- Returns **boolean**
- **Reactive** - tracks dependency
- Wrapper around `items.includes()`

---

## Summary

`collection.includes()` **returns true if collection contains the item**. Standard array includes with reactivity.

```javascript
if (collection.includes(item)) { ... }
```
