# `collection.sort()` - Sort Array In Place

## Quick Start (30 seconds)

```javascript
const numbers = collection([3, 1, 4, 1, 5, 9, 2, 6]);

// Sort in place
numbers.sort((a, b) => a - b);
console.log(numbers.items); // [1, 1, 2, 3, 4, 5, 6, 9]

// Chainable
const users = collection([
  { name: 'Charlie', age: 35 },
  { name: 'Alice', age: 28 },
  { name: 'Bob', age: 42 }
]);

users
  .sort((a, b) => a.age - b.age)
  .forEach(u => console.log(u.name));
// Output: Alice, Charlie, Bob
```

**That's it.** Sort the collection in place. Triggers reactivity and is chainable.

---

## What is `collection.sort()`?

`collection.sort()` **sorts the items array in place using a compare function and returns the collection for chaining**. It's a wrapper around array `sort()` that triggers reactivity.

**Syntax:**
```javascript
collection.sort((a, b) => {
  return comparison;
});
```

**Parameters:**
- `compareFn` (optional) - Compare function `(a, b) => number`

**Returns:**
- The collection itself (for chaining)

**Key Points:**
- Sorts **in place** (modifies collection)
- **Chainable** (returns collection)
- Triggers **reactivity**
- Wrapper around `items.sort()`

---

## Summary

`collection.sort()` **sorts items in place**. Chainable wrapper around array sort with reactivity.

```javascript
collection.sort((a, b) => a - b)
```
