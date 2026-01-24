# `createState()` - State + Bindings in One Call

## Quick Start (30 seconds)

```javascript
// Create reactive state WITH automatic DOM bindings in one call
const user = createState(
  { name: 'Alice', status: 'online' },
  {
    '#user-name': () => user.name,
    '#user-status': () => user.status
  }
);

user.name = 'Bob'; // ✨ State updates + DOM updates automatically!
```

**That's it.** One function call creates your reactive state AND sets up all DOM synchronization. No separate `bindings()` call needed.

---

## What is `createState()`?

`createState()` is **the all-in-one state creation function**. It combines `state()` + `bindings()` into a single call, creating reactive state that's automatically wired to your DOM.

Think of it as **hiring a personal assistant who not only manages your data but also updates all your displays automatically**. You define the data and where it should appear, and everything else is handled for you.

**In practical terms:** Instead of calling `state()` to create reactive data, then calling `bindings()` to sync it to DOM, you do both in one step. Less code, clearer intent.

---

## Syntax

```javascript
// Basic usage
const myState = createState(initialState, bindingDefinitions);

// Just state (no bindings)
const user = createState({ name: 'Alice', email: 'alice@example.com' });

// State with bindings
const counter = createState(
  { count: 0 },
  {
    '#counter-display': () => counter.count,
    '#counter-badge': () => counter.count
  }
);

// Complex example
const app = createState(
  {
    user: { name: 'Alice', role: 'Admin' },
    stats: { views: 1250, likes: 342 }
  },
  {
    '#user-name': () => app.user.name,
    '#user-role': () => app.user.role,
    '#view-count': () => app.stats.views.toLocaleString(),
    '#like-count': () => app.stats.likes.toLocaleString()
  }
);
```

**Parameters:**
- `initialState` - Object with initial state values
- `bindingDefinitions` (optional) - Object mapping selectors to functions

**Returns:**
- Reactive state proxy with all binding effects already set up

---

## Why Does This Exist?

### The Problem Without createState

Let's build a user profile that displays data and updates the DOM:

```javascript
// ❌ Vanilla JavaScript - manual everything
const userData = {
  name: 'Alice Johnson',
  email: 'alice@example.com',
  role: 'Admin',
  status: 'online',
  lastLogin: '2 hours ago'
};

// Manual DOM updates every single time
function updateDisplay() {
  document.getElementById('user-name').textContent = userData.name;
  document.getElementById('user-email').textContent = userData.email;
  document.getElementById('user-role').textContent = userData.role;
  document.getElementById('user-status').textContent = userData.status;
  document.getElementById('last-login').textContent = userData.lastLogin;
}

// Must call manually on every change
userData.name = 'Bob Smith';
updateDisplay(); // Easy to forget!

userData.status = 'away';
updateDisplay(); // Must remember again!

// Event handlers also need manual updates
document.getElementById('logout-btn').addEventListener('click', () => {
  userData.status = 'offline';
  updateDisplay(); // Error-prone!
});
```

**What's the Real Issue?**

```
Create Plain Data
    ↓
Write Manual Update Function
    ↓
Wire Up DOM Manually
    ↓
Remember to Call Update Function
    ↓
Easy to Forget = Bugs
```

**Problems:**
❌ **Manual synchronization** - Must call `updateDisplay()` after every change
❌ **Two-step setup** - Create data, then wire DOM separately
❌ **Error-prone** - Forget one `updateDisplay()` call = stale UI
❌ **Verbose** - Lots of repetitive `document.getElementById` calls
❌ **Hard to maintain** - Changes require updating multiple places

### The Solution with `createState()`

```javascript
// ✅ DOM Helpers + Reactive State - automatic everything
const user = createState(
  {
    name: 'Alice Johnson',
    email: 'alice@example.com',
    role: 'Admin',
    status: 'online',
    lastLogin: '2 hours ago'
  },
  {
    '#user-name': () => user.name,
    '#user-email': () => user.email,
    '#user-role': () => user.role,
    '#user-status': () => user.status,
    '#last-login': () => user.lastLogin
  }
);

// State updates automatically sync to DOM
user.name = 'Bob Smith'; // ✨ DOM updates automatically!
user.status = 'away'; // ✨ DOM updates automatically!

// Event handlers using bulk event binding
Elements.update({
  logoutBtn: {
    addEventListener: ['click', () => {
      user.status = 'offline';
      // ✨ No manual update call needed!
    }]
  }
});
```

**What Just Happened?**

```
One createState() Call
    ↓
Creates Reactive State
    ↓
Sets Up All DOM Bindings
    ↓
Changes Automatically Flow to UI
    ↓
Zero Manual Updates Needed
```

**Benefits:**
✅ **All-in-one** - State creation + DOM sync in one call
✅ **Automatic updates** - Change state, DOM updates automatically
✅ **Declarative** - Define what shows where, system handles how
✅ **Less code** - 70% less boilerplate than vanilla JS
✅ **Impossible to forget** - No manual update calls to forget

---

## Mental Model: Smart Dashboard Setup

Think of `createState()` like **setting up a complete smart home dashboard** in one installation:

**Without createState (DIY Setup):**
```
┌─────────────────────────────┐
│  Manual Setup               │
│                             │
│  1. Install sensors         │
│  2. Buy displays            │
│  3. Wire each sensor        │
│  4. Program each display    │
│  5. Test connections        │
│  6. Debug wiring issues     │
│                             │
│  Hours of work!             │
└─────────────────────────────┘
```

**With createState (All-in-One Install):**
```
┌─────────────────────────────┐
│  One-Call Setup             │
│                             │
│  📦 Pre-configured Package  │
│                             │
│  ✓ Sensors installed        │
│  ✓ Displays installed       │
│  ✓ Everything wired         │
│  ✓ All synced automatically │
│                             │
│  Ready to use!              │
└─────────────────────────────┘
```

You get **reactive state + automatic DOM sync** in a single function call.

---

## How Does It Work?

Internally, `createState()` does two things:

```
Your Call
    ↓
createState(data, bindings)
    ↓
Internally Does:
    ↓
1. state = state(data)
2. bindings(bindings)
    ↓
Returns: Reactive state with bindings active
```

**Step-by-step:**

1️⃣ **You call createState:**
```javascript
const counter = createState(
  { count: 0 },
  { '#display': () => counter.count }
);
```

2️⃣ **Internally creates reactive state:**
```javascript
// Like calling: const counter = state({ count: 0 });
```

3️⃣ **Internally sets up bindings:**
```javascript
// Like calling: bindings({ '#display': () => counter.count });
```

4️⃣ **Returns the reactive state:**
```javascript
// counter is now reactive AND bound to DOM
```

**Key Insight:**
`createState()` is just **syntactic sugar** that saves you from making two separate calls. It's the same as calling `state()` then `bindings()`, but cleaner.

---

## Basic Usage

### Example 1: Simple Counter with Display

```javascript
const counter = createState(
  { count: 0 },
  { '#count-display': () => counter.count }
);

// Update counter using bulk event binding
Elements.update({
  incrementBtn: {
    addEventListener: ['click', () => {
      counter.count++;
      // ✨ Display updates automatically
    }]
  }
});
```

**What's happening?**
- State created: `{ count: 0 }`
- Binding created: `#count-display` shows `counter.count`
- Changes to `counter.count` automatically update the display

---

### Example 2: User Profile

```javascript
const profile = createState(
  {
    name: 'Alice Johnson',
    email: 'alice@example.com',
    avatar: '👩‍💼'
  },
  {
    '#profile-name': () => profile.name,
    '#profile-email': () => profile.email,
    '#profile-avatar': () => profile.avatar
  }
);

// Later: update profile
profile.name = 'Alice Smith';
// ✨ All DOM elements update automatically
```

---

### Example 3: Dashboard Stats

```javascript
const dashboard = createState(
  {
    users: 1250,
    revenue: 45780,
    orders: 342
  },
  {
    '#user-count': () => dashboard.users.toLocaleString(),
    '#revenue': () => `$${dashboard.revenue.toLocaleString()}`,
    '#order-count': () => dashboard.orders.toLocaleString(),
    '#avg-order': () => {
      const avg = dashboard.revenue / dashboard.orders;
      return `$${avg.toFixed(2)}`;
    }
  }
);

// Simulate stats update
setInterval(() => {
  dashboard.users += Math.floor(Math.random() * 10);
  dashboard.revenue += Math.floor(Math.random() * 1000);
  dashboard.orders += Math.floor(Math.random() * 5);
  // ✨ All displays update automatically
}, 3000);
```

---

### Example 4: Theme Toggle

```javascript
const ui = createState(
  { darkMode: false },
  {
    '#theme-label': () => ui.darkMode ? 'Dark Mode' : 'Light Mode',
    '#theme-icon': () => ui.darkMode ? '🌙' : '☀️'
  }
);

// Additional effect for body class using bulk updates
effect(() => {
  Elements.update({
    body: {
      classList: {
        toggle: ['dark-mode', ui.darkMode]
      }
    }
  });
});

// Toggle with bulk event binding
Elements.update({
  themeToggle: {
    addEventListener: ['click', () => {
      ui.darkMode = !ui.darkMode;
    }]
  }
});
```

---

### Example 5: Form State with Validation

```javascript
const form = createState(
  {
    email: '',
    password: '',
    confirmPassword: ''
  },
  {
    '#email-value': () => form.email,
    '#email-error': () => {
      const valid = form.email.includes('@') && form.email.includes('.');
      return valid || form.email === '' ? '' : 'Invalid email';
    },
    '#password-match': () => {
      if (form.confirmPassword === '') return '';
      return form.password === form.confirmPassword
        ? '✓ Passwords match'
        : '✗ Passwords do not match';
    }
  }
);

// Input handlers using bulk event binding
Elements.update({
  emailInput: {
    addEventListener: ['input', (e) => {
      form.email = e.target.value;
    }]
  },
  passwordInput: {
    addEventListener: ['input', (e) => {
      form.password = e.target.value;
    }]
  },
  confirmPasswordInput: {
    addEventListener: ['input', (e) => {
      form.confirmPassword = e.target.value;
    }]
  }
});
```

---

### Example 6: Search with Results

```javascript
const search = createState(
  {
    query: '',
    results: []
  },
  {
    '#search-query': () => search.query,
    '#result-count': () => `${search.results.length} results`,
    '#results-list': () => {
      return search.results
        .map(item => `<li>${item}</li>`)
        .join('');
    }
  }
);

const allItems = ['Apple', 'Banana', 'Cherry', 'Date', 'Elderberry'];

// Search handler using bulk event binding
Elements.update({
  searchInput: {
    addEventListener: ['input', (e) => {
      search.query = e.target.value;
      search.results = allItems.filter(item =>
        item.toLowerCase().includes(search.query.toLowerCase())
      );
      // ✨ All result displays update automatically
    }]
  }
});
```

---

### Example 7: Shopping Cart Summary

```javascript
const cart = createState(
  {
    items: [
      { name: 'Laptop', price: 999, qty: 1 },
      { name: 'Mouse', price: 29, qty: 2 }
    ],
    tax: 0.08
  },
  {
    '#item-count': () => {
      const total = cart.items.reduce((sum, item) => sum + item.qty, 0);
      return `${total} items`;
    },
    '#subtotal': () => {
      const subtotal = cart.items.reduce((sum, item) =>
        sum + (item.price * item.qty), 0
      );
      return `$${subtotal.toFixed(2)}`;
    },
    '#tax-amount': () => {
      const subtotal = cart.items.reduce((sum, item) =>
        sum + (item.price * item.qty), 0
      );
      return `$${(subtotal * cart.tax).toFixed(2)}`;
    },
    '#total': () => {
      const subtotal = cart.items.reduce((sum, item) =>
        sum + (item.price * item.qty), 0
      );
      return `$${(subtotal * (1 + cart.tax)).toFixed(2)}`;
    }
  }
);

// Update quantity
cart.items[1].qty = 3;
// ✨ All totals recalculate automatically
```

---

### Example 8: Live Clock

```javascript
const clock = createState(
  { time: new Date() },
  {
    '#current-time': () => clock.time.toLocaleTimeString(),
    '#current-date': () => clock.time.toLocaleDateString(),
    '#day-of-week': () => {
      const days = ['Sunday', 'Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday'];
      return days[clock.time.getDay()];
    }
  }
);

// Update every second
setInterval(() => {
  clock.time = new Date();
}, 1000);
```

---

### Example 9: Notification System

```javascript
const notifications = createState(
  {
    count: 0,
    latest: '',
    unread: []
  },
  {
    '#notification-count': () => notifications.count,
    '#notification-badge': () => {
      return notifications.count > 0 ? notifications.count : '';
    },
    '#latest-notification': () => notifications.latest,
    '#notification-list': () => {
      return notifications.unread
        .map(msg => `<li>${msg}</li>`)
        .join('');
    }
  }
);

// Add notification
function addNotification(message) {
  notifications.unread.push(message);
  notifications.count = notifications.unread.length;
  notifications.latest = message;
  // ✨ All displays update automatically
}

addNotification('New message from Bob');
```

---

### Example 10: Multi-Language Support

```javascript
const app = createState(
  {
    language: 'en',
    translations: {
      en: {
        welcome: 'Welcome',
        logout: 'Logout',
        settings: 'Settings'
      },
      es: {
        welcome: 'Bienvenido',
        logout: 'Cerrar sesión',
        settings: 'Configuración'
      },
      fr: {
        welcome: 'Bienvenue',
        logout: 'Déconnexion',
        settings: 'Paramètres'
      }
    }
  },
  {
    '#welcome-text': () => app.translations[app.language].welcome,
    '#logout-text': () => app.translations[app.language].logout,
    '#settings-text': () => app.translations[app.language].settings,
    '#current-language': () => app.language.toUpperCase()
  }
);

// Change language using bulk event binding
Elements.update({
  languageSelect: {
    addEventListener: ['change', (e) => {
      app.language = e.target.value;
      // ✨ All text updates to new language
    }]
  }
});
```

---

## Advanced Usage: Without Bindings

You can use `createState()` without bindings if you just want reactive state:

```javascript
// Just creates reactive state (same as state())
const user = createState({
  name: 'Alice',
  email: 'alice@example.com'
});

// Set up bindings later if needed
bindings({
  '#user-name': () => user.name,
  '#user-email': () => user.email
});
```

This is equivalent to just calling `state()`, but keeps your API consistent.

---

## Advanced Usage: Computed Values in Bindings

```javascript
const products = createState(
  {
    items: [
      { name: 'Laptop', price: 999, inStock: true },
      { name: 'Mouse', price: 29, inStock: false },
      { name: 'Keyboard', price: 79, inStock: true }
    ],
    filter: 'all'
  },
  {
    '#total-products': () => products.items.length,
    '#in-stock-count': () => {
      return products.items.filter(p => p.inStock).length;
    },
    '#out-of-stock-count': () => {
      return products.items.filter(p => !p.inStock).length;
    },
    '#product-list': () => {
      const filtered = products.filter === 'all'
        ? products.items
        : products.items.filter(p =>
            products.filter === 'inStock' ? p.inStock : !p.inStock
          );

      return filtered
        .map(p => `
          <li class="${p.inStock ? 'in-stock' : 'out-of-stock'}">
            ${p.name} - $${p.price}
            ${p.inStock ? '✓' : '✗'}
          </li>
        `)
        .join('');
    }
  }
);
```

---

## Advanced Usage: Nested State

```javascript
const app = createState(
  {
    user: {
      profile: {
        firstName: 'Alice',
        lastName: 'Johnson',
        email: 'alice@example.com'
      },
      preferences: {
        theme: 'light',
        notifications: true
      }
    },
    settings: {
      language: 'en',
      timezone: 'UTC'
    }
  },
  {
    '#full-name': () => {
      return `${app.user.profile.firstName} ${app.user.profile.lastName}`;
    },
    '#user-email': () => app.user.profile.email,
    '#theme-display': () => app.user.preferences.theme,
    '#language-display': () => app.settings.language
  }
);

// Deep updates work
app.user.profile.firstName = 'Bob';
// ✨ Full name display updates automatically
```

---

## Advanced Usage: Array Methods with Bindings

```javascript
const todos = createState(
  { items: [] },
  {
    '#todo-count': () => todos.items.length,
    '#todo-list': () => {
      return todos.items
        .map((item, index) => `
          <li>
            <input type="checkbox" ${item.done ? 'checked' : ''}
                   data-index="${index}">
            <span>${item.text}</span>
          </li>
        `)
        .join('');
    },
    '#active-count': () => {
      return todos.items.filter(t => !t.done).length;
    }
  }
);

// Add todo using bulk event binding
Elements.update({
  addTodoBtn: {
    addEventListener: ['click', () => {
      todos.items.push({
        text: Elements.todoInput.value,
        done: false
      });
      Elements.todoInput.value = '';
      // ✨ All todo displays update automatically
    }]
  }
});
```

---

## createState() vs state() + bindings()

Both approaches are equivalent:

### Approach 1: createState (All-in-One)
```javascript
const user = createState(
  { name: 'Alice' },
  { '#user-name': () => user.name }
);
```

### Approach 2: Separate Calls
```javascript
const user = state({ name: 'Alice' });
bindings({ '#user-name': () => user.name });
```

**When to use createState():**
✅ You know your bindings upfront
✅ You want cleaner, more declarative code
✅ You're setting up state for a specific component/feature
✅ You prefer all-in-one initialization

**When to use state() + bindings():**
✅ You need state first, bindings later
✅ Bindings depend on runtime conditions
✅ You're adding bindings dynamically
✅ You prefer separation of concerns

---

## Real-World Example: Complete User Dashboard

```javascript
const dashboard = createState(
  {
    user: {
      name: 'Alice Johnson',
      role: 'Admin',
      avatar: '👩‍💼',
      email: 'alice@example.com'
    },
    stats: {
      views: 12450,
      likes: 3421,
      comments: 892,
      shares: 156
    },
    activity: {
      lastLogin: new Date(),
      isOnline: true
    },
    notifications: {
      count: 5,
      messages: ['New comment', 'New follower', 'Post liked']
    }
  },
  {
    // User info
    '#user-name': () => dashboard.user.name,
    '#user-role': () => dashboard.user.role,
    '#user-avatar': () => dashboard.user.avatar,
    '#user-email': () => dashboard.user.email,

    // Stats
    '#view-count': () => dashboard.stats.views.toLocaleString(),
    '#like-count': () => dashboard.stats.likes.toLocaleString(),
    '#comment-count': () => dashboard.stats.comments.toLocaleString(),
    '#share-count': () => dashboard.stats.shares.toLocaleString(),

    // Engagement rate
    '#engagement-rate': () => {
      const total = dashboard.stats.likes +
                    dashboard.stats.comments +
                    dashboard.stats.shares;
      const rate = (total / dashboard.stats.views) * 100;
      return `${rate.toFixed(1)}%`;
    },

    // Activity
    '#last-login': () => {
      const now = new Date();
      const diff = now - dashboard.activity.lastLogin;
      const minutes = Math.floor(diff / 60000);
      return minutes < 1 ? 'Just now' : `${minutes} minutes ago`;
    },
    '#online-status': () => {
      return dashboard.activity.isOnline ? '🟢 Online' : '⚪ Offline';
    },

    // Notifications
    '#notification-count': () => dashboard.notifications.count,
    '#notification-badge': () => {
      return dashboard.notifications.count > 0
        ? dashboard.notifications.count
        : '';
    },
    '#notification-list': () => {
      return dashboard.notifications.messages
        .map(msg => `<li>${msg}</li>`)
        .join('');
    }
  }
);

// Simulate real-time updates
setInterval(() => {
  dashboard.stats.views += Math.floor(Math.random() * 10);
  dashboard.stats.likes += Math.floor(Math.random() * 3);
  // ✨ All calculated metrics update automatically
}, 5000);
```

---

## Common Patterns

### Pattern 1: Conditional Display

```javascript
const app = createState(
  { isLoggedIn: false, userName: '' },
  {
    '#welcome-message': () => {
      return app.isLoggedIn
        ? `Welcome back, ${app.userName}!`
        : 'Please log in';
    },
    '#login-btn': () => app.isLoggedIn ? 'Logout' : 'Login'
  }
);
```

### Pattern 2: Loading States

```javascript
const data = createState(
  {
    isLoading: false,
    items: [],
    error: null
  },
  {
    '#loading-spinner': () => data.isLoading ? 'Loading...' : '',
    '#item-list': () => {
      if (data.error) return `Error: ${data.error}`;
      if (data.isLoading) return 'Loading...';
      return data.items.map(i => `<li>${i}</li>`).join('');
    }
  }
);
```

### Pattern 3: Progress Tracking

```javascript
const upload = createState(
  {
    fileName: 'document.pdf',
    progress: 0,
    total: 100
  },
  {
    '#file-name': () => upload.fileName,
    '#progress-percent': () => `${upload.progress}%`,
    '#progress-bar': () => upload.progress,
    '#status': () => {
      if (upload.progress === 0) return 'Not started';
      if (upload.progress === upload.total) return '✓ Complete';
      return 'Uploading...';
    }
  }
);
```

---

## Key Takeaways

✅ **All-in-one** - Creates reactive state + DOM bindings in one call
✅ **Less boilerplate** - Eliminates separate `bindings()` call
✅ **Declarative** - Define data and where it shows, done
✅ **Automatic sync** - State changes flow to DOM automatically
✅ **Optional bindings** - Can omit bindings if you just want reactive state
✅ **Same power** - Equivalent to `state()` + `bindings()`, just cleaner

---

## What's Next?

Now that you understand `createState()`, explore:

- **`builder()`** - Chainable API for complex state setup
- **`ref()`** - Create reactive primitives for simple values
- **`createCollection()`** - Reactive arrays with built-in methods
- **`computed()`** - Add cached computed properties to state

---

## Summary

`createState()` is the **convenient all-in-one function** for creating reactive state with automatic DOM synchronization. It combines `state()` + `bindings()` into a single call, making your code cleaner and more declarative.

**The magic formula:**
```
createState(data, bindings) = state(data) + bindings(bindings)
──────────────────────────────────────────────────────────────
Clean, one-call state setup with automatic DOM sync
```

Think of it as **ordering a complete pre-configured package** instead of buying and assembling parts separately. Same result, less work, clearer code.
