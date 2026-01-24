# `collection.filter()` - Filter Items to New Array

## Quick Start (30 seconds)

```javascript
const tasks = collection([
  { id: 1, done: false },
  { id: 2, done: true },
  { id: 3, done: false }
]);

// Filter to new array
const incomplete = tasks.filter(task => !task.done);
console.log(incomplete); // [{ id: 1, ... }, { id: 3, ... }]

// Original collection unchanged
console.log(tasks.length); // 3

// Use in effects
effect(() => {
  const active = tasks.filter(t => !t.done);
  console.log('Active tasks:', active.length);
});
```

**That's it.** Filter items to a new array. Returns array, not collection.

---

## What is `collection.filter()`?

`collection.filter()` **creates a new array with all items that pass a predicate function**. It's a direct wrapper around the standard array `filter()` method.

**Syntax:**
```javascript
const newArray = collection.filter((item, index, array) => {
  return condition;
});
```

**Parameters:**
- `predicate` - Function `(item, index, array) => boolean`

**Returns:**
- New **array** (not a collection)

**Key Points:**
- Returns **new array**, not collection
- Original collection **unchanged**
- **Reactive** when used in effects
- Wrapper around `items.filter()`

---

## Summary

`collection.filter()` **filters items to a new array**. Standard array filter with reactivity.

```javascript
const filtered = collection.filter(item => condition)
```
