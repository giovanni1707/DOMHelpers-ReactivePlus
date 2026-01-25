# `collection.pop()` - Remove Last Item

## Quick Start (30 seconds)

```javascript
const items = collection(['a', 'b', 'c']);

// Remove and return last item
const last = items.pop();
console.log(last); // 'c'
console.log(items.items); // ['a', 'b']

// Returns undefined if empty
const empty = collection([]);
const nothing = empty.pop();
console.log(nothing); // undefined

// Use in loops
while (!items.isEmpty()) {
  const item = items.pop();
  console.log(item);
}
// Output: b, a
```

**That's it.** Remove and return the last item. Triggers reactivity.

---

## What is `collection.pop()`?

`collection.pop()` **removes and returns the last item from the collection, or undefined if empty**. It's a direct wrapper around the standard array `pop()` method.

**Syntax:**
```javascript
const item = collection.pop();
```

**Parameters:** None

**Returns:**
- The removed item or `undefined` if empty

**Key Points:**
- Removes from **end** of array
- **Not chainable** (returns the item, not collection)
- Triggers **reactivity**
- Returns `undefined` if empty

---

## Summary

`collection.pop()` **removes and returns the last item**. Standard array pop with reactivity.

```javascript
const last = collection.pop()
```
