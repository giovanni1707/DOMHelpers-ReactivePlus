# `collection.updateWhere()` - Update ALL Matching Items

## Quick Start (30 seconds)

```javascript
// Create collection
const tasks = collection([
  { id: 1, done: false, priority: 'high' },
  { id: 2, done: false, priority: 'low' },
  { id: 3, done: true, priority: 'high' },
  { id: 4, done: false, priority: 'high' }
]);

// Update ALL high priority tasks
tasks.updateWhere(
  task => task.priority === 'high',
  { urgent: true, notified: true }
);

// All high priority tasks now have urgent and notified properties
console.log(tasks.items.filter(t => t.priority === 'high'));
// All have urgent: true and notified: true
```

**That's it.** Update all items matching a predicate. Unlike `update()` which updates only the first match, `updateWhere()` updates **all** matching items.

---

## What is `collection.updateWhere()`?

`collection.updateWhere()` **updates ALL items that match a predicate function by merging properties using Object.assign**. It applies the same updates to every matching item.

Think of it as **bulk update** — find all matches and update them all at once.

**In practical terms:** Use `updateWhere()` to apply the same changes to multiple items, like marking all incomplete tasks as urgent or updating all items in a category.

---

## Syntax

```javascript
// Update all matching items
collection.updateWhere(
  item => condition,
  { property: value }
);

// Example
collection.updateWhere(
  item => item.status === 'pending',
  { status: 'active', startedAt: new Date() }
);

// Returns collection (chainable)
const result = collection.updateWhere(predicate, updates);
```

**Parameters:**
- `predicate` - Function `(item, index) => boolean` - Returns true for items to update
- `updates` - Object with properties to merge into matching items

**Returns:**
- The collection itself (for chaining)

**Important:**
- Updates **ALL matches** (not just first)
- Uses **Object.assign** (shallow merge)
- **Chainable** — returns collection
- Triggers **reactivity**
- Predicate must be a **function**

---

## updateWhere() vs update()

| Feature | update() | updateWhere() |
|---------|----------|---------------|
| Matches | First only | All matches |
| Predicate | Function or value | Function only |
| Use case | Update specific item | Update all matching |
| Performance | Stops after first | Checks all items |

---

## Summary

`collection.updateWhere()` **updates ALL items matching a predicate** with the same property changes. Perfect for bulk update operations.

**The magic formula:**
```
collection.updateWhere(
  item => item.active,
  { verified: true }
)
  =
Update all matching items
──────────────────────────
Bulk update, chainable, reactive
```

Think of it as **bulk update** — apply the same changes to all items matching a condition. Perfect for batch operations, category updates, or status changes across multiple items.
