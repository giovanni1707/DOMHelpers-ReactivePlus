# `collection.shift()` - Remove First Item

## Quick Start (30 seconds)

```javascript
const items = collection(['a', 'b', 'c']);

// Remove and return first item
const first = items.shift();
console.log(first); // 'a'
console.log(items.items); // ['b', 'c']

// Returns undefined if empty
const empty = collection([]);
const nothing = empty.shift();
console.log(nothing); // undefined

// Use in loops (queue pattern)
while (!items.isEmpty()) {
  const item = items.shift();
  console.log(item);
}
// Output: b, c
```

**That's it.** Remove and return the first item. Triggers reactivity.

---

## What is `collection.shift()`?

`collection.shift()` **removes and returns the first item from the collection, or undefined if empty**. It's a direct wrapper around the standard array `shift()` method.

**Syntax:**
```javascript
const item = collection.shift();
```

**Parameters:** None

**Returns:**
- The removed item or `undefined` if empty

**Key Points:**
- Removes from **start** of array
- **Not chainable** (returns the item, not collection)
- Triggers **reactivity**
- Returns `undefined` if empty

---

## Summary

`collection.shift()` **removes and returns the first item**. Standard array shift with reactivity.

```javascript
const first = collection.shift()
```
