# `collection.forEach()` - Iterate Over Items

## Quick Start (30 seconds)

```javascript
const todos = collection([
  { id: 1, text: 'Buy milk', done: false },
  { id: 2, text: 'Walk dog', done: true }
]);

// Iterate over all items
todos.forEach((item, index) => {
  console.log(`${index}: ${item.text}`);
});
// Output:
// 0: Buy milk
// 1: Walk dog

// Chainable - returns collection
todos
  .forEach(item => console.log(item.text))
  .add({ id: 3, text: 'Read book', done: false });
```

**That's it.** Iterate over collection items with a callback. Chainable.

---

## What is `collection.forEach()`?

`collection.forEach()` **executes a callback function for each item in the collection and returns the collection for chaining**. Unlike array `forEach()`, this version is chainable.

**Syntax:**
```javascript
collection.forEach((item, index, array) => {
  // Do something with item
});
```

**Parameters:**
- `fn` - Callback function `(item, index, array) => void`

**Returns:**
- The collection itself (for chaining)

**Key Points:**
- Executes callback for **each item**
- **Chainable** (returns collection)
- **Reactive** when used in effects
- Wrapper around `items.forEach()`

---

## Summary

`collection.forEach()` **iterates over items with a callback**. Chainable version of array forEach.

```javascript
collection.forEach((item, index) => {
  console.log(item);
})
```
