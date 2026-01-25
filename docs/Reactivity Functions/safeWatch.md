# `safeWatch()` - Watch with Error Handling

## Quick Start (30 seconds)

```javascript
const data = state({ value: 0 });

// Watch with automatic error handling
safeWatch(data, 'value', (newVal) => {
  // This might throw
  riskyOperation(newVal);
}, {
  onError: (error) => {
    console.error('Watcher error:', error);
    // Watcher continues working!
  }
});

data.value = 5;
// If riskyOperation throws, error is caught and watcher keeps running
```

**That's it.** Like `watch()` but with built-in error handling. Errors are caught and watchers continue working instead of breaking.

---

## What is `safeWatch()`?

`safeWatch()` is **watch() with error boundaries**. It wraps watch callbacks in try-catch blocks so errors don't break your watchers.

Think of it as **a safety net for watchers** — if something goes wrong in a callback, the error is caught, reported, and the watcher keeps working.

**In practical terms:** Use `safeWatch()` when watch callbacks might fail (API calls, parsing, external operations) and you don't want one error to stop the watcher.

---

## Syntax

```javascript
// Watch with error handling
safeWatch(stateObject, keyOrFunction, callback, options);

// Parameters
safeWatch(
  state,
  'propertyName',  // or function
  (newVal, oldVal) => { /* may throw */ },
  {
    onError: (error, state) => { /* handle error */ },
    immediate: false  // Run callback immediately
  }
);
```

---

## Why Does This Exist?

### Without safeWatch() - Watcher Breaks

```javascript
// ❌ Regular watch - error stops the watcher
const data = state({ apiUrl: '/api/data' });

watch(data, {
  apiUrl: async (url) => {
    const response = await fetch(url);
    // If fetch fails, watcher is permanently broken!
    const json = await response.json();
  }
});

data.apiUrl = '/invalid-url';
// ❌ Fetch fails, watcher stops working forever
```

### With safeWatch() - Watcher Survives

```javascript
// ✅ safeWatch - error is caught, watcher continues
const data = state({ apiUrl: '/api/data' });

safeWatch(data, 'apiUrl', async (url) => {
  const response = await fetch(url);
  const json = await response.json();
}, {
  onError: (error) => {
    console.error('Failed to fetch:', error);
    // Watcher still works for next change!
  }
});

data.apiUrl = '/invalid-url';
// ✨ Error caught, logged, watcher keeps working

data.apiUrl = '/valid-url';
// ✨ Watcher runs successfully
```

---

## Basic Examples

### Example 1: API Calls

```javascript
const user = state({ userId: 1 });

safeWatch(user, 'userId', async (id) => {
  const response = await fetch(`/api/users/${id}`);
  const userData = await response.json();
  Elements.update({
    userName: { textContent: userData.name }
  });
}, {
  onError: (error) => {
    Elements.update({
      userError: { textContent: `Failed to load user: ${error.message}` }
    });
  }
});
```

### Example 2: JSON Parsing

```javascript
const config = state({ jsonString: '' });

safeWatch(config, 'jsonString', (jsonStr) => {
  const parsed = JSON.parse(jsonStr); // Might throw
  config.parsedData = parsed;
}, {
  onError: (error) => {
    console.error('Invalid JSON:', error);
    config.parsedData = null;
  }
});
```

### Example 3: localStorage

```javascript
const settings = state({ theme: 'light' });

safeWatch(settings, 'theme', (theme) => {
  localStorage.setItem('theme', theme);
  // Might throw in private browsing
}, {
  onError: (error) => {
    console.warn('Could not save to localStorage:', error);
  }
});
```

See full documentation with 10+ examples in the complete version.

---

## Key Takeaways

✅ **Error handling** - Catches errors in watch callbacks
✅ **Watcher survives** - Continues working after errors
✅ **onError callback** - Custom error handling
✅ **Production-ready** - Safe for unreliable operations

---

## Summary

`safeWatch()` is **watch() with error boundaries**. Use it when watch callbacks might fail and you want the watcher to survive errors.
