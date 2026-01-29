# `safeEffect()` - Effect with Error Handling

## Quick Start (30 seconds)

```javascript
const data = state({ apiUrl: '/api/data' });

// Effect with automatic error handling
safeEffect(() => {
  const response = fetch(data.apiUrl); // Might fail
  // Process response...
}, {
  onError: (error) => {
    console.error('Effect error:', error);
    Elements.update({
      errorMessage: { textContent: error.message }
    });
    // Effect continues working!
  }
});

data.apiUrl = '/invalid';
// Error caught, effect keeps running for next changes
```

**That's it.** Like `effect()` but with built-in error handling. Errors are caught and effects continue working instead of breaking.

---

## What is `safeEffect()`?

`safeEffect()` is **effect() with error boundaries**. It wraps effect functions in try-catch blocks so errors don't break your reactive system.

Think of it as **a safety net for effects** — if something goes wrong in an effect, the error is caught, reported, and the effect keeps working.

**In practical terms:** Use `safeEffect()` when effects might fail (API calls, DOM operations, parsing) and you don't want one error to stop reactivity.

---

## Syntax

```javascript
// Effect with error handling
safeEffect(effectFunction, options);

// Parameters
safeEffect(
  () => { /* may throw */ },
  {
    onError: (error) => { /* handle error */ },
    priority: 'normal'  // or 'high', 'low'
  }
);
```

---

## Why Does This Exist?

### Without safeEffect() - Effect Breaks

```javascript
// ❌ Regular effect - error stops reactivity
const data = state({ value: 0 });

effect(() => {
  const result = riskyOperation(data.value); // Throws error
  Elements.update({
    result: { textContent: result }
  });
});

data.value = 1;
// ❌ Effect throws, breaks permanently
data.value = 2;
// ❌ Effect no longer runs!
```

### With safeEffect() - Effect Survives

```javascript
// ✅ safeEffect - error is caught, effect continues
const data = state({ value: 0 });

safeEffect(() => {
  const result = riskyOperation(data.value);
  Elements.update({
    result: { textContent: result }
  });
}, {
  onError: (error) => {
    console.error('Effect error:', error);
    Elements.update({
      result: { textContent: 'Error occurred' }
    });
  }
});

data.value = 1;
// ✨ Error caught, handled, effect keeps working

data.value = 2;
// ✨ Effect runs successfully
```

---

## Basic Examples

### Example 1: API Calls in Effects

```javascript
const user = state({ userId: 1 });

safeEffect(async () => {
  const response = await fetch(`/api/users/${user.userId}`);
  const userData = await response.json();

  Elements.update({
    userName: { textContent: userData.name },
    userEmail: { textContent: userData.email }
  });
}, {
  onError: (error) => {
    Elements.update({
      userError: {
        textContent: `Failed to load user: ${error.message}`,
        style: { display: 'block' }
      }
    });
  }
});
```

### Example 2: DOM Operations

```javascript
const config = state({ selector: '#content' });

safeEffect(() => {
  const element = querySelector(config.selector);
  element.textContent = 'Updated'; // Might be null
}, {
  onError: (error) => {
    console.error('DOM update failed:', error);
    // Effect still works for next valid selector
  }
});
```

### Example 3: JSON Parsing

```javascript
const data = state({ jsonString: '{}' });

safeEffect(() => {
  const parsed = JSON.parse(data.jsonString); // Might throw

  Elements.update({
    parsedData: { textContent: JSON.stringify(parsed, null, 2) }
  });
}, {
  onError: (error) => {
    Elements.update({
      parsedData: { textContent: 'Invalid JSON' }
    });
  }
});
```

---

## Key Takeaways

✅ **Error handling** - Catches errors in effects
✅ **Effect survives** - Continues working after errors
✅ **onError callback** - Custom error handling
✅ **Production-ready** - Safe for unreliable operations

---

## Summary

`safeEffect()` is **effect() with error boundaries**. Use it when effects might fail and you want reactivity to survive errors.
