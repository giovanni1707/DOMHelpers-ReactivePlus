# `collection.last` - Get Last Item (Property)

## Quick Start (30 seconds)

```javascript
const items = collection(['a', 'b', 'c']);

// Get last item
console.log(items.last); // 'c'

// Reactive in effects
effect(() => {
  console.log('Last:', items.last);
});
// Output: Last: c

items.push('d');
// Output: Last: d

// Returns undefined if empty
const empty = collection([]);
console.log(empty.last); // undefined
```

**That's it.** Get the last item in the collection. Reactive property.

---

## What is `collection.last`?

`collection.last` **is a reactive getter property that returns the last item in the collection, or undefined if empty**. It's equivalent to `items[items.length - 1]`.

**Syntax:**
```javascript
const lastItem = collection.last;
```

**Type:** Getter property (not a method)

**Returns:**
- Last item or `undefined` if empty

**Key Points:**
- **Getter property**, not a method (no parentheses)
- **Reactive** - changes trigger effects
- Returns `undefined` for empty collections
- Equivalent to `collection.items[items.length - 1]`

---

## Summary

`collection.last` **returns the last item or undefined**. Reactive property.

```javascript
const last = collection.last;
```
