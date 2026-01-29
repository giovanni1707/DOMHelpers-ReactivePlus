# `collection.reverse()` - Reverse Array In Place

## Quick Start (30 seconds)

```javascript
const items = collection(['a', 'b', 'c', 'd']);

// Reverse in place
items.reverse();
console.log(items.items); // ['d', 'c', 'b', 'a']

// Chainable
items
  .reverse()  // Back to ['a', 'b', 'c', 'd']
  .add('e')
  .reverse(); // ['e', 'd', 'c', 'b', 'a']

console.log(items.first); // 'e'
```

**That's it.** Reverse the collection in place. Triggers reactivity and is chainable.

---

## What is `collection.reverse()`?

`collection.reverse()` **reverses the order of items in the array in place and returns the collection for chaining**. It's a wrapper around array `reverse()` that triggers reactivity.

**Syntax:**
```javascript
collection.reverse();
```

**Parameters:** None

**Returns:**
- The collection itself (for chaining)

**Key Points:**
- Reverses **in place** (modifies collection)
- **Chainable** (returns collection)
- Triggers **reactivity**
- Wrapper around `items.reverse()`

---

## Summary

`collection.reverse()` **reverses items in place**. Chainable wrapper around array reverse with reactivity.

```javascript
collection.reverse()
```
