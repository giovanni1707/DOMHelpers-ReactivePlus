# `patchReactiveArray()` - Legacy Array Patching Function

## Quick Start (30 seconds)

```javascript
// Legacy function (use patchArray instead)
const app = state({
  items: [1, 2, 3]
});

// Patch array for reactivity
patchReactiveArray(app, 'items');

// Now reactive
effect(() => {
  console.log('Count:', app.items.length);
});

app.items.push(4);
// Output: Count: 4
```

**Note:** This is a **legacy alias**. Use `patchArray()` instead for new code.

---

## What is `patchReactiveArray()`?

`patchReactiveArray()` **is the legacy name for `patchArray()`**. It manually patches an array property to make array methods trigger reactivity.

**Syntax:**
```javascript
patchReactiveArray(stateObject, 'propertyName');
```

**Parameters:**
- `state` - Reactive state object
- `key` - String property name of the array

**Returns:**
- `undefined` (side-effect function)

---

## Migration

**Old (Legacy):**
```javascript
patchReactiveArray(state, 'items');
```

**New (Preferred):**
```javascript
patchArray(state, 'items');
```

Both work identically. Use `patchArray()` for consistency with the modern API.

---

## Summary

`patchReactiveArray()` **is the legacy alias for `patchArray()`**. Use `patchArray()` instead.

```javascript
// Legacy
patchReactiveArray(state, 'items')

// Modern
patchArray(state, 'items')
```
