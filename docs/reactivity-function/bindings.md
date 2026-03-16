[![Sponsor](https://img.shields.io/badge/Sponsor-💖-pink)](https://github.com/sponsors/giovanni1707)

[![Sponsor](https://img.shields.io/badge/Sponsor-PayPal-blue?logo=paypal)](https://paypal.me/GiovanniSylvain)

# `bindings()` - Declarative DOM Synchronization

## Quick Start (30 seconds)

```javascript
const user = state({
  name: 'Alice',
  status: 'online'
});

// Bind state to DOM - zero boilerplate
bindings({
  '#user-name': () => user.name,
  '#user-status': () => user.status
});

user.name = 'Bob'; // ✨ DOM updates automatically
```

**That's it.** Map state to elements with simple selector-to-function pairs. No manual `effect()` calls, no `.update()` calls, just pure declarative bindings.

---

## What is `bindings()`?

`bindings()` is **the declarative way to sync state to DOM**. Instead of writing manual `effect()` calls for every element update, you define a simple mapping: "this selector gets this value from state."

Think of it as **setting up a dashboard of synchronized displays**. Like a car dashboard where the speedometer automatically shows current speed, fuel gauge shows fuel level, and temperature shows engine temp — all synchronized to the car's sensors without you doing anything.

**In practical terms:** You provide an object where keys are DOM selectors (using DOM Helpers) and values are functions that return what should be displayed. The system automatically creates effects, finds elements, and keeps everything in sync.

---

## Syntax

```javascript
// Basic binding
bindings({
  selector: () => valueFromState
});

// Multiple bindings
bindings({
  '#element1': () => state.value1,
  '#element2': () => state.value2,
  '.class-name': () => state.value3
});

// With DOM Helpers selectors
bindings({
  '#user-name': () => user.name,
  '.status-badge': () => user.status,
  '#counter': () => counter.count
});

// Returns dispose function
const dispose = bindings({
  '#display': () => value
});

// Later: stop all bindings
dispose();
```

**Parameters:**
- `bindingDefinitions` - Object mapping selectors to functions that return values

**Returns:**
- A `dispose` function to stop all bindings

---

## Why Does This Exist?

### The Problem Without Bindings

Let's say you're building a user profile that shows various pieces of user data:

```javascript
// ❌ Vanilla JavaScript - lots of manual updates
const user = {
  name: 'Alice',
  email: 'alice@example.com',
  status: 'online',
  avatar: '👩',
  role: 'Admin',
  lastSeen: '2 minutes ago'
};

// Manual update function for each property
function updateUserDisplay() {
  document.getElementById('user-name').textContent = user.name;
  document.getElementById('user-email').textContent = user.email;
  document.getElementById('user-status').textContent = user.status;
  document.getElementById('user-avatar').textContent = user.avatar;
  document.getElementById('user-role').textContent = user.role;
  document.getElementById('user-last-seen').textContent = user.lastSeen;
}

// Must call this every time user data changes
user.name = 'Bob';
updateUserDisplay(); // Easy to forget!

user.status = 'away';
updateUserDisplay(); // Must remember again!

// Lots of boilerplate and error-prone!
```

**What's the Real Issue?**

```
For Each Element:
    ↓
Write effect(() => { ... })
    ↓
Find element (Elements.xxx)
    ↓
Set textContent or other property
    ↓
Repeat for every element
    ↓
Lots of repetitive code
```

**Problems:**
❌ **Repetitive boilerplate** - Every binding needs a full `effect()` call
❌ **Hard to read** - Intent buried in implementation details
❌ **Easy to miss elements** - No clear overview of what's bound
❌ **Verbose** - 6 effects for 6 simple text updates
❌ **Maintenance burden** - Add/remove bindings = add/remove effect blocks

### The Solution with `bindings()`

```javascript
// ✅ DOM Helpers + Reactive State with bindings() - clean and declarative
const user = state({
  name: 'Alice',
  email: 'alice@example.com',
  status: 'online',
  avatar: '👩',
  role: 'Admin',
  lastSeen: '2 minutes ago'
});

// All bindings in one clear object
bindings({
  '#user-name': () => user.name,
  '#user-email': () => user.email,
  '#user-status': () => user.status,
  '#user-avatar': () => user.avatar,
  '#user-role': () => user.role,
  '#user-last-seen': () => user.lastSeen
});

// That's it! All elements auto-update when state changes
user.name = 'Bob'; // ✨ DOM updates automatically, no manual calls needed!
```

**What Just Happened?**

```
Define Bindings
    ↓
System creates effects automatically
    ↓
Elements found via DOM Helpers
    ↓
State changes flow to DOM
    ↓
Zero boilerplate, pure declaration
```

**Benefits:**
✅ **Declarative** - State what should sync, not how to sync it
✅ **Concise** - One line per binding
✅ **Clear overview** - See all bindings at a glance
✅ **Less code** - 80% less boilerplate than manual effects
✅ **Automatic cleanup** - Dispose all bindings with one call

---

## Mental Model: Smart Dashboard

Think of `bindings()` like a **car dashboard** or **smart home control panel**:

**Without Bindings (Manual Gauges):**
```
┌─────────────────────────────┐
│  Manual Updates             │
│                             │
│  Speed changes              │
│  ↓                          │
│  You manually update gauge  │
│  ↓                          │
│  Fuel changes               │
│  ↓                          │
│  You manually update gauge  │
│  ↓                          │
│  Temperature changes        │
│  ↓                          │
│  You manually update gauge  │
└─────────────────────────────┘
Tedious and error-prone!
```

**With Bindings (Smart Dashboard):**
```
┌─────────────────────────────┐
│  Automatic Sync             │
│                             │
│  Speedometer → Current Speed│
│  Fuel Gauge → Fuel Level    │
│  Temp Gauge → Engine Temp   │
│                             │
│  All synced automatically   │
│  Just drive, dashboard      │
│  updates itself!            │
└─────────────────────────────┘
```

You define **what displays what**, the system handles **keeping it in sync**.

---

## How Does It Work?

Bindings are syntactic sugar over effects with DOM Helpers:

```
Your Bindings Definition
         ↓
bindings({
  '#display': () => state.value
})
         ↓
Internally Creates:
         ↓
effect(() => {
  const element = querySelector('#display');
  if (element) {
    element.textContent = state.value;
  }
})
         ↓
Element Auto-Updates When state.value Changes
```

**Step-by-step:**

1️⃣ **You define bindings:**
```javascript
bindings({
  '#counter': () => counter.count
});
```

2️⃣ **System creates an effect for each binding:**
```javascript
// Internally (simplified):
effect(() => {
  const element = querySelector('#counter');
  if (element) {
    element.textContent = counter.count;
  }
});
```

3️⃣ **Effect tracks dependencies automatically:**
```javascript
// Reads counter.count, so effect depends on it
```

4️⃣ **When state changes, effect re-runs:**
```javascript
counter.count = 5;
// Effect runs → finds #counter → updates textContent
```

**Key Insight:**
`bindings()` doesn't do anything magical. It's just a **cleaner, more declarative** way to create effects that update DOM elements.

---

## Basic Usage

### Example 1: Simple Text Binding

```javascript
const greeting = state({ message: 'Hello, World!' });

// Bind state to element
bindings({
  '#greeting': () => greeting.message
});

// Later...
greeting.message = 'Welcome!'; // ✨ DOM updates automatically
```

**What's happening?**
- Creates effect that reads `greeting.message`
- Finds element with ID `greeting`
- Sets `textContent` to current message
- Re-runs whenever `greeting.message` changes

---

### Example 2: Multiple Bindings

```javascript
const stats = state({
  users: 1250,
  posts: 3840,
  comments: 12560
});

bindings({
  '#user-count': () => stats.users.toLocaleString(),
  '#post-count': () => stats.posts.toLocaleString(),
  '#comment-count': () => stats.comments.toLocaleString()
});

// Update stats
stats.users = 1300;    // ✨ Only #user-count updates
stats.posts = 4000;    // ✨ Only #post-count updates
stats.comments = 13000; // ✨ Only #comment-count updates
```

**Key Insight:**
Each binding is independent. Changing one piece of state only updates the relevant element.

---

### Example 3: Computed Values in Bindings

```javascript
const counter = state({ count: 0 });

bindings({
  '#count': () => counter.count,
  '#double': () => counter.count * 2,
  '#square': () => counter.count ** 2,
  '#message': () => counter.count > 10 ? 'High!' : 'Low'
});

counter.count = 5;
// ✨ All four elements update with calculated values
```

**What's happening?**
Bindings can contain any logic. The function runs every time dependencies change, and the result updates the DOM.

---

### Example 4: Using Class Selectors (Collections)

```javascript
const theme = state({ mode: 'light' });

// Bind to all elements with class
bindings({
  '.theme-label': () => `Current theme: ${theme.mode}`
});

theme.mode = 'dark';
// ✨ All elements with class "theme-label" update
```

**What's happening?**
When using a class selector, `bindings()` uses DOM Helpers Collections to update **all matching elements** at once.

---

### Example 5: Conditional Content

```javascript
const auth = state({
  isLoggedIn: false,
  userName: 'Guest'
});

bindings({
  '#welcome-message': () => {
    if (auth.isLoggedIn) {
      return `Welcome back, ${auth.userName}!`;
    } else {
      return 'Please log in';
    }
  }
});

// Login
auth.isLoggedIn = true;
auth.userName = 'Alice';
// ✨ Message changes to "Welcome back, Alice!"
```

---

### Example 6: Combining Multiple State Properties

```javascript
const user = state({
  firstName: 'Alice',
  lastName: 'Johnson'
});

bindings({
  '#full-name': () => `${user.firstName} ${user.lastName}`,
  '#initials': () => `${user.firstName[0]}${user.lastName[0]}`
});

user.firstName = 'Bob';
user.lastName = 'Smith';
// ✨ Both elements update: "Bob Smith" and "BS"
```

---

### Example 7: Number Formatting

```javascript
const product = state({
  price: 29.99,
  quantity: 5,
  tax: 0.08
});

bindings({
  '#price': () => `$${product.price.toFixed(2)}`,
  '#quantity': () => product.quantity,
  '#subtotal': () => `$${(product.price * product.quantity).toFixed(2)}`,
  '#tax': () => `$${(product.price * product.quantity * product.tax).toFixed(2)}`,
  '#total': () => {
    const subtotal = product.price * product.quantity;
    const total = subtotal * (1 + product.tax);
    return `$${total.toFixed(2)}`;
  }
});
```

---

### Example 8: Array Length and Emptiness

```javascript
const todos = state({
  items: ['Task 1', 'Task 2', 'Task 3']
});

bindings({
  '#todo-count': () => todos.items.length,
  '#todo-label': () => {
    const count = todos.items.length;
    return count === 1 ? '1 task' : `${count} tasks`;
  },
  '#empty-message': () => {
    return todos.items.length === 0 ? 'No tasks yet!' : '';
  }
});

todos.items.push('Task 4');
// ✨ Count updates to 4, label to "4 tasks"

todos.items = [];
// ✨ Count to 0, label to "0 tasks", empty message appears
```

---

### Example 9: Date and Time Formatting

```javascript
const event = state({
  name: 'Product Launch',
  date: new Date('2025-06-15'),
  attendees: 45
});

bindings({
  '#event-name': () => event.name,
  '#event-date': () => event.date.toLocaleDateString(),
  '#event-day': () => event.date.toLocaleDateString('en-US', { weekday: 'long' }),
  '#attendee-count': () => `${event.attendees} attendees`,
  '#days-until': () => {
    const days = Math.ceil((event.date - new Date()) / (1000 * 60 * 60 * 24));
    return `${days} days until event`;
  }
});
```

---

### Example 10: Real-World Dashboard

```javascript
const dashboard = state({
  revenue: 125780,
  orders: 342,
  customers: 1456,
  conversionRate: 0.034,
  period: 'Last 30 days'
});

bindings({
  '#revenue': () => `$${dashboard.revenue.toLocaleString()}`,
  '#orders': () => dashboard.orders.toLocaleString(),
  '#customers': () => dashboard.customers.toLocaleString(),
  '#conversion-rate': () => `${(dashboard.conversionRate * 100).toFixed(1)}%`,
  '#avg-order': () => {
    const avg = dashboard.revenue / dashboard.orders;
    return `$${avg.toFixed(2)}`;
  },
  '#period-label': () => dashboard.period,
  '#revenue-per-customer': () => {
    const rpc = dashboard.revenue / dashboard.customers;
    return `$${rpc.toFixed(2)}`;
  }
});

// Update for new period
dashboard.revenue = 98450;
dashboard.orders = 267;
dashboard.customers = 1122;
dashboard.period = 'Last 7 days';
// ✨ All calculated metrics update automatically
```

---

## Advanced Usage: Bindings with DOM Helpers

### Using Elements Namespace

```javascript
const app = state({ title: 'My App' });

// Bind using Elements (for IDs)
bindings({
  '#app-title': () => app.title
});

// Equivalent to:
effect(() => {
  Elements.appTitle.textContent = app.title;
});
```

**Both work!** Use `bindings()` for cleaner declaration, or manual `effect()` when you need more control.

---

### Using Collections for Multiple Elements

```javascript
const notification = state({
  message: 'New update available',
  count: 3
});

bindings({
  '.notification-message': () => notification.message,
  '.notification-badge': () => notification.count
});

// Updates ALL elements with these classes
```

**What's happening?**
- Finds all `.notification-message` elements (via Collections)
- Updates all `.notification-badge` elements
- Perfect for repeated UI elements

---

### Combining with Selector Methods

```javascript
const filter = state({ activeFilter: 'all' });

bindings({
  '#active-filter': () => filter.activeFilter,
  '.filter-count': () => `Showing: ${filter.activeFilter}`
});

// Works with any CSS selector
```

---

## Advanced Bindings Patterns

### Pattern 1: Status Indicators

```javascript
const server = state({
  status: 'online',
  latency: 45,
  uptime: 99.98
});

bindings({
  '#server-status': () => {
    const emoji = {
      online: '🟢',
      offline: '🔴',
      maintenance: '🟡'
    };
    return `${emoji[server.status]} ${server.status.toUpperCase()}`;
  },
  '#latency': () => `${server.latency}ms`,
  '#uptime': () => `${server.uptime}%`
});
```

---

### Pattern 2: Progress Indicators

```javascript
const upload = state({
  fileName: 'document.pdf',
  progress: 0,
  total: 100
});

bindings({
  '#file-name': () => upload.fileName,
  '#progress-percent': () => `${upload.progress}%`,
  '#progress-text': () => `${upload.progress} / ${upload.total} MB`,
  '#progress-status': () => {
    if (upload.progress === 0) return 'Not started';
    if (upload.progress === upload.total) return 'Complete';
    return 'Uploading...';
  }
});

// Simulate upload
let interval = setInterval(() => {
  if (upload.progress < upload.total) {
    upload.progress += 10;
  } else {
    clearInterval(interval);
  }
}, 500);
```

---

### Pattern 3: Live Search Results

```javascript
const search = state({
  query: '',
  results: [],
  isSearching: false
});

bindings({
  '#search-query': () => search.query,
  '#result-count': () => {
    if (search.isSearching) return 'Searching...';
    if (search.query === '') return 'Enter a search term';
    return `${search.results.length} results found`;
  },
  '#no-results': () => {
    const show = !search.isSearching
      && search.query !== ''
      && search.results.length === 0;
    return show ? 'No results found' : '';
  }
});
```

---

### Pattern 4: Multi-Language Support

```javascript
const app = state({
  language: 'en',
  translations: {
    en: { welcome: 'Welcome', logout: 'Logout' },
    es: { welcome: 'Bienvenido', logout: 'Cerrar sesión' },
    fr: { welcome: 'Bienvenue', logout: 'Déconnexion' }
  }
});

bindings({
  '#welcome-text': () => app.translations[app.language].welcome,
  '#logout-text': () => app.translations[app.language].logout
});

app.language = 'es';
// ✨ All text updates to Spanish
```

---

### Pattern 5: Shopping Cart Summary

```javascript
const cart = state({
  items: [
    { name: 'Laptop', price: 999, qty: 1 },
    { name: 'Mouse', price: 29, qty: 2 }
  ],
  shippingCost: 10,
  taxRate: 0.08
});

bindings({
  '#item-count': () => {
    const total = cart.items.reduce((sum, item) => sum + item.qty, 0);
    return total === 1 ? '1 item' : `${total} items`;
  },

  '#subtotal': () => {
    const subtotal = cart.items.reduce((sum, item) =>
      sum + (item.price * item.qty), 0
    );
    return `$${subtotal.toFixed(2)}`;
  },

  '#shipping': () => `$${cart.shippingCost.toFixed(2)}`,

  '#tax': () => {
    const subtotal = cart.items.reduce((sum, item) =>
      sum + (item.price * item.qty), 0
    );
    const tax = subtotal * cart.taxRate;
    return `$${tax.toFixed(2)}`;
  },

  '#total': () => {
    const subtotal = cart.items.reduce((sum, item) =>
      sum + (item.price * item.qty), 0
    );
    const total = subtotal + cart.shippingCost + (subtotal * cart.taxRate);
    return `$${total.toFixed(2)}`;
  }
});
```

**Note:** While this works, using `computed()` would be more efficient for sharing calculated values across bindings.

---

## Bindings vs Manual Effects

### When to Use `bindings()`

✅ **Simple text updates** - Just displaying state values
✅ **Lots of similar bindings** - Syncing many elements to state
✅ **Declarative mapping** - Clear "this shows that" relationships
✅ **Quick prototyping** - Get UI synced fast

```javascript
// Perfect use case for bindings
bindings({
  '#name': () => user.name,
  '#email': () => user.email,
  '#status': () => user.status
});
```

---

### When to Use Manual `effect()`

✅ **Complex DOM manipulation** - Multiple operations per update
✅ **Conditional logic** - Different updates based on state
✅ **Multiple elements** - Updating several elements together
✅ **Performance optimization** - Need fine-grained control

```javascript
// Better as manual effect
effect(() => {
  if (user.isActive) {
    Elements.userCard.classList.add('active');
    Elements.userCard.style.borderColor = 'green';
    Elements.statusDot.className = 'dot online';
    Elements.statusText.textContent = 'Online';
  } else {
    Elements.userCard.classList.remove('active');
    Elements.userCard.style.borderColor = 'gray';
    Elements.statusDot.className = 'dot offline';
    Elements.statusText.textContent = 'Offline';
  }
});
```

---

## Combining Bindings with Effects

You can mix both approaches in the same app:

```javascript
const app = state({
  user: 'Alice',
  count: 0,
  theme: 'light'
});

// Simple bindings for text
bindings({
  '#user-name': () => app.user,
  '#counter': () => app.count
});

// Complex effect for theme
effect(() => {
  document.body.className = `theme-${app.theme}`;
  document.body.style.setProperty('--primary',
    app.theme === 'dark' ? '#fff' : '#000'
  );
});
```

**Best Practice:** Use `bindings()` for simple cases, `effect()` for complex cases. Mix and match as needed.

---

## Cleanup and Disposal

### Stopping All Bindings

```javascript
// Create bindings and save dispose function
const dispose = bindings({
  '#display1': () => state.value1,
  '#display2': () => state.value2
});

// Later: stop all bindings
dispose();

// State changes no longer update DOM
state.value1 = 'New value'; // No update
```

---

### Use Case: Component Lifecycle

```javascript
function createUserCard(userData) {
  const user = state(userData);

  // Create bindings
  const unbind = bindings({
    '#user-name': () => user.name,
    '#user-email': () => user.email
  });

  return {
    state: user,
    destroy() {
      unbind(); // Clean up when component is removed
    }
  };
}

const card = createUserCard({ name: 'Alice', email: 'alice@example.com' });

// Later: destroy component
card.destroy();
```

---

## Bindings with DOM Helpers Integration

### Using Elements Namespace

```javascript
const counter = state({ count: 0 });

// Both work identically
bindings({
  '#counter': () => counter.count
});

// Internally finds Elements.counter
```

---

### Using Collections for Bulk Updates

```javascript
const theme = state({ primary: '#007bff' });

bindings({
  '.theme-color': () => theme.primary
});

// Updates all elements with .theme-color class
// Uses Collections.ClassName.themeColor internally
```

---

### Using Selector for Complex Queries

```javascript
const app = state({ activeSection: 'home' });

bindings({
  '[data-section]': () => app.activeSection
});

// Works with attribute selectors
// Uses Selector.queryAll('[data-section]') internally
```

---

## Common Pitfalls and Solutions

### Pitfall 1: Forgetting to Return a Value

```javascript
// ❌ Wrong - function doesn't return anything
bindings({
  '#display': () => {
    console.log(state.value); // Just logging
  }
});

// ✅ Correct - return the value
bindings({
  '#display': () => {
    console.log(state.value);
    return state.value; // Return it
  }
});

// ✅ Better - keep it simple
bindings({
  '#display': () => state.value
});
```

---

### Pitfall 2: Binding to Non-Existent Elements

```javascript
// Element doesn't exist yet
bindings({
  '#future-element': () => state.value
});
// Binding silently fails

// ✅ Solution: Create bindings after DOM is ready
document.addEventListener('DOMContentLoaded', () => {
  bindings({
    '#future-element': () => state.value
  });
});
```

---

### Pitfall 3: Heavy Calculations in Bindings

```javascript
// ❌ Recalculates on every render
bindings({
  '#total': () => {
    // Expensive calculation runs on every update
    return items.reduce((sum, item) => sum + calculatePrice(item), 0);
  }
});

// ✅ Use computed() for expensive calculations
const totals = computed(items, {
  total: function() {
    return this.items.reduce((sum, item) => sum + calculatePrice(item), 0);
  }
});

bindings({
  '#total': () => totals.total // Cached value
});
```

---

## Real-World Example: Complete Dashboard

```javascript
const dashboard = state({
  user: {
    name: 'Alice Johnson',
    role: 'Admin',
    avatar: '👩‍💼'
  },
  stats: {
    sales: 125780,
    orders: 342,
    customers: 1456,
    growth: 12.5
  },
  notifications: {
    unread: 5,
    latest: 'New order #3421'
  },
  period: 'Last 30 days',
  theme: 'light'
});

// All dashboard bindings in one place
bindings({
  // User info
  '#user-name': () => dashboard.user.name,
  '#user-role': () => dashboard.user.role,
  '#user-avatar': () => dashboard.user.avatar,

  // Statistics
  '#sales': () => `$${dashboard.stats.sales.toLocaleString()}`,
  '#orders': () => dashboard.stats.orders.toLocaleString(),
  '#customers': () => dashboard.stats.customers.toLocaleString(),
  '#growth': () => `${dashboard.stats.growth}%`,

  // Calculated metrics
  '#avg-order': () => {
    const avg = dashboard.stats.sales / dashboard.stats.orders;
    return `$${avg.toFixed(2)}`;
  },
  '#sales-per-customer': () => {
    const spc = dashboard.stats.sales / dashboard.stats.customers;
    return `$${spc.toFixed(2)}`;
  },

  // Notifications
  '#notification-count': () => dashboard.notifications.unread,
  '#notification-latest': () => dashboard.notifications.latest,
  '#notification-badge': () => {
    return dashboard.notifications.unread > 0
      ? dashboard.notifications.unread
      : '';
  },

  // Period
  '#period-display': () => dashboard.period
});

// Separate effect for complex theme handling
effect(() => {
  document.body.className = `theme-${dashboard.theme}`;
  Elements.themeToggle.textContent = dashboard.theme === 'light' ? '🌙' : '☀️';
});

// Update stats (simulated)
setInterval(() => {
  dashboard.stats.sales += Math.floor(Math.random() * 1000);
  dashboard.stats.orders += Math.floor(Math.random() * 5);
  dashboard.stats.customers += Math.floor(Math.random() * 3);
  dashboard.notifications.unread = Math.floor(Math.random() * 10);
}, 5000);
```

---

## Key Takeaways

✅ **Bindings are declarative** - Map selectors to state values
✅ **Less boilerplate** - One line per binding vs full effect blocks
✅ **Automatic updates** - Change state, DOM updates automatically
✅ **Works with DOM Helpers** - Leverages Elements, Collections, Selector
✅ **Clean and maintainable** - See all bindings at a glance
✅ **Perfect for simple cases** - Text updates, number displays, status messages
✅ **Use effects for complex cases** - Multiple operations, conditional logic

---

## What's Next?

Now that you understand `bindings()`, explore:

- **`computed()`** - Share expensive calculations across multiple bindings
- **`watch()`** - React to specific state changes with callbacks
- **`updateAll()`** - Mix state updates with DOM updates in one call
- **`createState()`** - Create state with bindings in one step

---

## Summary

`bindings()` is the **zero-boilerplate way** to sync state to DOM. Instead of writing manual effects for every element, you declare simple mappings and let the system handle the rest.

**The magic formula:**
```
state() = reactive data
bindings() = declarative sync
DOM Helpers = element access
────────────────────────────
Clean, maintainable UI updates
```

Think of it as **setting up your dashboard**: define what each display shows, and the system keeps everything in sync automatically. No manual updates, no boilerplate, just clean declarations.