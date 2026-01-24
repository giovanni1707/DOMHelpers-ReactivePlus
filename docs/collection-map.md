# `collection.map()` - Transform Items to New Array

## Quick Start (30 seconds)

```javascript
const users = collection([
  { id: 1, name: 'Alice', age: 28 },
  { id: 2, name: 'Bob', age: 35 }
]);

// Map to new array
const names = users.map(user => user.name);
console.log(names); // ['Alice', 'Bob']

// Transform items
const formatted = users.map(u => ({
  label: u.name,
  value: u.id
}));
console.log(formatted);
// [{ label: 'Alice', value: 1 }, { label: 'Bob', value: 2 }]

// Use in effects
effect(() => {
  const html = users.map(u => `<li>${u.name}</li>`).join('');
  document.getElementById('list').innerHTML = html;
});
```

**That's it.** Transform collection items into a new array. Returns array, not collection.

---

## What is `collection.map()`?

`collection.map()` **transforms each item using a callback function and returns a new array**. It's a direct wrapper around the standard array `map()` method.

**Syntax:**
```javascript
const newArray = collection.map((item, index, array) => {
  return transformedValue;
});
```

**Parameters:**
- `fn` - Transformation function `(item, index, array) => newValue`

**Returns:**
- New **array** (not a collection)

**Key Points:**
- Returns **new array**, not collection
- **Not chainable** (returns array)
- **Reactive** when used in effects
- Wrapper around `items.map()`

---

## Summary

`collection.map()` **transforms items to a new array**. Standard array map with reactivity.

```javascript
const names = collection.map(item => item.name)
```
