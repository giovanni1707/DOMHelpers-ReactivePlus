# `collection.length` - Get Collection Size (Property)

## Quick Start (30 seconds)

```javascript
const items = collection(['a', 'b', 'c']);

// Get length
console.log(items.length); // 3

// Reactive in effects
effect(() => {
  console.log('Count:', items.length);
});
// Output: Count: 3

items.add('d');
// Output: Count: 4

// Use in computed
const stats = state({ count: 0 });
computed(stats, {
  isEmpty: () => items.length === 0,
  hasItems: () => items.length > 0
});
```

**That's it.** Get the number of items in the collection. Reactive property.

---

## What is `collection.length`?

`collection.length` **is a reactive getter property that returns the number of items in the collection**. It's a proxy to `items.length`.

**Syntax:**
```javascript
const count = collection.length;
```

**Type:** Getter property (not a method)

**Returns:**
- Number of items in collection

**Key Points:**
- **Getter property**, not a method (no parentheses)
- **Reactive** - changes trigger effects
- Always returns current count
- Read-only (setting it doesn't work like arrays)

---

## Summary

`collection.length` **returns the number of items** in the collection. Reactive property.

```javascript
if (collection.length > 0) { ... }
```
