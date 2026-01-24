# `updateAll()` - Unified State and DOM Updates

## Quick Start (30 seconds)

```javascript
// Create state
const app = state({
  count: 0,
  title: 'Counter App'
});

// Update both state AND DOM in one call
updateAll(app, {
  count: app.count + 1,
  title: 'Updated Counter',
  // DOM updates use selectors
  '#counter': { textContent: app.count + 1 },
  '#title': { textContent: 'Updated Counter' }
});

// State updated: app.count is now 1
// DOM updated: Elements with IDs automatically sync
```

**That's it.** Update reactive state and DOM elements in a single unified function call. No separate update steps needed.

---

## What is `updateAll()`?

`updateAll()` **provides unified updates for both reactive state and DOM elements in a single function call**. It combines state property updates with DOM element updates using CSS selectors.

Think of it as **the one-stop update function** — instead of separately updating state and then updating DOM, you do both at once in a single call.

**In practical terms:** Any time you need to change state and reflect those changes in the DOM, `updateAll()` handles both operations simultaneously and efficiently.

---

## Syntax

```javascript
// Update state and/or DOM
updateAll(stateObject, updates);

// Updates object
const updates = {
  // State properties (no selector)
  propertyName: newValue,

  // DOM updates (with CSS selector)
  '#elementId': { /* property updates */ },
  '.className': { /* property updates */ }
};

// Example
const user = state({
  name: 'Alice',
  score: 100
});

updateAll(user, {
  // Update state
  score: 150,
  // Update DOM
  '#userName': { textContent: 'Alice' },
  '#userScore': { textContent: '150' }
});
```

**Parameters:**
- `stateObject` - Reactive state object to update
- `updates` - Object containing state property updates and/or DOM selector updates

**Returns:**
- The state object (for chaining)

**Important:**
- Keys without selectors (`#`, `.`, or element tag) update state properties
- Keys with selectors update DOM elements
- All updates are batched automatically for performance

---

## Why Does This Exist?

### The Problem Without updateAll()

Updating state and DOM separately is verbose and error-prone:

```javascript
// ❌ Vanilla JavaScript - separate state and DOM updates
const appData = {
  count: 0,
  username: 'Guest'
};

function incrementCounter() {
  // Update state
  appData.count++;

  // Update DOM manually
  document.getElementById('counter').textContent = appData.count;
  document.getElementById('status').textContent = `${appData.username}: ${appData.count}`;

  // More DOM updates...
  const btn = document.getElementById('btn');
  btn.className = appData.count > 10 ? 'btn-success' : 'btn-default';
}

function changeUser(name) {
  // Update state
  appData.username = name;

  // Update DOM manually
  document.getElementById('username').textContent = name;
  document.getElementById('status').textContent = `${name}: ${appData.count}`;
}
```

**Problems:**
❌ **Scattered updates** - State and DOM updated in different places
❌ **Manual sync** - Must remember to update all related DOM elements
❌ **Error-prone** - Easy to forget DOM updates or use stale values
❌ **Verbose** - Repetitive `document.getElementById` calls
❌ **No batching** - Multiple DOM updates trigger multiple reflows

### The Solution with `updateAll()`

```javascript
// ✅ DOM Helpers + Reactive State with updateAll() - unified updates
const app = state({
  count: 0,
  username: 'Guest'
});

function incrementCounter() {
  updateAll(app, {
    count: app.count + 1,
    '#counter': { textContent: app.count + 1 },
    '#status': { textContent: `${app.username}: ${app.count + 1}` },
    '#btn': { className: app.count + 1 > 10 ? 'btn-success' : 'btn-default' }
  });
}

function changeUser(name) {
  updateAll(app, {
    username: name,
    '#username': { textContent: name },
    '#status': { textContent: `${name}: ${app.count}` }
  });
}
```

**Benefits:**
✅ **Unified updates** - State and DOM updated together
✅ **Single function call** - Everything in one place
✅ **Automatic batching** - All updates happen efficiently
✅ **Concise syntax** - No repeated `document.getElementById`
✅ **Reduced errors** - Updates coupled together, harder to forget

---

## Mental Model: Unified Update Command

Think of `updateAll()` like **a single command that updates everything at once**:

**Without updateAll() (Separate Steps):**
```
┌─────────────────────────────┐
│  Multiple Separate Steps    │
│                             │
│  Step 1: Update state       │
│  → app.count = 5            │
│                             │
│  Step 2: Update DOM         │
│  → elem.textContent = 5     │
│                             │
│  Step 3: Update more DOM    │
│  → btn.disabled = false     │
│                             │
│  Easy to forget steps!      │
└─────────────────────────────┘
```

**With updateAll() (Single Command):**
```
┌─────────────────────────────┐
│  Single Unified Command     │
│                             │
│  updateAll(app, {           │
│    count: 5,                │
│    '#counter': {...},       │
│    '#btn': {...}            │
│  })                         │
│                             │
│  Everything updates once!   │
│  Batched automatically!     │
└─────────────────────────────┘
```

`updateAll()` is **one command for all updates** — state and DOM together.

---

## How Does It Work?

`updateAll()` processes updates and routes them to state or DOM:

```
Call updateAll(state, updates)
    ↓
Iterate through update keys
    ↓
For each key:
  ┌─────────────────────┐
  │ Is it a selector?   │
  │ (#id, .class, tag)  │
  └─────────────────────┘
         ↓          ↓
        Yes         No
         ↓          ↓
    Update DOM   Update State
    (by selector) (by property)
    ↓              ↓
    └──────┬───────┘
           ↓
    All updates batched
           ↓
    Return state object
```

**Key behaviors:**
- Keys starting with `#`, `.`, or tag names are **DOM selectors**
- Other keys are **state properties**
- All updates are **batched** for performance
- State changes **trigger reactivity**

---

## Basic Usage

### Example 1: Counter with Status

```javascript
const counter = state({
  count: 0,
  lastUpdated: null
});

// Increment and update display
function increment() {
  const newCount = counter.count + 1;
  const now = new Date().toLocaleTimeString();

  updateAll(counter, {
    // State updates
    count: newCount,
    lastUpdated: now,

    // DOM updates
    '#count': { textContent: newCount },
    '#lastUpdated': { textContent: `Updated: ${now}` },
    '#incrementBtn': {
      className: newCount >= 10 ? 'btn-max' : 'btn-default'
    }
  });
}
```

---

### Example 2: Form Validation

```javascript
const form = state({
  email: '',
  emailValid: false
});

// Update email and validate
function updateEmail(value) {
  const isValid = value.includes('@') && value.includes('.');

  updateAll(form, {
    // State updates
    email: value,
    emailValid: isValid,

    // DOM updates
    '#emailInput': {
      className: isValid ? 'input-valid' : 'input-invalid'
    },
    '#emailError': {
      textContent: isValid ? '' : 'Invalid email format',
      style: { display: isValid ? 'none' : 'block' }
    }
  });
}
```

---

### Example 3: Shopping Cart

```javascript
const cart = state({
  items: [],
  total: 0
});

// Add item and update display
function addToCart(item) {
  const newItems = [...cart.items, item];
  const newTotal = newItems.reduce((sum, i) => sum + i.price, 0);

  updateAll(cart, {
    // State updates
    items: newItems,
    total: newTotal,

    // DOM updates
    '#cartCount': { textContent: newItems.length },
    '#cartTotal': { textContent: `$${newTotal.toFixed(2)}` },
    '#emptyCart': {
      style: { display: newItems.length === 0 ? 'block' : 'none' }
    },
    '#cartItems': {
      innerHTML: newItems.map(i => `<li>${i.name} - $${i.price}</li>`).join('')
    }
  });
}
```

---

### Example 4: User Profile

```javascript
const profile = state({
  firstName: 'Alice',
  lastName: 'Johnson',
  avatar: '/avatars/default.png'
});

// Update profile
function updateProfile(updates) {
  const fullName = `${updates.firstName} ${updates.lastName}`;

  updateAll(profile, {
    // State updates
    firstName: updates.firstName,
    lastName: updates.lastName,
    avatar: updates.avatar || profile.avatar,

    // DOM updates
    '#profileName': { textContent: fullName },
    '#firstName': { textContent: updates.firstName },
    '#lastName': { textContent: updates.lastName },
    '#avatar': { src: updates.avatar || profile.avatar },
    '.user-initials': {
      textContent: `${updates.firstName[0]}${updates.lastName[0]}`
    }
  });
}
```

---

### Example 5: Loading States

```javascript
const api = state({
  loading: false,
  data: null,
  error: null
});

// Start loading
function startLoading() {
  updateAll(api, {
    // State updates
    loading: true,
    error: null,

    // DOM updates
    '#loadingSpinner': {
      style: { display: 'block' }
    },
    '#content': {
      style: { opacity: '0.5' }
    },
    '#loadBtn': { disabled: true }
  });
}

// Finish loading
function finishLoading(data) {
  updateAll(api, {
    // State updates
    loading: false,
    data: data,

    // DOM updates
    '#loadingSpinner': {
      style: { display: 'none' }
    },
    '#content': {
      style: { opacity: '1' },
      innerHTML: renderData(data)
    },
    '#loadBtn': { disabled: false }
  });
}
```

---

### Example 6: Theme Switcher

```javascript
const app = state({
  theme: 'light',
  fontSize: 16
});

// Toggle theme
function toggleTheme() {
  const newTheme = app.theme === 'light' ? 'dark' : 'light';

  updateAll(app, {
    // State updates
    theme: newTheme,

    // DOM updates
    body: {
      className: `theme-${newTheme}`,
      style: {
        backgroundColor: newTheme === 'dark' ? '#1a1a1a' : '#ffffff',
        color: newTheme === 'dark' ? '#ffffff' : '#000000'
      }
    },
    '#themeToggle': {
      textContent: newTheme === 'dark' ? '☀️ Light' : '🌙 Dark'
    }
  });
}

// Change font size
function changeFontSize(delta) {
  const newSize = app.fontSize + delta;

  updateAll(app, {
    // State updates
    fontSize: newSize,

    // DOM updates
    body: {
      style: { fontSize: `${newSize}px` }
    },
    '#fontSize': { textContent: `${newSize}px` }
  });
}
```

---

### Example 7: Search Results

```javascript
const search = state({
  query: '',
  results: [],
  searching: false
});

// Perform search
async function performSearch(query) {
  // Start search
  updateAll(search, {
    query: query,
    searching: true,
    '#searchInput': { value: query },
    '#searchSpinner': { style: { display: 'inline-block' } }
  });

  // Fetch results
  const results = await fetchResults(query);

  // Update with results
  updateAll(search, {
    results: results,
    searching: false,
    '#searchResults': {
      innerHTML: results.map(r => `<div>${r.title}</div>`).join('')
    },
    '#resultCount': { textContent: `${results.length} results` },
    '#searchSpinner': { style: { display: 'none' } }
  });
}
```

---

### Example 8: Progress Tracker

```javascript
const progress = state({
  current: 0,
  total: 100,
  status: 'idle'
});

// Update progress
function updateProgress(current) {
  const percentage = (current / progress.total) * 100;
  const isComplete = current >= progress.total;

  updateAll(progress, {
    // State updates
    current: current,
    status: isComplete ? 'complete' : 'in-progress',

    // DOM updates
    '#progressBar': {
      style: { width: `${percentage}%` },
      className: isComplete ? 'progress-complete' : 'progress-active'
    },
    '#progressText': { textContent: `${current} / ${progress.total}` },
    '#progressPercent': { textContent: `${percentage.toFixed(0)}%` },
    '#completeIcon': {
      style: { display: isComplete ? 'inline' : 'none' }
    }
  });
}
```

---

## Advanced Usage: Bulk Updates with Classes

```javascript
const app = state({
  notifications: [],
  unreadCount: 0
});

// Add notification and update multiple elements
function addNotification(message) {
  const newNotifications = [...app.notifications, {
    id: Date.now(),
    message,
    read: false
  }];

  const unread = newNotifications.filter(n => !n.read).length;

  updateAll(app, {
    // State updates
    notifications: newNotifications,
    unreadCount: unread,

    // Update multiple elements by class
    '.notification-badge': {
      textContent: unread,
      style: { display: unread > 0 ? 'inline' : 'none' }
    },
    '.notification-list': {
      innerHTML: newNotifications
        .map(n => `<li class="${n.read ? 'read' : 'unread'}">${n.message}</li>`)
        .join('')
    },
    '#notificationCount': { textContent: `${unread} unread` }
  });
}
```

---

## Common Patterns

### Pattern 1: Toggle State with Visual Feedback

```javascript
function toggleFeature(enabled) {
  updateAll(app, {
    featureEnabled: enabled,
    '#featureToggle': {
      textContent: enabled ? 'ON' : 'OFF',
      className: enabled ? 'btn-success' : 'btn-default'
    }
  });
}
```

### Pattern 2: Update with Validation

```javascript
function updateField(value) {
  const isValid = validateValue(value);

  updateAll(form, {
    value: value,
    isValid: isValid,
    '#field': { className: isValid ? 'valid' : 'invalid' },
    '#error': { textContent: isValid ? '' : 'Invalid value' }
  });
}
```

### Pattern 3: State + Multiple DOM Targets

```javascript
function updateCounter(newValue) {
  updateAll(counter, {
    count: newValue,
    '#mainCounter': { textContent: newValue },
    '#sidebarCounter': { textContent: newValue },
    '.counter-badge': { textContent: newValue }
  });
}
```

---

## Key Takeaways

✅ **Unified updates** - State and DOM updated in one call
✅ **Automatic batching** - All updates happen efficiently
✅ **Concise syntax** - No repetitive DOM queries
✅ **Reduced errors** - Updates coupled together
✅ **CSS selectors** - Use `#id`, `.class`, or tag names for DOM
✅ **Reactive** - State changes trigger effects

---

## What's Next?

- **`set()`** - Functional state updates
- **`batch()`** - Manual batching control
- **`notify()`** - Manual dependency notifications

---

## Summary

`updateAll()` **provides unified updates for reactive state and DOM elements in a single function call**. It automatically routes updates to state properties or DOM elements based on key patterns, batching everything for performance.

**The magic formula:**
```
updateAll(state, {
  prop: value,        // State update
  '#id': { ... }      // DOM update
}) =
  State + DOM updated together
──────────────────────────────
One call for all updates
```

Think of it as **the one-stop update function** — update state and DOM together without separate steps. Perfect for keeping state and UI in sync with minimal code.
