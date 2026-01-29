# Async State Properties - Complete Reference

**Quick Start (30 seconds)**
```javascript
const userState = ReactiveUtils.asyncState();

// Execute async operation
userState.$execute(async (signal) => {
  const response = await fetch('/api/user', { signal });
  return response.json();
});

// Access state properties
console.log(userState.loading); // true (during request)
console.log(userState.data); // null (initially)
console.log(userState.error); // null (no error yet)
console.log(userState.requestId); // 1 (first request)
console.log(userState.abortController); // AbortController instance

// Access computed properties
console.log(userState.isSuccess); // false (still loading)
console.log(userState.isError); // false (no error)
console.log(userState.isIdle); // false (loading in progress)

// After success
console.log(userState.loading); // false
console.log(userState.data); // { id: 1, name: 'John' }
console.log(userState.isSuccess); // true
```

---

## **What are Async State Properties?**

Async state properties are **reactive properties** that automatically manage the lifecycle of asynchronous operations, handling loading states, data storage, error handling, and request cancellation.

**The 8 core properties:**

**State Properties (5):**
1. **`data`** - Stores the result of successful async operations
2. **`loading`** - Boolean indicating if an async operation is in progress
3. **`error`** - Stores error object from failed operations
4. **`requestId`** - Counter tracking request sequence (prevents race conditions)
5. **`abortController`** - Current AbortController for request cancellation

**Computed Properties (3):**
6. **`isSuccess`** - True when data loaded successfully without errors
7. **`isError`** - True when operation failed with an error
8. **`isIdle`** - True when no operation has run yet

**Key characteristics:**
- **Reactive**: All properties trigger effects on change
- **Automatic**: State updates managed automatically
- **Race-Safe**: RequestId prevents race conditions
- **Cancellable**: AbortController enables request cancellation
- **Type Safe**: Predictable property types and values

---

## **State Properties**

### **1. `data`**

**Type**: `any`
**Default**: `null` (or initialValue)
**Reactive**: Yes

Stores the result of successful async operations. Updated only when request completes successfully and is still the latest request.

```javascript
const state = ReactiveUtils.asyncState();

console.log(state.data); // null (initial)

await state.$execute(async () => {
  return { name: 'John', age: 30 };
});

console.log(state.data); // { name: 'John', age: 30 }
```

**When updated:**
- After successful async operation completion
- When `$reset()` is called (back to initialValue)
- Only if requestId matches (latest request wins)

**Use cases:**
- Display fetched data
- Conditional rendering
- Data transformations
- Caching results
- Success indicators

---

### **2. `loading`**

**Type**: `boolean`
**Default**: `false`
**Reactive**: Yes

Indicates whether an async operation is currently in progress.

```javascript
const state = ReactiveUtils.asyncState();

console.log(state.loading); // false

const promise = state.$execute(async () => {
  await new Promise(resolve => setTimeout(resolve, 1000));
  return 'Done';
});

console.log(state.loading); // true

await promise;
console.log(state.loading); // false
```

**When `true`:**
- After `$execute()` is called
- During async operation
- Before result or error is set

**When `false`:**
- Initially
- After operation completes (success or error)
- After `$abort()` or `$reset()`

**Use cases:**
- Show loading spinner
- Disable buttons during request
- Loading skeletons
- Progress indicators
- Prevent double submissions

---

### **3. `error`**

**Type**: `Error | null`
**Default**: `null`
**Reactive**: Yes

Stores error object from failed async operations.

```javascript
const state = ReactiveUtils.asyncState();

console.log(state.error); // null

await state.$execute(async () => {
  throw new Error('API failed');
});

console.log(state.error); // Error: API failed
console.log(state.error.message); // 'API failed'
```

**When set:**
- After async operation throws an error
- Only if requestId matches (latest request)
- Cleared before each new request

**When `null`:**
- Initially
- Cleared when new request starts
- After `$reset()`

**Use cases:**
- Display error messages
- Error recovery
- Retry logic
- Error logging
- Conditional rendering

---

### **4. `requestId`**

**Type**: `number`
**Default**: `0`
**Reactive**: Yes

Counter that increments with each request, used to identify and track request sequence to prevent race conditions.

```javascript
const state = ReactiveUtils.asyncState();

console.log(state.requestId); // 0

await state.$execute(async () => 'First');
console.log(state.requestId); // 1

await state.$execute(async () => 'Second');
console.log(state.requestId); // 2

state.$reset();
console.log(state.requestId); // 0 (reset)
```

**How it works:**
- Increments before each request
- Compared in finally block
- Only latest request updates state
- Prevents stale data updates

**Use cases:**
- Race condition prevention
- Request deduplication
- Debug request tracking
- Request cancellation logic
- Stale data detection

---

### **5. `abortController`**

**Type**: `AbortController | null`
**Default**: `null`
**Reactive**: Yes

Current AbortController instance for the in-progress request, enabling request cancellation.

```javascript
const state = ReactiveUtils.asyncState();

console.log(state.abortController); // null

state.$execute(async (signal) => {
  const response = await fetch('/api/data', { signal });
  return response.json();
});

console.log(state.abortController); // AbortController instance
console.log(state.abortController.signal); // AbortSignal

// Abort the request
state.$abort();
console.log(state.abortController); // null
```

**When set:**
- Created before each async operation
- Stored during operation
- Accessible via signal parameter in async function

**When `null`:**
- Initially
- After operation completes
- After `$abort()` is called
- After automatic cancellation (new request)

**Use cases:**
- Manual request cancellation
- Pass signal to fetch()
- Cancel on component unmount
- Timeout implementation
- User-initiated cancellation

---

## **Computed Properties**

### **6. `isSuccess`**

**Type**: Computed boolean
**Returns**: `boolean`
**Reactive**: Yes

Returns `true` when data has been loaded successfully without errors and not currently loading.

```javascript
const state = ReactiveUtils.asyncState();

console.log(state.isSuccess); // false (no data yet)

await state.$execute(async () => ({ id: 1 }));

console.log(state.isSuccess); // true
console.log(state.data); // { id: 1 }
console.log(state.loading); // false
console.log(state.error); // null
```

**Implementation:**
```javascript
state.$computed('isSuccess', function() {
  return !this.loading && !this.error && this.data !== null;
});
```

**When `true`:**
- Data is not null
- Not currently loading
- No error exists

**Use cases:**
- Show success message
- Conditional rendering of data
- Enable dependent actions
- Success animations
- Cache validation

---

### **7. `isError`**

**Type**: Computed boolean
**Returns**: `boolean`
**Reactive**: Yes

Returns `true` when operation has failed with an error and not currently loading.

```javascript
const state = ReactiveUtils.asyncState();

console.log(state.isError); // false

await state.$execute(async () => {
  throw new Error('Failed');
});

console.log(state.isError); // true
console.log(state.error); // Error: Failed
console.log(state.loading); // false
```

**Implementation:**
```javascript
state.$computed('isError', function() {
  return !this.loading && this.error !== null;
});
```

**When `true`:**
- Error is not null
- Not currently loading

**Use cases:**
- Display error UI
- Show retry button
- Error tracking
- Fallback rendering
- Error recovery

---

### **8. `isIdle`**

**Type**: Computed boolean
**Returns**: `boolean`
**Reactive**: Yes

Returns `true` when no async operation has been executed yet (pristine state).

```javascript
const state = ReactiveUtils.asyncState();

console.log(state.isIdle); // true (initial state)

await state.$execute(async () => 'data');

console.log(state.isIdle); // false (has data now)

state.$reset();

console.log(state.isIdle); // true (back to pristine)
```

**Implementation:**
```javascript
state.$computed('isIdle', function() {
  return !this.loading && this.data === null && this.error === null;
});
```

**When `true`:**
- Data is null
- Error is null
- Not currently loading

**Use cases:**
- Show placeholder UI
- Initial state detection
- Lazy loading triggers
- Onboarding flows
- Empty state rendering

---

## **Examples**

### **Example 1: Basic Data Fetching**
```javascript
const userState = ReactiveUtils.asyncState();

ReactiveUtils.effect(() => {
  if (userState.loading) {
    console.log('Loading user...');
  }

  if (userState.isSuccess) {
    console.log('User loaded:', userState.data);
  }

  if (userState.isError) {
    console.log('Error:', userState.error.message);
  }
});

await userState.$execute(async (signal) => {
  const response = await fetch('/api/user', { signal });
  return response.json();
});
```

### **Example 2: Loading Indicator**
```javascript
const postsState = ReactiveUtils.asyncState();

ReactiveUtils.effect(() => {
  const spinner = document.getElementById('spinner');
  const content = document.getElementById('content');

  if (postsState.loading) {
    spinner.style.display = 'block';
    content.style.opacity = '0.5';
  } else {
    spinner.style.display = 'none';
    content.style.opacity = '1';
  }
});

postsState.$execute(async () => {
  const response = await fetch('/api/posts');
  return response.json();
});
```

### **Example 3: Error Handling**
```javascript
const apiState = ReactiveUtils.asyncState();

ReactiveUtils.effect(() => {
  const errorDiv = document.getElementById('error');

  if (apiState.isError) {
    errorDiv.textContent = `Error: ${apiState.error.message}`;
    errorDiv.style.display = 'block';
  } else {
    errorDiv.style.display = 'none';
  }
});

await apiState.$execute(async () => {
  const response = await fetch('/api/data');
  if (!response.ok) {
    throw new Error(`HTTP ${response.status}`);
  }
  return response.json();
});
```

### **Example 4: Race Condition Prevention**
```javascript
const searchState = ReactiveUtils.asyncState();

async function search(query) {
  const currentRequestId = searchState.requestId + 1;

  await searchState.$execute(async (signal) => {
    console.log(`Starting request ${currentRequestId} for: ${query}`);

    await new Promise(resolve => setTimeout(resolve, Math.random() * 1000));

    if (signal.aborted) {
      console.log(`Request ${currentRequestId} aborted`);
      return;
    }

    console.log(`Completing request ${currentRequestId}`);
    return { query, results: [`Result for ${query}`] };
  });
}

// Rapid searches - only last one updates state
search('a');
search('ab');
search('abc'); // Only this completes and updates data

ReactiveUtils.effect(() => {
  console.log('Current requestId:', searchState.requestId);
  console.log('Current data:', searchState.data);
});
```

### **Example 5: Manual Abort**
```javascript
const downloadState = ReactiveUtils.asyncState();

const downloadBtn = document.getElementById('download');
const cancelBtn = document.getElementById('cancel');

downloadBtn.addEventListener('click', () => {
  downloadState.$execute(async (signal) => {
    const response = await fetch('/api/large-file', { signal });
    return response.blob();
  });
});

cancelBtn.addEventListener('click', () => {
  if (downloadState.abortController) {
    downloadState.$abort();
    console.log('Download cancelled');
  }
});

ReactiveUtils.effect(() => {
  cancelBtn.disabled = !downloadState.loading;
  downloadBtn.disabled = downloadState.loading;
});
```

### **Example 6: Success/Error/Idle States**
```javascript
const dataState = ReactiveUtils.asyncState();

ReactiveUtils.effect(() => {
  const container = document.getElementById('container');

  if (dataState.isIdle) {
    container.innerHTML = '<button onclick="loadData()">Load Data</button>';
  } else if (dataState.loading) {
    container.innerHTML = '<div class="spinner">Loading...</div>';
  } else if (dataState.isSuccess) {
    container.innerHTML = `<div class="data">${JSON.stringify(dataState.data)}</div>`;
  } else if (dataState.isError) {
    container.innerHTML = `
      <div class="error">
        ${dataState.error.message}
        <button onclick="retry()">Retry</button>
      </div>
    `;
  }
});

function loadData() {
  dataState.$execute(async () => {
    const response = await fetch('/api/data');
    return response.json();
  });
}

function retry() {
  dataState.$refetch();
}
```

### **Example 7: Request Tracking**
```javascript
const state = ReactiveUtils.asyncState();

ReactiveUtils.effect(() => {
  console.log('\n📊 Request Status:');
  console.log('─'.repeat(40));
  console.log(`Request ID: ${state.requestId}`);
  console.log(`Loading: ${state.loading}`);
  console.log(`Has Data: ${state.data !== null}`);
  console.log(`Has Error: ${state.error !== null}`);
  console.log(`Has AbortController: ${state.abortController !== null}`);
  console.log('─'.repeat(40));
  console.log(`Status: ${
    state.isIdle ? 'Idle' :
    state.loading ? 'Loading' :
    state.isSuccess ? 'Success' :
    state.isError ? 'Error' :
    'Unknown'
  }`);
});

// Make requests
await state.$execute(async () => 'First');
await state.$execute(async () => 'Second');
await state.$execute(async () => 'Third');
```

### **Example 8: Timeout Implementation**
```javascript
const apiState = ReactiveUtils.asyncState();

async function fetchWithTimeout(url, timeoutMs = 5000) {
  await apiState.$execute(async (signal) => {
    const timeoutId = setTimeout(() => {
      if (apiState.abortController) {
        apiState.abortController.abort();
      }
    }, timeoutMs);

    try {
      const response = await fetch(url, { signal });
      clearTimeout(timeoutId);

      if (!response.ok) {
        throw new Error(`HTTP ${response.status}`);
      }

      return response.json();
    } catch (error) {
      clearTimeout(timeoutId);

      if (error.name === 'AbortError') {
        throw new Error('Request timeout');
      }

      throw error;
    }
  });
}

await fetchWithTimeout('/api/slow-endpoint', 3000);

if (apiState.isError) {
  console.log('Error:', apiState.error.message); // 'Request timeout'
}
```

### **Example 9: Data Transformation**
```javascript
const userState = ReactiveUtils.asyncState();

// Reactive computed property based on data
ReactiveUtils.effect(() => {
  if (userState.isSuccess && userState.data) {
    const user = userState.data;
    const displayName = `${user.firstName} ${user.lastName}`;
    const isAdmin = user.role === 'admin';

    document.getElementById('user-name').textContent = displayName;
    document.getElementById('admin-badge').style.display =
      isAdmin ? 'block' : 'none';
  }
});

await userState.$execute(async () => {
  const response = await fetch('/api/user/123');
  return response.json();
  // Returns: { firstName: 'John', lastName: 'Doe', role: 'admin' }
});
```

### **Example 10: Polling with Abort**
```javascript
const statusState = ReactiveUtils.asyncState();
let pollingInterval;

function startPolling() {
  pollingInterval = setInterval(() => {
    statusState.$execute(async (signal) => {
      const response = await fetch('/api/status', { signal });
      return response.json();
    });
  }, 2000);
}

function stopPolling() {
  clearInterval(pollingInterval);
  statusState.$abort();
}

ReactiveUtils.effect(() => {
  if (statusState.isSuccess) {
    console.log('Status:', statusState.data.status);

    // Stop polling if job complete
    if (statusState.data.status === 'complete') {
      stopPolling();
    }
  }
});

startPolling();
```

### **Example 11: Retry Logic**
```javascript
const apiState = ReactiveUtils.asyncState();
let retryCount = 0;
const maxRetries = 3;

async function fetchWithRetry(url) {
  retryCount = 0;

  while (retryCount < maxRetries) {
    await apiState.$execute(async (signal) => {
      try {
        const response = await fetch(url, { signal });

        if (!response.ok) {
          throw new Error(`HTTP ${response.status}`);
        }

        retryCount = 0; // Reset on success
        return response.json();
      } catch (error) {
        retryCount++;

        if (retryCount >= maxRetries) {
          throw new Error(`Failed after ${maxRetries} attempts: ${error.message}`);
        }

        // Wait before retry
        await new Promise(resolve =>
          setTimeout(resolve, 1000 * retryCount)
        );

        throw error; // Re-throw to trigger retry
      }
    });

    if (apiState.isSuccess) {
      break;
    }
  }
}

await fetchWithRetry('/api/unstable-endpoint');
```

### **Example 12: Dependent Requests**
```javascript
const userState = ReactiveUtils.asyncState();
const postsState = ReactiveUtils.asyncState();

// Load user first
await userState.$execute(async () => {
  const response = await fetch('/api/user');
  return response.json();
});

// Load user's posts after user loads
ReactiveUtils.effect(() => {
  if (userState.isSuccess && userState.data) {
    const userId = userState.data.id;

    postsState.$execute(async (signal) => {
      const response = await fetch(`/api/users/${userId}/posts`, { signal });
      return response.json();
    });
  }
});

ReactiveUtils.effect(() => {
  if (userState.loading || postsState.loading) {
    console.log('Loading...');
  }

  if (userState.isSuccess && postsState.isSuccess) {
    console.log('User:', userState.data);
    console.log('Posts:', postsState.data);
  }
});
```

### **Example 13: State Machine Visualization**
```javascript
const state = ReactiveUtils.asyncState();

ReactiveUtils.effect(() => {
  const phases = [
    { name: 'Idle', active: state.isIdle },
    { name: 'Loading', active: state.loading },
    { name: 'Success', active: state.isSuccess },
    { name: 'Error', active: state.isError }
  ];

  console.log('\n🔄 State Machine:');
  console.log('─'.repeat(40));
  phases.forEach(phase => {
    const indicator = phase.active ? '●' : '○';
    const style = phase.active ? 'ACTIVE' : '';
    console.log(`${indicator} ${phase.name} ${style}`);
  });
  console.log('─'.repeat(40));

  console.log('Properties:');
  console.log(`  data: ${state.data !== null ? '✓' : '✗'}`);
  console.log(`  loading: ${state.loading ? '✓' : '✗'}`);
  console.log(`  error: ${state.error !== null ? '✓' : '✗'}`);
  console.log(`  requestId: ${state.requestId}`);
});

// Trigger different states
state.$execute(async () => 'Success');
await new Promise(resolve => setTimeout(resolve, 100));

state.$execute(async () => { throw new Error('Failed'); });
await new Promise(resolve => setTimeout(resolve, 100));

state.$reset();
```

### **Example 14: Request Deduplication**
```javascript
const dataState = ReactiveUtils.asyncState();
let lastQuery = null;

async function fetchData(query) {
  // Skip if same query
  if (query === lastQuery && dataState.isSuccess) {
    console.log('Using cached data');
    return;
  }

  lastQuery = query;

  await dataState.$execute(async (signal) => {
    const response = await fetch(`/api/search?q=${query}`, { signal });
    return response.json();
  });
}

// Duplicate calls - only first actually fetches
await fetchData('react');
await fetchData('react'); // Uses cache
await fetchData('vue');   // New fetch
```

### **Example 15: Progressive Loading**
```javascript
const pageState = ReactiveUtils.asyncState({ items: [], page: 1, hasMore: true });

async function loadMore() {
  if (pageState.loading || !pageState.data.hasMore) {
    return;
  }

  await pageState.$execute(async (signal) => {
    const page = pageState.data.page;
    const response = await fetch(`/api/items?page=${page}`, { signal });
    const newItems = await response.json();

    return {
      items: [...pageState.data.items, ...newItems],
      page: page + 1,
      hasMore: newItems.length > 0
    };
  });
}

ReactiveUtils.effect(() => {
  if (pageState.isSuccess) {
    console.log(`Loaded ${pageState.data.items.length} items`);
    console.log(`Page: ${pageState.data.page}`);
    console.log(`Has more: ${pageState.data.hasMore}`);
  }
});

// Load pages
await loadMore(); // Page 1
await loadMore(); // Page 2
await loadMore(); // Page 3
```

### **Example 16: Error Recovery with Data Preservation**
```javascript
const state = ReactiveUtils.asyncState();

// Load initial data
await state.$execute(async () => {
  return { id: 1, name: 'John' };
});

console.log('Initial data:', state.data); // { id: 1, name: 'John' }

// Attempt update that fails
const previousData = state.data;

await state.$execute(async () => {
  throw new Error('Update failed');
});

// Check state after error
console.log('After error:');
console.log('  data:', state.data); // Still { id: 1, name: 'John' }
console.log('  error:', state.error.message); // 'Update failed'
console.log('  isError:', state.isError); // true

// Data preserved, can show error and keep UI working
ReactiveUtils.effect(() => {
  if (state.isError && state.data) {
    console.log('Showing error but keeping previous data visible');
  }
});
```

### **Example 17: Request Cancellation Cascade**
```javascript
const parentState = ReactiveUtils.asyncState();
const childState = ReactiveUtils.asyncState();

// When parent request changes, cancel child
ReactiveUtils.effect(() => {
  if (parentState.loading) {
    // Parent started loading, cancel any child request
    childState.$abort();
  }
});

// Parent request
parentState.$execute(async () => {
  const response = await fetch('/api/parent');
  return response.json();
});

// Child request (will be cancelled if parent changes)
ReactiveUtils.effect(() => {
  if (parentState.isSuccess) {
    childState.$execute(async () => {
      const response = await fetch(`/api/child/${parentState.data.id}`);
      return response.json();
    });
  }
});
```

### **Example 18: Loading State Aggregation**
```javascript
const user = ReactiveUtils.asyncState();
const posts = ReactiveUtils.asyncState();
const comments = ReactiveUtils.asyncState();

// Aggregate loading state
ReactiveUtils.effect(() => {
  const allLoading = [user, posts, comments].every(s => s.loading);
  const anyLoading = [user, posts, comments].some(s => s.loading);
  const allSuccess = [user, posts, comments].every(s => s.isSuccess);
  const anyError = [user, posts, comments].some(s => s.isError);

  console.log('Loading states:');
  console.log(`  All loading: ${allLoading}`);
  console.log(`  Any loading: ${anyLoading}`);
  console.log(`  All success: ${allSuccess}`);
  console.log(`  Any error: ${anyError}`);

  const overlay = document.getElementById('loading-overlay');
  overlay.style.display = anyLoading ? 'block' : 'none';
});

// Trigger parallel loads
Promise.all([
  user.$execute(async () => fetch('/api/user').then(r => r.json())),
  posts.$execute(async () => fetch('/api/posts').then(r => r.json())),
  comments.$execute(async () => fetch('/api/comments').then(r => r.json()))
]);
```

### **Example 19: Request Metadata Tracking**
```javascript
const state = ReactiveUtils.asyncState();
const requestLog = [];

// Wrap execute to track metadata
const originalExecute = state.$execute;
state.$execute = async function(fn) {
  const startTime = Date.now();
  const requestId = this.requestId + 1;

  requestLog.push({
    requestId,
    startTime,
    status: 'pending'
  });

  const result = await originalExecute.call(this, fn);

  const logEntry = requestLog.find(r => r.requestId === requestId);
  logEntry.endTime = Date.now();
  logEntry.duration = logEntry.endTime - logEntry.startTime;
  logEntry.status = result.success ? 'success' : 'error';

  console.log('Request log:', requestLog);

  return result;
};

// Make requests
await state.$execute(async () => 'First');
await state.$execute(async () => 'Second');

console.log('Total requests:', requestLog.length);
console.log('Average duration:',
  requestLog.reduce((sum, r) => sum + r.duration, 0) / requestLog.length
);
```

### **Example 20: Complete CRUD Dashboard**
```javascript
const listState = ReactiveUtils.asyncState([]);
const createState = ReactiveUtils.asyncState();
const updateState = ReactiveUtils.asyncState();
const deleteState = ReactiveUtils.asyncState();

// Reactive dashboard
ReactiveUtils.effect(() => {
  const dashboard = {
    list: {
      loading: listState.loading,
      count: listState.data?.length || 0,
      hasData: listState.isSuccess,
      error: listState.error?.message
    },
    create: {
      loading: createState.loading,
      success: createState.isSuccess,
      error: createState.error?.message
    },
    update: {
      loading: updateState.loading,
      success: updateState.isSuccess,
      error: updateState.error?.message
    },
    delete: {
      loading: deleteState.loading,
      success: deleteState.isSuccess,
      error: deleteState.error?.message
    },
    anyLoading: [listState, createState, updateState, deleteState]
      .some(s => s.loading)
  };

  console.clear();
  console.log('📊 CRUD Dashboard:');
  console.log('─'.repeat(60));
  console.log(`List: ${dashboard.list.count} items | Loading: ${dashboard.list.loading}`);
  console.log(`Create: ${createState.loading ? 'Loading' : createState.isSuccess ? 'Success' : 'Idle'}`);
  console.log(`Update: ${updateState.loading ? 'Loading' : updateState.isSuccess ? 'Success' : 'Idle'}`);
  console.log(`Delete: ${deleteState.loading ? 'Loading' : deleteState.isSuccess ? 'Success' : 'Idle'}`);
  console.log('─'.repeat(60));
});

// CRUD operations
async function loadList() {
  await listState.$execute(async () => {
    const response = await fetch('/api/items');
    return response.json();
  });
}

async function createItem(data) {
  await createState.$execute(async () => {
    const response = await fetch('/api/items', {
      method: 'POST',
      body: JSON.stringify(data)
    });
    return response.json();
  });

  if (createState.isSuccess) {
    loadList(); // Refresh list
  }
}

async function updateItem(id, data) {
  await updateState.$execute(async () => {
    const response = await fetch(`/api/items/${id}`, {
      method: 'PUT',
      body: JSON.stringify(data)
    });
    return response.json();
  });

  if (updateState.isSuccess) {
    loadList();
  }
}

async function deleteItem(id) {
  await deleteState.$execute(async () => {
    await fetch(`/api/items/${id}`, { method: 'DELETE' });
  });

  if (deleteState.isSuccess) {
    loadList();
  }
}
```

---

## **Common Patterns**

### **Pattern 1: Check Loading State**
```javascript
if (state.loading) {
  showSpinner();
}
```

### **Pattern 2: Display Data**
```javascript
if (state.isSuccess) {
  renderData(state.data);
}
```

### **Pattern 3: Show Error**
```javascript
if (state.isError) {
  showError(state.error.message);
}
```

### **Pattern 4: Show Placeholder**
```javascript
if (state.isIdle) {
  showPlaceholder();
}
```

### **Pattern 5: Abort on Unmount**
```javascript
onUnmount(() => {
  state.$abort();
});
```

---

## **Property Structures**

### **State Properties**
```javascript
{
  data: any | null,              // Result data
  loading: boolean,              // Loading state
  error: Error | null,           // Error object
  requestId: number,             // Request counter
  abortController: AbortController | null  // Abort controller
}
```

### **Computed Properties**
```javascript
{
  isSuccess: boolean,  // !loading && !error && data !== null
  isError: boolean,    // !loading && error !== null
  isIdle: boolean      // !loading && data === null && error === null
}
```

---

## **State Lifecycle**

```javascript
// 1. Initial (Idle)
{ data: null, loading: false, error: null, requestId: 0, isIdle: true }

// 2. Execute called
{ data: null, loading: true, error: null, requestId: 1, isIdle: false }

// 3. Success
{ data: {...}, loading: false, error: null, requestId: 1, isSuccess: true }

// 4. New request (auto-abort previous)
{ data: {...}, loading: true, error: null, requestId: 2, isSuccess: false }

// 5. Error
{ data: {...}, loading: false, error: Error, requestId: 2, isError: true }

// 6. Reset
{ data: null, loading: false, error: null, requestId: 0, isIdle: true }
```

---

## **State Transitions**

| From | To | Trigger |
|------|----|----|
| Idle | Loading | `$execute()` called |
| Loading | Success | Request completes successfully |
| Loading | Error | Request throws error |
| Loading | Loading | New `$execute()` (aborts previous) |
| Any | Idle | `$reset()` called |
| Loading | Idle | `$abort()` called |

---

## **Property Comparison Table**

| Property | Type | Default | Updates On | Use Case |
|----------|------|---------|------------|----------|
| `data` | any | null | Success | Display fetched data |
| `loading` | boolean | false | Start/end | Show spinner |
| `error` | Error\|null | null | Error | Show error message |
| `requestId` | number | 0 | Each request | Race prevention |
| `abortController` | AbortController\|null | null | Start/end | Cancel requests |
| `isSuccess` | boolean | computed | State changes | Success rendering |
| `isError` | boolean | computed | State changes | Error rendering |
| `isIdle` | boolean | computed | State changes | Placeholder rendering |

---

## **Reactive Behavior**

All properties are reactive and trigger effects:

```javascript
const state = ReactiveUtils.asyncState();

ReactiveUtils.effect(() => {
  console.log('Data:', state.data);          // Tracks data
  console.log('Loading:', state.loading);    // Tracks loading
  console.log('Error:', state.error);        // Tracks error
  console.log('RequestId:', state.requestId); // Tracks requestId
  console.log('Abort:', state.abortController); // Tracks abortController
  console.log('Success:', state.isSuccess);  // Tracks success
  console.log('Error:', state.isError);      // Tracks error state
  console.log('Idle:', state.isIdle);        // Tracks idle
});

// Any change triggers the effect
await state.$execute(async () => 'data');
```

---

## **Key Takeaways**

1. **`data`** - Stores successful async operation results
2. **`loading`** - Boolean indicating operation in progress
3. **`error`** - Stores error from failed operations
4. **`requestId`** - Counter preventing race conditions
5. **`abortController`** - Enables request cancellation
6. **`isSuccess`** - Computed: data loaded without errors
7. **`isError`** - Computed: has error, not loading
8. **`isIdle`** - Computed: pristine state, no data/error
9. **Reactive** - All trigger effects automatically
10. **Race-Safe** - RequestId ensures latest request wins
11. **Cancellable** - AbortController enables cleanup
12. **Type-Safe** - Predictable property types

---

## **Summary**

Async state properties provide complete lifecycle management for asynchronous operations through 8 reactive properties: 5 state properties (`data`, `loading`, `error`, `requestId`, `abortController`) and 3 computed properties (`isSuccess`, `isError`, `isIdle`). The `data` property stores successful results, `loading` indicates operation status, `error` captures failures, `requestId` prevents race conditions by tracking request sequence, and `abortController` enables request cancellation. The three computed properties provide convenient state checks: `isSuccess` confirms data loaded without errors, `isError` indicates failure state, and `isIdle` detects pristine/reset state. All properties are reactive and automatically trigger effects, enabling declarative UI updates. The requestId system ensures only the latest request updates state, preventing race conditions in rapid sequential requests. The AbortController integration provides graceful cancellation for in-flight requests, essential for cleanup on component unmount or rapid user interactions. Use these properties together to build robust async data flows with loading states, error handling, request cancellation, and race-condition-safe data fetching.
