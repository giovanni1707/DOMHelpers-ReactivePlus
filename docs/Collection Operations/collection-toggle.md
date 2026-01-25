# `collection.toggle()` - Toggle Boolean Field on Single Item

## Quick Start (30 seconds)

```javascript
// Create collection
const todos = collection([
  { id: 1, text: 'Buy milk', done: false },
  { id: 2, text: 'Walk dog', done: true },
  { id: 3, text: 'Read book', done: false }
]);

// Toggle 'done' field of first item
todos.toggle(item => item.id === 1);

console.log(todos.items[0].done); // true

// Toggle custom field
const settings = collection([
  { id: 'notifications', enabled: false },
  { id: 'darkMode', enabled: false }
]);

settings.toggle(s => s.id === 'darkMode', 'enabled');
console.log(settings.items[1].enabled); // true
```

**That's it.** Toggle a boolean field on the first matching item. Defaults to `done` but works with any boolean field.

---

## What is `collection.toggle()`?

`collection.toggle()` **finds the first item matching a predicate and toggles a boolean field** (from true to false or false to true). It defaults to toggling the `done` field but can toggle any boolean property.

Think of it as **smart toggle** — find an item and flip a boolean switch.

**In practical terms:** Use `toggle()` for checkboxes, switches, completion states, or any boolean property that needs to flip between true/false.

---

## Syntax

```javascript
// Toggle 'done' field (default)
collection.toggle(item => item.id === 5);

// Toggle custom field
collection.toggle(item => item.id === 5, 'active');

// By value/reference
collection.toggle(itemReference, 'enabled');

// Returns collection (chainable)
const result = collection.toggle(predicate, field);
```

**Parameters:**
- `predicate` - Either:
  - **Function**: `(item) => boolean` - Returns true for item to toggle
  - **Value**: Direct item reference to toggle
- `field` (optional) - String name of boolean field to toggle (default: `'done'`)

**Returns:**
- The collection itself (for chaining)

**Important:**
- Toggles only **first match**
- Default field is **'done'**
- Sets `field = !field`
- **Chainable** — returns collection
- Safe if **not found**

---

## Summary

`collection.toggle()` **toggles a boolean field on the first matching item**. Defaults to `done` field but works with any boolean property.

**The magic formula:**
```
collection.toggle(item => item.id === 5, 'active')
  =
Find item, flip boolean
────────────────────────
true ↔ false, chainable, reactive
```

Think of it as **smart toggle** — find an item and flip a boolean switch with one call. Perfect for checkboxes, switches, and toggle UI elements.
