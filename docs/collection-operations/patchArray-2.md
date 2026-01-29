# `patchArray()` - Manually Patch Array Property for Reactivity

## Quick Start (30 seconds)

```javascript
// Create state with array property
const app = state({
  items: [1, 2, 3]
});

// After creating state, if array methods aren't reactive
app.items.push(4);  // Might not trigger effects

// Manually patch the array
patchArray(app, 'items');

// Now array methods trigger reactivity
effect(() => {
  console.log('Items:', app.items.length);
});
// Output: Items: 4

app.items.push(5);
// Output: Items: 5
```

**That's it.** Manually patch an array property to make array methods (push, pop, etc.) trigger reactivity.

---

## What is `patchArray()`?

`patchArray()` **manually patches an array property on a reactive state object to ensure array methods trigger reactivity**. This is typically done automatically, but this function allows manual patching when needed.

**Syntax:**
```javascript
patchArray(stateObject, 'propertyName');
```

**Parameters:**
- `state` - Reactive state object
- `key` - String property name of the array

**Returns:**
- `undefined` (side-effect function)

**Key Points:**
- Patches array **mutating methods** (push, pop, splice, etc.)
- Makes them **trigger reactivity**
- Usually **automatic** with `collection()` or `state()`
- Use for **edge cases** or **dynamically added arrays**

---

## When to Use

**Rarely needed** - modern reactive systems patch arrays automatically. Use when:
- Adding array properties **after** state creation
- Working with **legacy code**
- Arrays not automatically patched
- Debugging reactivity issues

**Don't use when:**
- Using `collection()` (already patched)
- Arrays created with initial state (auto-patched)
- Not experiencing reactivity problems

---

## Summary

`patchArray()` **manually patches an array property for reactivity**. Rarely needed in modern code.

```javascript
patchArray(state, 'items')
```

**Alias:** `patchReactiveArray()` (legacy name, same function)
