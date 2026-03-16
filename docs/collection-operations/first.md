[![Sponsor](https://img.shields.io/badge/Sponsor-💖-pink)](https://github.com/sponsors/giovanni1707)

[![Sponsor](https://img.shields.io/badge/Sponsor-PayPal-blue?logo=paypal)](https://paypal.me/GiovanniSylvain)

# `collection.first` - Get First Item (Property)

## Quick Start (30 seconds)

```javascript
const items = collection(['a', 'b', 'c']);

// Get first item
console.log(items.first); // 'a'

// Reactive in effects
effect(() => {
  console.log('First:', items.first);
});
// Output: First: a

items.unshift('z');
// Output: First: z

// Returns undefined if empty
const empty = collection([]);
console.log(empty.first); // undefined
```

**That's it.** Get the first item in the collection. Reactive property.

---

## What is `collection.first`?

`collection.first` **is a reactive getter property that returns the first item in the collection, or undefined if empty**. It's equivalent to `items[0]`.

**Syntax:**
```javascript
const firstItem = collection.first;
```

**Type:** Getter property (not a method)

**Returns:**
- First item or `undefined` if empty

**Key Points:**
- **Getter property**, not a method (no parentheses)
- **Reactive** - changes trigger effects
- Returns `undefined` for empty collections
- Equivalent to `collection.items[0]`

---

## Summary

`collection.first` **returns the first item or undefined**. Reactive property.

```javascript
const first = collection.first;
```