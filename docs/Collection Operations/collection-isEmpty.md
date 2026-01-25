# `collection.isEmpty()` - Check if Collection is Empty

## Quick Start (30 seconds)

```javascript
const items = collection([]);

// Check if empty
console.log(items.isEmpty()); // true

items.add('a');
console.log(items.isEmpty()); // false

// Use in conditionals
if (items.isEmpty()) {
  console.log('No items');
} else {
  console.log('Has items');
}

// Reactive in effects
effect(() => {
  const empty = items.isEmpty();
  document.getElementById('empty-state').style.display = 
    empty ? 'block' : 'none';
});
```

**That's it.** Check if the collection has zero items. Returns boolean.

---

## What is `collection.isEmpty()`?

`collection.isEmpty()` **returns true if the collection has no items, false otherwise**. It's equivalent to `items.length === 0`.

**Syntax:**
```javascript
const empty = collection.isEmpty();
```

**Parameters:** None

**Returns:**
- `true` if collection is empty, `false` otherwise

**Key Points:**
- **Method** (requires parentheses)
- **Reactive** - changes trigger effects
- Equivalent to `collection.length === 0`
- More semantic than length check

---

## Summary

`collection.isEmpty()` **returns true if collection has no items**. Semantic empty check.

```javascript
if (collection.isEmpty()) { ... }
```
