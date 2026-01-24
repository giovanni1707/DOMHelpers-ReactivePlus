# `collection.toggleAll()` - Toggle Boolean Field on ALL Matching Items

## Quick Start (30 seconds)

```javascript
// Create collection
const todos = collection([
  { id: 1, text: 'Buy milk', done: false },
  { id: 2, text: 'Walk dog', done: false },
  { id: 3, text: 'Read book', done: true }
]);

// Toggle 'done' on ALL incomplete tasks
const count = todos.toggleAll(item => !item.done);

console.log(count); // 2 (number of items toggled)
console.log(todos.items[0].done); // true
console.log(todos.items[1].done); // true
console.log(todos.items[2].done); // true (was already true)

// Toggle custom field on all items
const tasks = collection([
  { id: 1, urgent: false },
  { id: 2, urgent: false }
]);

tasks.toggleAll(item => true, 'urgent');
// All tasks now have urgent: true
```

**That's it.** Toggle a boolean field on ALL matching items. Unlike `toggle()` which affects only the first match, `toggleAll()` toggles **all** matching items.

---

## What is `collection.toggleAll()`?

`collection.toggleAll()` **finds ALL items matching a predicate and toggles a boolean field on each**. It defaults to toggling the `done` field but can toggle any boolean property.

Think of it as **bulk toggle** — flip a boolean switch on all matching items at once.

**In practical terms:** Use `toggleAll()` for "mark all as done", "select all", "enable all matching", or any bulk boolean toggle operation.

---

## Syntax

```javascript
// Toggle 'done' field (default) on all matches
const count = collection.toggleAll(item => condition);

// Toggle custom field on all matches
const count = collection.toggleAll(item => condition, 'active');

// Returns count of toggled items
console.log(count); // Number of items toggled
```

**Parameters:**
- `predicate` - Function `(item, index) => boolean` - Returns true for items to toggle
- `field` (optional) - String name of boolean field to toggle (default: `'done'`)

**Returns:**
- **Number** of items toggled (not the collection!)

**Important:**
- Toggles **ALL matches** (not just first)
- Default field is **'done'**
- Returns **count**, not collection (not chainable)
- Safe if **no matches**
- Predicate must be **function**

---

## toggleAll() vs toggle()

| Feature | toggle() | toggleAll() |
|---------|----------|-------------|
| Matches | First only | All matches |
| Returns | Collection | Count number |
| Chainable | Yes | No |
| Use case | Toggle one item | Toggle all matching |

---

## Summary

`collection.toggleAll()` **toggles a boolean field on ALL matching items** and returns the count of items toggled.

**The magic formula:**
```
collection.toggleAll(item => item.active, 'enabled')
  =
Find all matches, flip boolean
────────────────────────────────
true ↔ false on all, returns count
```

Think of it as **bulk toggle** — flip a boolean switch on all matching items. Perfect for "select all", "mark all done", or bulk enable/disable operations. Returns count instead of collection.
