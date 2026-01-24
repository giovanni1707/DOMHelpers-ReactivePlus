# `collection.find()` - Find First Matching Item

## Quick Start (30 seconds)

```javascript
const users = collection([
  { id: 1, name: 'Alice', active: true },
  { id: 2, name: 'Bob', active: false },
  { id: 3, name: 'Charlie', active: true }
]);

// Find first active user
const activeUser = users.find(user => user.active);
console.log(activeUser); // { id: 1, name: 'Alice', active: true }

// Find by ID
const user = users.find(u => u.id === 2);
console.log(user.name); // 'Bob'

// Not found returns undefined
const notFound = users.find(u => u.id === 999);
console.log(notFound); // undefined
```

**That's it.** Find the first item matching a predicate. Returns the item or undefined.

---

## What is `collection.find()`?

`collection.find()` **returns the first item that matches a predicate function, or undefined if no match is found**. It's a direct wrapper around the standard array `find()` method.

**Syntax:**
```javascript
const item = collection.find(item => condition);
const item = collection.find(value); // Also accepts direct value
```

**Parameters:**
- `predicate` - Function `(item) => boolean` or direct value

**Returns:**
- The matching item or `undefined`

**Key Points:**
- Returns **first match** only
- Returns **undefined** if not found
- **Reactive** - tracks dependency when used in effects
- Wrapper around `items.find()`

---

## Summary

`collection.find()` **returns the first item matching a predicate or undefined**. Standard array find with reactivity.

```javascript
const item = collection.find(item => item.id === 5)
```
