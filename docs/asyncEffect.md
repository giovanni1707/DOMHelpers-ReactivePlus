# `asyncEffect()` - Async Effects with AbortSignal

## Quick Start (30 seconds)

```javascript
const search = state({ query: '' });

// Async effect with automatic cancellation
asyncEffect(async (signal) => {
  if (!search.query) return;

  // AbortSignal automatically cancels old requests
  const response = await fetch(`/api/search?q=${search.query}`, { signal });
  const results = await response.json();

  Elements.update({
    results: {
      innerHTML: results.map(r => `<li>${r.title}</li>`).join('')
    }
  });
});

search.query = 'laptop';
// Starts fetch

search.query = 'mouse';
// Previous fetch automatically cancelled, new one starts!
```

**That's it.** Run async operations in effects with automatic cancellation. When dependencies change, old async operations are aborted automatically.

---

## What is `asyncEffect()`?

`asyncEffect()` is **effect() for async operations with automatic cancellation**. It provides an AbortSignal that automatically cancels when the effect re-runs.

Think of it as **async/await with automatic cleanup** — you write normal async code, and old operations are cancelled when dependencies change.

**In practical terms:** Use `asyncEffect()` for fetching data, debounced operations, or any async work that should be cancelled if dependencies change.

---

## Syntax

```javascript
// Async effect with AbortSignal
asyncEffect(async (signal) => {
  // signal is automatically aborted on re-run
  await fetch(url, { signal });
}, options);

// Options
asyncEffect(async (signal) => {
  // ...
}, {
  onError: (error) => { /* handle errors */ },
  debounce: 500  // Debounce effect execution
});
```

---

## Why Does This Exist?

### Without asyncEffect() - Race Conditions

```javascript
// ❌ Vanilla JavaScript - race conditions
let searchQuery = '';

async function performSearch() {
  const query = searchQuery;

  const response = await fetch(`/api/search?q=${query}`);
  const results = await response.json();

  // BUG: If user typed again, this might be stale data!
  document.getElementById('results').innerHTML = results
    .map(r => `<li>${r.title}</li>`)
    .join('');
}

// User types fast
document.getElementById('search-input').addEventListener('input', (e) => {
  searchQuery = e.target.value;
  performSearch(); // Multiple requests flying!
});
```

### With asyncEffect() - Auto-cancellation

```javascript
// ✅ DOM Helpers + asyncEffect() - automatic cancellation
const search = state({ query: '' });

asyncEffect(async (signal) => {
  if (!search.query) return;

  // AbortSignal cancels old requests automatically
  const response = await fetch(`/api/search?q=${search.query}`, { signal });
  const results = await response.json();

  Elements.update({
    results: {
      innerHTML: results.map(r => `<li>${r.title}</li>`).join('')
    }
  });
});

// Input using bulk event binding
Elements.update({
  searchInput: {
    addEventListener: ['input', (e) => {
      search.query = e.target.value;
      // ✨ Old fetch automatically cancelled!
    }]
  }
});
```

---

## Basic Examples

### Example 1: Search with Auto-cancel

```javascript
const search = state({ query: '' });

asyncEffect(async (signal) => {
  if (search.query.length < 3) return;

  const response = await fetch(`/api/search?q=${search.query}`, { signal });
  const results = await response.json();

  Elements.update({
    searchResults: {
      innerHTML: results.map(r => `<li>${r.name}</li>`).join('')
    }
  });
}, {
  debounce: 300  // Wait 300ms before searching
});
```

### Example 2: User Data Loading

```javascript
const user = state({ userId: 1 });

asyncEffect(async (signal) => {
  Elements.update({
    loader: { style: { display: 'block' } }
  });

  const response = await fetch(`/api/users/${user.userId}`, { signal });
  const userData = await response.json();

  Elements.update({
    loader: { style: { display: 'none' } },
    userName: { textContent: userData.name },
    userEmail: { textContent: userData.email }
  });
});
```

### Example 3: Polling with Cancellation

```javascript
const status = state({ shouldPoll: true });

asyncEffect(async (signal) => {
  while (status.shouldPoll && !signal.aborted) {
    const response = await fetch('/api/status', { signal });
    const data = await response.json();

    Elements.update({
      statusDisplay: { textContent: data.message }
    });

    // Wait 5 seconds (respecting abort signal)
    await new Promise(resolve => {
      const timeout = setTimeout(resolve, 5000);
      signal.addEventListener('abort', () => clearTimeout(timeout));
    });
  }
});

// Stop polling
status.shouldPoll = false;
// ✨ AbortSignal fired, polling stops
```

---

## Key Takeaways

✅ **Automatic cancellation** - AbortSignal provided automatically
✅ **No race conditions** - Old requests cancelled on re-run
✅ **Debouncing support** - Built-in debounce option
✅ **Clean code** - Write normal async/await

---

## Summary

`asyncEffect()` runs **async operations with automatic cancellation**. Perfect for API calls, searches, and any async work that should be cancelled when dependencies change.
