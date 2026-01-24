# `asyncState()` - Async Operations with Loading/Error States

## Quick Start (30 seconds)

```javascript
// Create async state for API calls
const userData = asyncState(null);

// Auto-update UI based on loading/error/success
effect(() => {
  Elements.update({
    loader: {
      style: { display: userData.loading ? 'block' : 'none' }
    },
    error: {
      textContent: userData.error?.message || '',
      style: { display: userData.error ? 'block' : 'none' }
    },
    content: {
      innerHTML: userData.data ? `<h1>${userData.data.name}</h1>` : '',
      style: { display: userData.isSuccess ? 'block' : 'none' }
    }
  });
});

// Execute async operation
userData.$execute(async () => {
  const response = await fetch('/api/user');
  return response.json();
});
// ✨ Loading indicator shows, then data displays automatically!
```

**That's it.** Manage async operations with automatic loading, error, and success state tracking. No manual flag management needed.

---

## What is `asyncState()`?

`asyncState()` creates **reactive state for async operations**. It automatically tracks loading status, errors, and data, giving you everything needed for async UI patterns.

Think of it as **a smart async wrapper** that handles all the boilerplate of loading states, error handling, and data management for you.

**In practical terms:** Instead of manually managing `isLoading`, `error`, and `data` separately for every API call, you get a pre-configured async state that tracks everything automatically.

---

## Syntax

```javascript
// Create async state
const myAsync = asyncState(initialValue);

// Properties
myAsync.data     // The fetched data
myAsync.loading  // Boolean: operation in progress
myAsync.error    // Error object if failed

// Computed properties
myAsync.isSuccess  // Auto-computed: loaded successfully
myAsync.isError    // Auto-computed: has error

// Methods
myAsync.$execute(asyncFn)  // Execute async operation
myAsync.$reset()           // Reset to initial state

// Example
const posts = asyncState([]);

posts.$execute(async () => {
  const response = await fetch('/api/posts');
  return response.json();
});

console.log(posts.loading);   // true (during fetch)
console.log(posts.isSuccess); // true (after success)
console.log(posts.data);      // Array of posts
```

**Parameters:**
- `initialValue` (optional) - Initial data value (default: `null`)

**Returns:**
- Reactive async state with data, loading, error, and methods

---

## Why Does This Exist?

### The Problem Without asyncState()

Managing async operations manually requires tracking multiple states:

```javascript
// ❌ Vanilla JavaScript - manual async state management
let userData = null;
let isLoading = false;
let error = null;

function updateUI() {
  const loader = document.getElementById('loader');
  const errorDiv = document.getElementById('error');
  const content = document.getElementById('content');

  loader.style.display = isLoading ? 'block' : 'none';

  if (error) {
    errorDiv.textContent = error.message;
    errorDiv.style.display = 'block';
    content.style.display = 'none';
  } else if (userData) {
    errorDiv.style.display = 'none';
    content.innerHTML = `
      <h1>${userData.name}</h1>
      <p>${userData.email}</p>
    `;
    content.style.display = 'block';
  } else {
    errorDiv.style.display = 'none';
    content.style.display = 'none';
  }
}

async function fetchUser() {
  isLoading = true;
  error = null;
  updateUI(); // Manual update

  try {
    const response = await fetch('/api/user');
    if (!response.ok) throw new Error('Failed to fetch');

    userData = await response.json();
    error = null;
  } catch (e) {
    error = e;
    userData = null;
  } finally {
    isLoading = false;
    updateUI(); // Manual update
  }
}

// Initial state
updateUI();
```

**Problems:**
❌ **Manual state tracking** - Separate variables for data, loading, error
❌ **Manual UI updates** - Must call `updateUI()` at every state change
❌ **Repetitive pattern** - Same boilerplate for every async operation
❌ **Easy to miss** - Forget to set loading=false = stuck in loading state
❌ **No computed states** - Must manually check isSuccess, isError

### The Solution with `asyncState()`

```javascript
// ✅ DOM Helpers + Reactive State with asyncState() - automatic
const user = asyncState(null);

// Auto-update UI using bulk updates
effect(() => {
  Elements.update({
    loader: {
      style: { display: user.loading ? 'block' : 'none' }
    },
    errorDiv: {
      textContent: user.error?.message || '',
      style: { display: user.error ? 'block' : 'none' }
    },
    content: {
      innerHTML: user.data ? `
        <h1>${user.data.name}</h1>
        <p>${user.data.email}</p>
      ` : '',
      style: { display: user.isSuccess ? 'block' : 'none' }
    }
  });
});

// Fetch user - state updates automatically
user.$execute(async () => {
  const response = await fetch('/api/user');
  if (!response.ok) throw new Error('Failed to fetch');
  return response.json();
});
// ✨ Loading shows, then data or error displays automatically!

// Refetch button using bulk event binding
Elements.update({
  refetchBtn: {
    addEventListener: ['click', () => {
      user.$execute(async () => {
        const response = await fetch('/api/user');
        return response.json();
      });
    }]
  }
});
```

**Benefits:**
✅ **Automatic state tracking** - data, loading, error managed automatically
✅ **Automatic UI updates** - Effects handle DOM sync
✅ **Computed states** - isSuccess and isError auto-calculated
✅ **Clean API** - $execute method handles everything
✅ **Error handling** - Built-in error capture

---

## Mental Model: Smart Async Manager

Think of `asyncState()` like **a smart assistant managing async tasks**:

**Without asyncState() (Manual Management):**
```
┌─────────────────────────────┐
│  You Manage Everything      │
│                             │
│  Set loading = true         │
│  Try/catch the operation    │
│  Set data on success        │
│  Set error on failure       │
│  Set loading = false        │
│  Update UI manually         │
│                             │
│  Lots of boilerplate!       │
└─────────────────────────────┘
```

**With asyncState() (Smart Manager):**
```
┌─────────────────────────────┐
│  Assistant Manages Async    │
│                             │
│  ✓ Sets loading = true      │
│  ✓ Runs your async function │
│  ✓ Catches errors           │
│  ✓ Sets data or error       │
│  ✓ Sets loading = false     │
│  ✓ UI auto-updates          │
│                             │
│  Just call $execute()!      │
└─────────────────────────────┘
```

The async state **handles all the async machinery** so you can focus on your logic.

---

## Built-in Properties

### State Properties
- `data` - The fetched/computed data (initially `initialValue`)
- `loading` - Boolean indicating operation in progress
- `error` - Error object if operation failed, `null` otherwise

### Computed Properties (Auto-calculated)
- `isSuccess` - `true` if loaded successfully (not loading, no error, has data)
- `isError` - `true` if operation failed (not loading, has error)

---

## Built-in Methods

### `$execute(asyncFn)`
Execute an async operation and manage state automatically.

```javascript
const users = asyncState([]);

users.$execute(async () => {
  const response = await fetch('/api/users');
  return response.json();
});
// loading = true (before fetch)
// loading = false, data = result (after success)
// loading = false, error = e (after failure)
```

**Returns:** Promise that resolves/rejects with the result

### `$reset()`
Reset state to initial values.

```javascript
users.$reset();
// data = initialValue
// loading = false
// error = null
```

---

## Basic Usage

### Example 1: Fetch User Data

```javascript
const user = asyncState(null);

// UI updates using bulk updates
effect(() => {
  Elements.update({
    loader: {
      style: { display: user.loading ? 'block' : 'none' }
    },
    userName: {
      textContent: user.data?.name || '',
      style: { display: user.isSuccess ? 'block' : 'none' }
    },
    userEmail: {
      textContent: user.data?.email || '',
      style: { display: user.isSuccess ? 'block' : 'none' }
    },
    errorMessage: {
      textContent: user.error?.message || '',
      style: { display: user.isError ? 'block' : 'none' }
    }
  });
});

// Fetch data using bulk event binding
Elements.update({
  loadBtn: {
    addEventListener: ['click', () => {
      user.$execute(async () => {
        const response = await fetch('/api/user/123');
        if (!response.ok) throw new Error('Failed to fetch user');
        return response.json();
      });
    }],
    disabled: user.loading
  }
});
```

---

### Example 2: Search with Async State

```javascript
const searchResults = asyncState([]);
const searchQuery = ref('');

// Debounced search
let searchTimeout;

effect(() => {
  clearTimeout(searchTimeout);

  if (searchQuery.value.length >= 3) {
    searchTimeout = setTimeout(() => {
      searchResults.$execute(async () => {
        const response = await fetch(`/api/search?q=${searchQuery.value}`);
        return response.json();
      });
    }, 500);
  } else {
    searchResults.$reset();
  }
});

// UI updates using bulk updates
effect(() => {
  Elements.update({
    searchLoader: {
      style: { display: searchResults.loading ? 'inline-block' : 'none' }
    },
    resultCount: {
      textContent: searchResults.isSuccess
        ? `${searchResults.data.length} results`
        : '',
      style: { display: searchResults.isSuccess ? 'block' : 'none' }
    },
    resultList: {
      innerHTML: searchResults.data
        .map(item => `<li>${item.title}</li>`)
        .join(''),
      style: { display: searchResults.isSuccess ? 'block' : 'none' }
    },
    noResults: {
      textContent: searchResults.isSuccess && searchResults.data.length === 0
        ? 'No results found'
        : '',
      style: {
        display: searchResults.isSuccess && searchResults.data.length === 0
          ? 'block'
          : 'none'
      }
    },
    searchError: {
      textContent: searchResults.error?.message || '',
      style: { display: searchResults.isError ? 'block' : 'none' }
    }
  });
});
```

---

### Example 3: Submit Form with Async State

```javascript
const submitState = asyncState(null);

// UI updates using bulk updates
effect(() => {
  Elements.update({
    submitBtn: {
      disabled: submitState.loading,
      textContent: submitState.loading ? 'Submitting...' : 'Submit'
    },
    successMessage: {
      textContent: submitState.isSuccess ? '✓ Form submitted successfully!' : '',
      style: {
        display: submitState.isSuccess ? 'block' : 'none',
        color: 'green'
      }
    },
    submitError: {
      textContent: submitState.error?.message || '',
      style: {
        display: submitState.isError ? 'block' : 'none',
        color: 'red'
      }
    }
  });
});

// Submit handler using bulk event binding
Elements.update({
  submitBtn: {
    addEventListener: ['click', async () => {
      await submitState.$execute(async () => {
        const response = await fetch('/api/submit', {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(formData)
        });

        if (!response.ok) {
          throw new Error('Submission failed');
        }

        return response.json();
      });

      // Reset form on success
      if (submitState.isSuccess) {
        setTimeout(() => {
          submitState.$reset();
          formData.$reset();
        }, 2000);
      }
    }]
  }
});
```

---

### Example 4: Pagination with Async State

```javascript
const posts = asyncState([]);
const currentPage = ref(1);
const pageSize = 10;

// Fetch page
async function fetchPage(page) {
  await posts.$execute(async () => {
    const response = await fetch(`/api/posts?page=${page}&size=${pageSize}`);
    return response.json();
  });
}

// Fetch when page changes
effect(() => {
  fetchPage(currentPage.value);
});

// UI updates using bulk updates
effect(() => {
  Elements.update({
    postList: {
      innerHTML: posts.loading
        ? '<li>Loading...</li>'
        : posts.isSuccess
          ? posts.data.map(p => `<li>${p.title}</li>`).join('')
          : posts.isError
            ? `<li>Error: ${posts.error.message}</li>`
            : '<li>No posts</li>'
    },
    prevBtn: {
      disabled: currentPage.value === 1 || posts.loading
    },
    nextBtn: {
      disabled: posts.loading || (posts.isSuccess && posts.data.length < pageSize)
    },
    pageIndicator: {
      textContent: `Page ${currentPage.value}`
    }
  });
});

// Navigation using bulk event binding
Elements.update({
  prevBtn: {
    addEventListener: ['click', () => {
      if (currentPage.value > 1) currentPage.value--;
    }]
  },
  nextBtn: {
    addEventListener: ['click', () => {
      currentPage.value++;
    }]
  }
});
```

---

### Example 5: Multiple Async States

```javascript
const userProfile = asyncState(null);
const userPosts = asyncState([]);
const userFollowers = asyncState([]);

// Fetch all user data
async function loadUserData(userId) {
  await Promise.all([
    userProfile.$execute(async () => {
      const response = await fetch(`/api/users/${userId}`);
      return response.json();
    }),
    userPosts.$execute(async () => {
      const response = await fetch(`/api/users/${userId}/posts`);
      return response.json();
    }),
    userFollowers.$execute(async () => {
      const response = await fetch(`/api/users/${userId}/followers`);
      return response.json();
    })
  ]);
}

// UI shows individual loading states
effect(() => {
  Elements.update({
    profileSection: {
      innerHTML: userProfile.loading
        ? 'Loading profile...'
        : userProfile.isSuccess
          ? `<h1>${userProfile.data.name}</h1>`
          : 'Failed to load profile'
    },
    postsSection: {
      innerHTML: userPosts.loading
        ? 'Loading posts...'
        : userPosts.isSuccess
          ? `${userPosts.data.length} posts`
          : 'Failed to load posts'
    },
    followersSection: {
      innerHTML: userFollowers.loading
        ? 'Loading followers...'
        : userFollowers.isSuccess
          ? `${userFollowers.data.length} followers`
          : 'Failed to load followers'
    }
  });
});
```

---

### Example 6: Dependent Async Operations

```javascript
const user = asyncState(null);
const userSettings = asyncState(null);

// Fetch user, then settings
async function loadUserAndSettings() {
  // First: fetch user
  await user.$execute(async () => {
    const response = await fetch('/api/user');
    return response.json();
  });

  // Then: fetch settings (only if user loaded)
  if (user.isSuccess) {
    await userSettings.$execute(async () => {
      const response = await fetch(`/api/users/${user.data.id}/settings`);
      return response.json();
    });
  }
}

// UI updates using bulk updates
effect(() => {
  Elements.update({
    userInfo: {
      textContent: user.isSuccess ? user.data.name : '',
      style: { display: user.isSuccess ? 'block' : 'none' }
    },
    settingsPanel: {
      style: {
        display: userSettings.isSuccess ? 'block' : 'none',
        opacity: userSettings.loading ? '0.5' : '1'
      }
    }
  });
});
```

---

### Example 7: Retry Logic

```javascript
const data = asyncState(null);
const retryCount = ref(0);
const maxRetries = 3;

async function fetchWithRetry() {
  retryCount.value = 0;

  while (retryCount.value <= maxRetries) {
    await data.$execute(async () => {
      const response = await fetch('/api/data');
      if (!response.ok) throw new Error('Fetch failed');
      return response.json();
    });

    if (data.isSuccess) break;

    retryCount.value++;
    if (retryCount.value <= maxRetries) {
      await new Promise(resolve => setTimeout(resolve, 1000 * retryCount.value));
    }
  }
}

// UI updates using bulk updates
effect(() => {
  Elements.update({
    retryIndicator: {
      textContent: data.loading && retryCount.value > 0
        ? `Retry attempt ${retryCount.value}/${maxRetries}...`
        : '',
      style: { display: data.loading && retryCount.value > 0 ? 'block' : 'none' }
    },
    errorMessage: {
      textContent: data.isError
        ? `Failed after ${retryCount.value} attempts: ${data.error.message}`
        : '',
      style: { display: data.isError ? 'block' : 'none' }
    }
  });
});
```

---

### Example 8: Optimistic Updates

```javascript
const todos = asyncState([
  { id: 1, text: 'Task 1', done: false }
]);

async function toggleTodo(id) {
  // Optimistic update
  const originalData = [...todos.data];
  todos.data = todos.data.map(t =>
    t.id === id ? { ...t, done: !t.done } : t
  );

  try {
    await todos.$execute(async () => {
      const response = await fetch(`/api/todos/${id}/toggle`, {
        method: 'POST'
      });
      return response.json();
    });
  } catch (error) {
    // Rollback on error
    todos.data = originalData;
  }
}
```

---

### Example 9: Polling with Async State

```javascript
const status = asyncState(null);
let pollingInterval;

function startPolling() {
  // Initial fetch
  status.$execute(async () => {
    const response = await fetch('/api/status');
    return response.json();
  });

  // Poll every 5 seconds
  pollingInterval = setInterval(() => {
    status.$execute(async () => {
      const response = await fetch('/api/status');
      return response.json();
    });
  }, 5000);
}

function stopPolling() {
  clearInterval(pollingInterval);
}

// UI updates using bulk updates
effect(() => {
  Elements.update({
    statusDisplay: {
      textContent: status.isSuccess ? status.data.message : 'Connecting...',
      style: {
        color: status.isSuccess
          ? status.data.healthy ? 'green' : 'red'
          : 'gray'
      }
    },
    lastUpdate: {
      textContent: status.isSuccess
        ? `Last updated: ${new Date().toLocaleTimeString()}`
        : ''
    }
  });
});

// Start/stop using bulk event binding
Elements.update({
  startBtn: {
    addEventListener: ['click', startPolling]
  },
  stopBtn: {
    addEventListener: ['click', stopPolling]
  }
});
```

---

### Example 10: File Upload with Progress

```javascript
const uploadState = asyncState(null);
const uploadProgress = ref(0);

async function uploadFile(file) {
  uploadProgress.value = 0;

  await uploadState.$execute(async () => {
    const formData = new FormData();
    formData.append('file', file);

    const response = await fetch('/api/upload', {
      method: 'POST',
      body: formData
    });

    // Simulate progress (in real app, use XMLHttpRequest)
    for (let i = 0; i <= 100; i += 10) {
      uploadProgress.value = i;
      await new Promise(resolve => setTimeout(resolve, 100));
    }

    return response.json();
  });
}

// UI updates using bulk updates
effect(() => {
  Elements.update({
    progressBar: {
      style: {
        width: `${uploadProgress.value}%`,
        display: uploadState.loading ? 'block' : 'none'
      }
    },
    progressText: {
      textContent: `${uploadProgress.value}%`,
      style: { display: uploadState.loading ? 'block' : 'none' }
    },
    uploadBtn: {
      disabled: uploadState.loading
    },
    successMessage: {
      textContent: uploadState.isSuccess ? '✓ Upload complete!' : '',
      style: { display: uploadState.isSuccess ? 'block' : 'none' }
    }
  });
});

// File input using bulk event binding
Elements.update({
  fileInput: {
    addEventListener: ['change', (e) => {
      const file = e.target.files[0];
      if (file) uploadFile(file);
    }]
  }
});
```

---

## Common Patterns

### Pattern 1: Loading Skeleton

```javascript
effect(() => {
  if (data.loading) {
    Elements.update({
      content: {
        innerHTML: `
          <div class="skeleton"></div>
          <div class="skeleton"></div>
          <div class="skeleton"></div>
        `
      }
    });
  } else if (data.isSuccess) {
    Elements.update({
      content: {
        innerHTML: renderContent(data.data)
      }
    });
  }
});
```

### Pattern 2: Error Retry

```javascript
effect(() => {
  Elements.update({
    errorPanel: {
      innerHTML: data.isError ? `
        <p>Error: ${data.error.message}</p>
        <button id="retry-btn">Retry</button>
      ` : '',
      style: { display: data.isError ? 'block' : 'none' }
    }
  });
});

Elements.update({
  errorPanel: {
    addEventListener: ['click', (e) => {
      if (e.target.id === 'retry-btn') {
        data.$execute(fetchDataFn);
      }
    }]
  }
});
```

### Pattern 3: Empty State

```javascript
effect(() => {
  const isEmpty = data.isSuccess && data.data.length === 0;

  Elements.update({
    emptyState: {
      innerHTML: isEmpty ? `
        <p>No items found</p>
        <button id="create-btn">Create First Item</button>
      ` : '',
      style: { display: isEmpty ? 'block' : 'none' }
    }
  });
});
```

---

## Key Takeaways

✅ **Automatic state tracking** - loading, error, data managed automatically
✅ **Computed states** - isSuccess and isError auto-calculated
✅ **Clean API** - $execute handles entire async flow
✅ **Error handling** - Built-in try/catch with error state
✅ **Reactive** - Works seamlessly with effects and bindings

---

## What's Next?

- **`form()`** - For form state management
- **`store()`** - For global state with actions
- **Enhanced async features** - AbortSignal, timeout, refetch (see reactive enhancements)

---

## Summary

`asyncState()` creates **reactive state for async operations** with automatic loading, error, and success tracking.

**The magic formula:**
```
asyncState(initialValue) =
  state({ data, loading, error }) +
  computed({ isSuccess, isError }) +
  $execute(asyncFn) method
───────────────────────────────────
Complete async state management
```

Think of it as **a smart async wrapper** — it handles loading states, catches errors, and updates data automatically. Just call `$execute()` with your async function and let it handle the rest. Perfect for API calls, data fetching, and any async operation that needs UI feedback.
