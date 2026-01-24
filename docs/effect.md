# `effect()` - Automatic Reactive Updates

## Quick Start (30 seconds)

```javascript
const counter = state({ count: 0 });

// This function automatically re-runs when counter.count changes
effect(() => {
  Elements.display.textContent = counter.count;
});

counter.count++; // ✨ Effect runs, DOM updates automatically
counter.count++; // ✨ Effect runs again
```

**That's it.** Write a function once, it runs whenever the data it uses changes. No event listeners, no manual calls, no framework.

---

## What is `effect()`?

`effect()` is **the bridge between reactive state and side effects**. It automatically runs a function whenever any reactive state used inside that function changes.

Think of it as **setting up a motion sensor**. You define what should happen (the function), and the system automatically detects when to trigger it (when dependencies change).

**In practical terms:** You write a function that reads from reactive state and does something (like updating the DOM, making API calls, or logging). The system watches which state properties your function reads, and automatically re-runs your function whenever those specific properties change.

---

## Syntax

```javascript
// Create an effect
effect(functionToRun);

// Example
effect(() => {
  console.log('Count is:', counter.count);
});

// With cleanup (advanced)
const dispose = effect(() => {
  // Effect code here
});

// Later: stop the effect
dispose();
```

**Parameters:**
- `functionToRun` - A function that will run immediately and re-run when its dependencies change

**Returns:**
- A `dispose` function that stops the effect when called

---

## Why Does This Exist?

### The Problem Without Effects

Let's build a simple search feature that filters a list:

```javascript
// ❌ The manual way (without effects)
const searchState = {
  query: '',
  results: []
};

function updateUI() {
  // Update search input
  Elements.searchInput.value = searchState.query;

  // Update results display
  Elements.resultsCount.textContent = searchState.results.length;

  // Update results list
  Elements.resultsList.innerHTML = searchState.results
    .map(item => `<li>${item}</li>`)
    .join('');

  // Show/hide empty message
  Elements.emptyMessage.style.display =
    searchState.results.length === 0 ? 'block' : 'none';
}

// Handle search input
Elements.searchInput.addEventListener('input', (e) => {
  searchState.query = e.target.value;

  // Filter results
  searchState.results = allItems.filter(item =>
    item.toLowerCase().includes(searchState.query.toLowerCase())
  );

  updateUI(); // Must remember to call this!
});

// Initial render
updateUI();
```

**What's the Real Issue?**

```
Data Changes
    ↓
❌ NOTHING HAPPENS (data and UI out of sync)
    ↓
Developer must remember to call updateUI()
    ↓
Easy to forget in complex apps
    ↓
Bugs and inconsistencies
```

**Problems:**
❌ **Manual updates everywhere** - Every data change needs a manual `updateUI()` call
❌ **Tight coupling** - UI update logic is mixed with business logic
❌ **Hard to maintain** - Add a new UI element? Update `updateUI()` function
❌ **Easy to forget** - Miss one update call and your UI is stale
❌ **Performance issues** - `updateUI()` updates everything, even if only one thing changed

### The Solution with `effect()`

```javascript
// ✅ The reactive way
const search = state({
  query: '',
  results: []
});

// Declare WHAT should happen, not WHEN
effect(() => {
  Elements.searchInput.value = search.query;
});

effect(() => {
  Elements.resultsCount.textContent = search.results.length;
});

effect(() => {
  Elements.resultsList.innerHTML = search.results
    .map(item => `<li>${item}</li>`)
    .join('');
});

effect(() => {
  Elements.emptyMessage.style.display =
    search.results.length === 0 ? 'block' : 'none';
});

// Handle search - just update the data
Elements.searchInput.addEventListener('input', (e) => {
  search.query = e.target.value;

  search.results = allItems.filter(item =>
    item.toLowerCase().includes(search.query.toLowerCase())
  );
  // ✨ All relevant effects run automatically!
});
```

**What Just Happened?**

```
Data Changes (search.query or search.results)
    ↓
✨ System detects which effects use that data
    ↓
✨ Only those effects re-run automatically
    ↓
✨ UI updates precisely
    ↓
Always in sync, zero manual work
```

**Benefits:**
✅ **Automatic updates** - Change the data, effects run automatically
✅ **Declarative** - Say "when X changes, Y should happen"
✅ **Fine-grained** - Only effects using changed data re-run
✅ **Decoupled** - Business logic and UI updates are separate
✅ **Scalable** - Add 100 effects, no complexity increase

---

## Mental Model: Motion Sensors

Think of `effect()` like a **motion sensor in a smart home**:

**Without Effects (Manual Light Switch):**
```
┌─────────────────────────────┐
│  Manual Updates             │
│                             │
│  You enter room             │
│  ↓                          │
│  Room stays dark            │
│  ↓                          │
│  You manually flip switch   │
│  ↓                          │
│  Light turns on             │
└─────────────────────────────┘
You must remember every time!
```

**With Effects (Motion Sensor):**
```
┌─────────────────────────────┐
│  Automatic Effects          │
│                             │
│  You enter room             │
│  ↓                          │
│  ✨ Sensor detects motion   │
│  ↓                          │
│  ✨ Light turns on          │
│  ↓                          │
│  You leave room             │
│  ↓                          │
│  ✨ Light turns off         │
└─────────────────────────────┘
System responds automatically!
```

Effects are like motion sensors for your data. They watch for changes and respond automatically.

---

## How Does It Work?

Effects use **dependency tracking** to know when to re-run:

```
Step 1: Effect Runs First Time
┌────────────────────────────┐
│ effect(() => {             │
│   console.log(state.count) │ ← Reads state.count
│ })                         │
└────────────────────────────┘
         ↓
System Records: "This effect depends on state.count"
         ↓
Effect is linked to state.count


Step 2: State Changes
┌────────────────────────────┐
│ state.count++              │
└────────────────────────────┘
         ↓
State notifies: "count changed!"
         ↓
System checks: "Which effects depend on count?"
         ↓
Finds our effect


Step 3: Effect Re-runs
┌────────────────────────────┐
│ effect runs again          │
│ console.log(state.count)   │ ← Logs new value
└────────────────────────────┘
```

**Detailed Flow:**

1️⃣ **Effect runs immediately:**
```javascript
effect(() => {
  Elements.display.textContent = counter.count; // Reads counter.count
});
```

2️⃣ **During execution, state tracks reads:**
```javascript
// Internally:
// "effect_1 just read counter.count"
// dependencies[counter.count] = [effect_1]
```

3️⃣ **When state changes:**
```javascript
counter.count = 5;
// Internally:
// "counter.count changed, notify [effect_1]"
```

4️⃣ **Effect re-runs automatically:**
```javascript
// Effect runs again
Elements.display.textContent = counter.count; // Now shows 5
```

**Key Insight:** You never tell the effect when to run. It figures that out by watching what data you read inside it.

---

## Basic Usage

### Example 1: Simple DOM Update

```javascript
const message = state({ text: 'Hello' });

// Effect runs immediately and whenever message.text changes
effect(() => {
  Elements.messageDisplay.textContent = message.text;
});

// Later...
message.text = 'Hello, World!'; // ✨ DOM updates automatically
message.text = 'Goodbye!';      // ✨ DOM updates again
```

**What's happening?**
- Effect runs immediately (shows "Hello")
- System remembers: "This effect depends on message.text"
- When message.text changes, effect re-runs
- DOM always shows current message.text value

---

### Example 2: Multiple Dependencies

```javascript
const user = state({
  firstName: 'Alice',
  lastName: 'Johnson'
});

// Effect depends on BOTH properties
effect(() => {
  Elements.fullName.textContent = `${user.firstName} ${user.lastName}`;
});

user.firstName = 'Bob';   // ✨ Effect runs (full name updates)
user.lastName = 'Smith';  // ✨ Effect runs (full name updates)
```

**What's happening?**
- Effect reads both firstName and lastName
- System tracks both dependencies
- Changing either property triggers the effect
- You don't need to specify dependencies manually

---

### Example 3: Conditional Dependencies

```javascript
const app = state({
  showDetails: false,
  userName: 'Alice',
  userEmail: 'alice@example.com'
});

effect(() => {
  if (app.showDetails) {
    Elements.details.textContent = `${app.userName} - ${app.userEmail}`;
    Elements.details.style.display = 'block';
  } else {
    Elements.details.style.display = 'none';
  }
});

// When showDetails is false:
app.userName = 'Bob';   // Effect does NOT run (userName not accessed)

// When showDetails is true:
app.showDetails = true; // Effect runs
app.userName = 'Bob';   // NOW effect runs (userName is accessed)
```

**Key Insight:**
Effects only track properties **actually read** during execution. This is called **dynamic dependency tracking**.

---

### Example 4: Working with Collections

```javascript
const todos = state({
  items: ['Task 1', 'Task 2', 'Task 3']
});

// Update todo list when items change
effect(() => {
  Elements.todoList.innerHTML = todos.items
    .map((item, index) => `
      <li data-index="${index}">${item}</li>
    `)
    .join('');
});

// Update count badge
effect(() => {
  Elements.todoCount.textContent = todos.items.length;
});

// Any change triggers effects
todos.items.push('Task 4');       // ✨ Both effects run
todos.items[0] = 'Updated Task';  // ✨ Both effects run
todos.items.splice(1, 1);         // ✨ Both effects run
```

---

### Example 5: Effects with DOM Helpers Shortcuts

```javascript
const theme = state({
  mode: 'light',
  fontSize: 16,
  spacing: 'comfortable'
});

// Update theme using DOM Helpers
effect(() => {
  document.body.className = `theme-${theme.mode}`;
  document.body.style.fontSize = `${theme.fontSize}px`;
  document.body.style.setProperty('--spacing',
    theme.spacing === 'comfortable' ? '1.5rem' : '1rem'
  );
});

// Update all theme controls
effect(() => {
  Collections.ClassName.themeOption.forEach(option => {
    option.classList.toggle('active',
      option.dataset.theme === theme.mode
    );
  });
});

// Change theme - both effects run
theme.mode = 'dark';
```

---

### Example 6: Show/Hide Based on State

```javascript
const auth = state({
  isLoggedIn: false,
  user: null
});

// Show/hide login form
effect(() => {
  Elements.loginForm.style.display = auth.isLoggedIn ? 'none' : 'block';
});

// Show/hide dashboard
effect(() => {
  Elements.dashboard.style.display = auth.isLoggedIn ? 'block' : 'none';
});

// Update user info
effect(() => {
  if (auth.isLoggedIn && auth.user) {
    Elements.userName.textContent = auth.user.name;
    Elements.userAvatar.src = auth.user.avatar;
  }
});

// Simulate login
function login(userData) {
  auth.user = userData;
  auth.isLoggedIn = true;
  // ✨ All three effects run automatically
}
```

---

### Example 7: Derived Calculations

```javascript
const cart = state({
  items: [
    { name: 'Laptop', price: 999, qty: 1 },
    { name: 'Mouse', price: 29, qty: 2 }
  ],
  taxRate: 0.08
});

// Calculate and display subtotal
effect(() => {
  const subtotal = cart.items.reduce((sum, item) =>
    sum + (item.price * item.qty), 0
  );
  Elements.subtotal.textContent = `$${subtotal.toFixed(2)}`;
});

// Calculate and display tax
effect(() => {
  const subtotal = cart.items.reduce((sum, item) =>
    sum + (item.price * item.qty), 0
  );
  const tax = subtotal * cart.taxRate;
  Elements.tax.textContent = `$${tax.toFixed(2)}`;
});

// Calculate and display total
effect(() => {
  const subtotal = cart.items.reduce((sum, item) =>
    sum + (item.price * item.qty), 0
  );
  const total = subtotal * (1 + cart.taxRate);
  Elements.total.textContent = `$${total.toFixed(2)}`;
});

// Update quantity
cart.items[1].qty = 3; // ✨ All three effects recalculate
```

**Note:** This works, but `computed()` provides a better way to share calculated values across effects.

---

### Example 8: Multiple Effects, Fine-Grained Updates

```javascript
const dashboard = state({
  users: 1250,
  revenue: 45780,
  orders: 328,
  period: 'today'
});

// Each stat has its own effect
effect(() => {
  Elements.userCount.textContent = dashboard.users.toLocaleString();
});

effect(() => {
  Elements.revenue.textContent = `$${dashboard.revenue.toLocaleString()}`;
});

effect(() => {
  Elements.orderCount.textContent = dashboard.orders;
});

effect(() => {
  Collections.ClassName.periodBtn.forEach(btn => {
    btn.classList.toggle('active', btn.dataset.period === dashboard.period);
  });
});

// Fine-grained updates
dashboard.users = 1300;    // ✨ Only user count effect runs
dashboard.revenue = 50000; // ✨ Only revenue effect runs
dashboard.period = 'week'; // ✨ Only period button effect runs
```

**Key Insight:**
Each effect only depends on the properties it actually reads. Changing `users` doesn't re-run the `revenue` effect. This is **automatic optimization**.

---

### Example 9: Effects with Event Handlers

```javascript
const counter = state({ count: 0 });

// Auto-update display
effect(() => {
  Elements.counterDisplay.textContent = counter.count;
  Elements.counterBadge.textContent = counter.count;
});

// Auto-update button states
effect(() => {
  Elements.decrementBtn.disabled = counter.count <= 0;
});

effect(() => {
  Elements.resetBtn.style.display = counter.count === 0 ? 'none' : 'inline-block';
});

// Event handlers just update state
Elements.incrementBtn.addEventListener('click', () => {
  counter.count++;
  // ✨ All effects run automatically
});

Elements.decrementBtn.addEventListener('click', () => {
  if (counter.count > 0) {
    counter.count--;
    // ✨ All effects run automatically
  }
});

Elements.resetBtn.addEventListener('click', () => {
  counter.count = 0;
  // ✨ All effects run automatically
});
```

---

### Example 10: Real-World Form Validation

```javascript
const form = state({
  email: '',
  password: '',
  confirmPassword: ''
});

// Email validation effect
effect(() => {
  const isValid = form.email.includes('@') && form.email.includes('.');

  Elements.emailError.update({
    textContent: isValid ? '' : 'Invalid email format',
    style: { display: isValid ? 'none' : 'block' }
  });

  Elements.emailInput.classList.toggle('error', !isValid && form.email.length > 0);
});

// Password match effect
effect(() => {
  const matches = form.password === form.confirmPassword;
  const show = form.confirmPassword.length > 0;

  Elements.passwordError.update({
    textContent: matches ? '' : 'Passwords do not match',
    style: { display: !matches && show ? 'block' : 'none' }
  });
});

// Submit button state effect
effect(() => {
  const emailValid = form.email.includes('@');
  const passwordValid = form.password.length >= 8;
  const passwordsMatch = form.password === form.confirmPassword;

  Elements.submitBtn.disabled = !(emailValid && passwordValid && passwordsMatch);
});

// Input handlers
Elements.emailInput.addEventListener('input', (e) => {
  form.email = e.target.value;
  // ✨ Email validation effect runs
});

Elements.passwordInput.addEventListener('input', (e) => {
  form.password = e.target.value;
  // ✨ Password match + submit effects run
});

Elements.confirmPasswordInput.addEventListener('input', (e) => {
  form.confirmPassword = e.target.value;
  // ✨ Password match + submit effects run
});
```

---

## Deep Dive: How Effects Track Dependencies

### Effects Run Immediately

```javascript
console.log('Before effect');

effect(() => {
  console.log('Effect running');
});

console.log('After effect');

// Output:
// Before effect
// Effect running
// After effect
```

**Why?**
Effects run immediately so they can:
1. Set up initial state (render initial UI)
2. Track their dependencies during first run
3. Ensure UI is in sync from the start

---

### Only What You Read is Tracked

```javascript
const data = state({
  a: 1,
  b: 2,
  c: 3
});

effect(() => {
  console.log(data.a); // Only reads 'a'
});

data.a = 10; // ✨ Effect runs
data.b = 20; // Effect does NOT run (b not read)
data.c = 30; // Effect does NOT run (c not read)
```

**Key Insight:**
The system doesn't track properties you **could** access, only properties you **actually** access during effect execution.

---

### Dependencies Update on Every Run

```javascript
const app = state({
  useA: true,
  a: 1,
  b: 2
});

effect(() => {
  if (app.useA) {
    console.log('A:', app.a);
  } else {
    console.log('B:', app.b);
  }
});

// Initially depends on: useA, a

app.a = 10; // ✨ Effect runs (depends on a)
app.b = 20; // Effect does NOT run (doesn't depend on b yet)

app.useA = false; // ✨ Effect runs

// Now depends on: useA, b (NOT a anymore!)

app.a = 30; // Effect does NOT run
app.b = 40; // ✨ Effect runs (now depends on b)
```

**What's happening?**
Dependencies are **re-tracked on every run**. This allows effects to have conditional logic that changes which properties they depend on.

---

### Effects Automatically Batch

```javascript
const state = state({ a: 1, b: 2 });

effect(() => {
  console.log('Effect ran:', state.a, state.b);
});

// Multiple updates in same tick
state.a = 10;
state.b = 20;
state.a = 15;

// Effect only runs ONCE with final values
// Output: "Effect ran: 15 20"
```

**Why?**
Effects are automatically batched within the same JavaScript execution tick. This prevents unnecessary re-runs and improves performance.

---

### Stopping Effects (Cleanup)

```javascript
// Create effect and get dispose function
const dispose = effect(() => {
  Elements.display.textContent = counter.count;
});

counter.count = 5;  // ✨ Effect runs
counter.count = 10; // ✨ Effect runs

// Stop the effect
dispose();

counter.count = 15; // Effect does NOT run (disposed)
```

**When to use dispose:**
- Cleaning up when a component is removed
- Stopping effects that are no longer needed
- Preventing memory leaks in long-running apps

---

### Effects with Async Operations

```javascript
const search = state({ query: '' });

effect(() => {
  const query = search.query;

  if (query.length > 2) {
    fetch(`/api/search?q=${query}`)
      .then(res => res.json())
      .then(results => {
        Elements.results.innerHTML = results
          .map(r => `<li>${r.title}</li>`)
          .join('');
      });
  }
});
```

**Important:** The effect tracks `search.query` **synchronously** during the initial function run. The async operations inside don't affect dependency tracking.

**For better async handling**, use `asyncEffect()` which provides AbortSignal support and automatic cancellation.

---

## Common Patterns

### Pattern 1: Toggle Classes Based on State

```javascript
const ui = state({ darkMode: false });

effect(() => {
  document.body.classList.toggle('dark-mode', ui.darkMode);
});

// Usage
Elements.themeToggle.addEventListener('click', () => {
  ui.darkMode = !ui.darkMode;
});
```

---

### Pattern 2: Update Multiple Elements

```javascript
const status = state({ message: 'Ready', type: 'info' });

effect(() => {
  Elements.update({
    statusMessage: { textContent: status.message },
    statusIcon: { className: `icon-${status.type}` },
    statusBadge: {
      textContent: status.type.toUpperCase(),
      classList: {
        remove: ['info', 'warning', 'error'],
        add: [status.type]
      }
    }
  });
});
```

---

### Pattern 3: Conditional Rendering

```javascript
const app = state({
  view: 'home', // 'home', 'about', 'settings'
  isLoading: false
});

effect(() => {
  Elements.homeView.style.display = app.view === 'home' ? 'block' : 'none';
  Elements.aboutView.style.display = app.view === 'about' ? 'block' : 'none';
  Elements.settingsView.style.display = app.view === 'settings' ? 'block' : 'none';
});

effect(() => {
  Elements.loadingSpinner.style.display = app.isLoading ? 'block' : 'none';
});
```

---

### Pattern 4: Derived UI States

```javascript
const cart = state({
  items: [],
  discount: 0
});

effect(() => {
  const isEmpty = cart.items.length === 0;

  Elements.emptyMessage.style.display = isEmpty ? 'block' : 'none';
  Elements.cartContents.style.display = isEmpty ? 'none' : 'block';
  Elements.checkoutBtn.disabled = isEmpty;
});

effect(() => {
  Elements.discountBadge.style.display = cart.discount > 0 ? 'inline' : 'none';
  Elements.discountAmount.textContent = `${cart.discount}% OFF`;
});
```

---

### Pattern 5: Synchronize Collections

```javascript
const filters = state({
  category: 'all',
  sortBy: 'name'
});

effect(() => {
  Collections.ClassName.categoryBtn.forEach(btn => {
    btn.classList.toggle('active',
      btn.dataset.category === filters.category
    );
  });
});

effect(() => {
  Collections.ClassName.sortBtn.forEach(btn => {
    btn.classList.toggle('active',
      btn.dataset.sort === filters.sortBy
    );
  });
});
```

---

## Comparing to Plain JavaScript

### Before (Event-Driven Updates):

```javascript
const state = { count: 0 };

function updateDisplay() {
  Elements.display.textContent = state.count;
}

function updateBadge() {
  Elements.badge.textContent = state.count;
}

function updateButton() {
  Elements.resetBtn.style.display = state.count > 0 ? 'block' : 'none';
}

function updateAll() {
  updateDisplay();
  updateBadge();
  updateButton();
}

Elements.incrementBtn.addEventListener('click', () => {
  state.count++;
  updateAll(); // Manual call
});

updateAll(); // Initial render
```

### After (Effect-Driven Updates):

```javascript
const state = state({ count: 0 });

effect(() => {
  Elements.display.textContent = state.count;
});

effect(() => {
  Elements.badge.textContent = state.count;
});

effect(() => {
  Elements.resetBtn.style.display = state.count > 0 ? 'block' : 'none';
});

Elements.incrementBtn.addEventListener('click', () => {
  state.count++;
  // ✨ All effects run automatically
});
```

**What Changed:**
- ❌ Removed `updateDisplay()`, `updateBadge()`, `updateButton()`, `updateAll()`
- ❌ Removed manual `updateAll()` calls
- ❌ Removed initial render call
- ✅ Added effects that run automatically
- ✅ Cleaner separation of concerns
- ✅ Impossible to forget updates

---

## Effects + DOM Helpers = Framework-Like Power

```javascript
// Realistic todo app
const todos = state({
  items: [],
  filter: 'all'
});

// Render todo list
effect(() => {
  const filtered = todos.items.filter(item => {
    if (todos.filter === 'active') return !item.done;
    if (todos.filter === 'completed') return item.done;
    return true;
  });

  Elements.todoList.innerHTML = filtered
    .map((item, index) => `
      <li class="${item.done ? 'done' : ''}">
        <input type="checkbox" ${item.done ? 'checked' : ''}
               data-index="${index}">
        <span>${item.text}</span>
      </li>
    `)
    .join('');
});

// Update counts
effect(() => {
  Elements.totalCount.textContent = todos.items.length;
  Elements.activeCount.textContent =
    todos.items.filter(t => !t.done).length;
});

// Update filter buttons
effect(() => {
  Collections.ClassName.filterBtn.forEach(btn => {
    btn.classList.toggle('active',
      btn.dataset.filter === todos.filter
    );
  });
});

// Add todo
Elements.addTodoForm.addEventListener('submit', (e) => {
  e.preventDefault();
  todos.items.push({
    text: Elements.todoInput.value,
    done: false
  });
  Elements.todoInput.value = '';
});

// Toggle todo
Elements.todoList.addEventListener('change', (e) => {
  if (e.target.type === 'checkbox') {
    const index = parseInt(e.target.dataset.index);
    todos.items[index].done = e.target.checked;
  }
});
```

**This feels like React/Vue, but it's just:**
- `state()` for data
- `effect()` for automatic updates
- DOM Helpers for clean DOM manipulation
- Plain JavaScript events

---

## Key Takeaways

✅ **Effects run automatically** when their dependencies change
✅ **Dependencies are tracked automatically** - no manual specification needed
✅ **Fine-grained updates** - only effects using changed data re-run
✅ **Run immediately** - effects execute on creation for initial setup
✅ **Dynamic tracking** - dependencies can change based on conditional logic
✅ **Works perfectly with DOM Helpers** - declarative UI updates made simple

---

## What's Next?

Now that you understand `effect()`, explore:

- **`computed()`** - Create cached derived values that multiple effects can share
- **`watch()`** - Run callbacks when specific properties change (with old/new values)
- **`bindings()`** - Skip writing effects manually, use declarative bindings instead
- **`batch()`** - Control when effects run for performance optimization

---

## Summary

`effect()` is the **automatic glue** between reactive state and side effects. Write a function once, and it automatically re-runs whenever the reactive data it uses changes.

**The magic formula:**
```
state() = smart data
effect() = automatic reactions
DOM Helpers = clean updates
────────────────────────────
Framework-like experience in plain JavaScript
```

Effects make your UI **declarative**: you describe what the UI should look like for any given state, and the system keeps it in sync automatically.
