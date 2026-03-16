[![Sponsor](https://img.shields.io/badge/Sponsor-💖-pink)](https://github.com/sponsors/giovanni1707)

[![Sponsor](https://img.shields.io/badge/Sponsor-PayPal-blue?logo=paypal)](https://paypal.me/GiovanniSylvain)

# `computed()` - Add Cached Computed Properties

## Quick Start (30 seconds)

```javascript
// Create state
const cart = state({
  items: [
    { name: 'Laptop', price: 999, qty: 1 },
    { name: 'Mouse', price: 29, qty: 2 }
  ],
  taxRate: 0.08
});

// Add computed properties
computed(cart, {
  subtotal: function() {
    return this.items.reduce((sum, item) => sum + (item.price * item.qty), 0);
  },
  tax: function() {
    return this.subtotal * this.taxRate;
  },
  total: function() {
    return this.subtotal + this.tax;
  }
});

console.log(cart.subtotal); // 1057 (calculated once)
console.log(cart.total);    // 1141.56 (uses cached subtotal)

cart.items[0].qty = 2;
console.log(cart.total);    // Recalculated automatically!
```

**That's it.** Add computed properties to any state object. They automatically recalculate when dependencies change and cache results for performance.

---

## What is `computed()`?

`computed()` **adds cached computed properties to existing state objects**. Computed properties are values derived from other state properties that automatically update and cache their results.

Think of it as **spreadsheet formulas** — you define a calculation once, it automatically recalculates when inputs change, and the result is cached until needed again.

**In practical terms:** Instead of manually calculating derived values every time you need them, you define them once as computed properties and they stay in sync automatically.

---

## Syntax

```javascript
// Add computed properties to state
computed(stateObject, computedDefs);

// Computed definitions
const computedDefs = {
  propertyName: function() {
    return /* calculated value using this.property */;
  }
};

// Example
const user = state({
  firstName: 'Alice',
  lastName: 'Johnson'
});

computed(user, {
  fullName: function() {
    return `${this.firstName} ${this.lastName}`;
  },
  initials: function() {
    return `${this.firstName[0]}${this.lastName[0]}`;
  }
});

console.log(user.fullName); // "Alice Johnson"
user.firstName = 'Bob';
console.log(user.fullName); // "Bob Johnson" (auto-updated!)
```

**Parameters:**
- `stateObject` - Reactive state object to add computed properties to
- `computedDefs` - Object with computed property functions

**Returns:**
- The same state object (for chaining)

**Important:** Computed functions use `this` to access state properties.

---

## Why Does This Exist?

### The Problem Without computed()

Calculating derived values manually is repetitive and error-prone:

```javascript
// ❌ Vanilla JavaScript - manual calculations everywhere
const cartData = {
  items: [
    { name: 'Laptop', price: 999, qty: 1 },
    { name: 'Mouse', price: 29, qty: 2 }
  ],
  taxRate: 0.08
};

// Must calculate manually every time
function getSubtotal() {
  return cartData.items.reduce((sum, item) => sum + (item.price * item.qty), 0);
}

function getTax() {
  return getSubtotal() * cartData.taxRate; // Recalculates subtotal!
}

function getTotal() {
  return getSubtotal() + getTax(); // Recalculates subtotal AND tax!
}

// Manual DOM updates
function updateDisplay() {
  document.getElementById('subtotal').textContent = `$${getSubtotal().toFixed(2)}`;
  document.getElementById('tax').textContent = `$${getTax().toFixed(2)}`;
  document.getElementById('total').textContent = `$${getTotal().toFixed(2)}`;
}

// Add item
cartData.items.push({ name: 'Keyboard', price: 79, qty: 1 });
updateDisplay(); // Must call manually, recalculates everything!

// Change quantity
cartData.items[0].qty = 2;
updateDisplay(); // Must call manually, recalculates everything again!
```

**Problems:**
❌ **Repeated calculations** - Same values calculated multiple times
❌ **No caching** - Every access recalculates from scratch
❌ **Manual updates** - Must call `updateDisplay()` after every change
❌ **Function calls** - `getSubtotal()` instead of clean property access
❌ **Hard to maintain** - Scattered calculation logic

### The Solution with `computed()`

```javascript
// ✅ DOM Helpers + Reactive State with computed() - automatic and cached
const cart = state({
  items: [
    { name: 'Laptop', price: 999, qty: 1 },
    { name: 'Mouse', price: 29, qty: 2 }
  ],
  taxRate: 0.08
});

// Add computed properties - calculated once, cached automatically
computed(cart, {
  subtotal: function() {
    console.log('Calculating subtotal'); // See when it runs
    return this.items.reduce((sum, item) => sum + (item.price * item.qty), 0);
  },
  tax: function() {
    console.log('Calculating tax');
    return this.subtotal * this.taxRate; // Uses cached subtotal
  },
  total: function() {
    console.log('Calculating total');
    return this.subtotal + this.tax; // Uses cached values
  }
});

// Auto-update DOM using bulk updates
effect(() => {
  Elements.update({
    subtotal: { textContent: `$${cart.subtotal.toFixed(2)}` },
    tax: { textContent: `$${cart.tax.toFixed(2)}` },
    total: { textContent: `$${cart.total.toFixed(2)}` }
  });
});

// Add item
cart.items.push({ name: 'Keyboard', price: 79, qty: 1 });
// ✨ Computed values recalculate once, DOM updates automatically!

// Change quantity
cart.items[0].qty = 2;
// ✨ Only affected computeds recalculate, DOM updates automatically!
```

**Benefits:**
✅ **Automatic recalculation** - Updates when dependencies change
✅ **Cached results** - Only recalculates when needed
✅ **Clean property access** - `cart.total` instead of `getTotal()`
✅ **Automatic DOM sync** - Effects handle updates
✅ **Efficient** - No redundant calculations

---

## Mental Model: Spreadsheet Formulas

Think of `computed()` like **formulas in a spreadsheet**:

**Without computed() (Manual Calculations):**
```
┌─────────────────────────────┐
│  Manual Calculations        │
│                             │
│  A1: 100  (price)           │
│  A2: 2    (qty)             │
│                             │
│  Want total?                │
│  → Calculate: A1 * A2       │
│  → Get: 200                 │
│                             │
│  Want again?                │
│  → Calculate: A1 * A2       │
│  → Get: 200                 │
│                             │
│  Recalculate every time!    │
└─────────────────────────────┘
```

**With computed() (Spreadsheet Formula):**
```
┌─────────────────────────────┐
│  Spreadsheet Formula        │
│                             │
│  A1: 100  (price)           │
│  A2: 2    (qty)             │
│  A3: =A1*A2  (total)        │
│                             │
│  Read A3: 200 ← cached      │
│  Read A3: 200 ← cached      │
│  Read A3: 200 ← cached      │
│                             │
│  Change A1 to 150           │
│  A3: =A1*A2  → 300 ← auto!  │
│                             │
│  Calculate once, use many!  │
└─────────────────────────────┘
```

Computed properties are **formulas that update automatically** and cache their results.

---

## How Does It Work?

Computed properties track their dependencies and recalculate only when needed:

```
Define Computed
    ↓
computed(state, {
  total: function() { return this.price * this.qty; }
})
    ↓
First Access (state.total)
    ↓
Run function, track dependencies (price, qty)
    ↓
Cache result
    ↓
Return cached result
    ↓
Dependency Changes (state.price = 200)
    ↓
Mark computed as dirty
    ↓
Next Access (state.total)
    ↓
Recalculate (dependencies changed)
    ↓
Cache new result
    ↓
Return cached result
```

**Key behaviors:**
- Computed functions run **on first access**
- Results are **cached** until dependencies change
- Multiple accesses return **cached value** (no recalculation)
- When dependencies change, computed is marked **dirty**
- Next access **recalculates** the value

---

## Basic Usage

### Example 1: Shopping Cart Totals

```javascript
const cart = state({
  items: [
    { name: 'Laptop', price: 999, qty: 1 },
    { name: 'Mouse', price: 29, qty: 2 }
  ],
  discountPercent: 10,
  taxRate: 0.08
});

computed(cart, {
  subtotal: function() {
    return this.items.reduce((sum, item) => sum + (item.price * item.qty), 0);
  },
  discount: function() {
    return this.subtotal * (this.discountPercent / 100);
  },
  afterDiscount: function() {
    return this.subtotal - this.discount;
  },
  tax: function() {
    return this.afterDiscount * this.taxRate;
  },
  total: function() {
    return this.afterDiscount + this.tax;
  }
});

// UI updates using bulk updates
effect(() => {
  Elements.update({
    subtotal: { textContent: `$${cart.subtotal.toFixed(2)}` },
    discount: { textContent: `-$${cart.discount.toFixed(2)}` },
    tax: { textContent: `$${cart.tax.toFixed(2)}` },
    total: { textContent: `$${cart.total.toFixed(2)}` }
  });
});
```

---

### Example 2: User Profile

```javascript
const user = state({
  firstName: 'Alice',
  lastName: 'Johnson',
  email: 'alice@example.com',
  birthYear: 1990
});

computed(user, {
  fullName: function() {
    return `${this.firstName} ${this.lastName}`;
  },
  initials: function() {
    return `${this.firstName[0]}${this.lastName[0]}`;
  },
  age: function() {
    return new Date().getFullYear() - this.birthYear;
  },
  emailDomain: function() {
    return this.email.split('@')[1];
  }
});

// UI updates using bulk updates
effect(() => {
  Elements.update({
    fullName: { textContent: user.fullName },
    initials: { textContent: user.initials },
    age: { textContent: `${user.age} years old` },
    emailDomain: { textContent: `Email domain: ${user.emailDomain}` }
  });
});
```

---

### Example 3: Form Validation

```javascript
const form = state({
  email: '',
  password: '',
  confirmPassword: '',
  agreeToTerms: false
});

computed(form, {
  emailValid: function() {
    return this.email.includes('@') && this.email.includes('.');
  },
  passwordValid: function() {
    return this.password.length >= 8;
  },
  passwordsMatch: function() {
    return this.password === this.confirmPassword;
  },
  isValid: function() {
    return this.emailValid &&
           this.passwordValid &&
           this.passwordsMatch &&
           this.agreeToTerms;
  }
});

// UI updates using bulk updates
effect(() => {
  Elements.update({
    emailError: {
      textContent: form.email && !form.emailValid ? 'Invalid email' : '',
      style: { display: form.email && !form.emailValid ? 'block' : 'none' }
    },
    passwordError: {
      textContent: form.password && !form.passwordValid
        ? 'Password must be at least 8 characters'
        : '',
      style: { display: form.password && !form.passwordValid ? 'block' : 'none' }
    },
    confirmError: {
      textContent: form.confirmPassword && !form.passwordsMatch
        ? 'Passwords do not match'
        : '',
      style: { display: form.confirmPassword && !form.passwordsMatch ? 'block' : 'none' }
    },
    submitBtn: { disabled: !form.isValid }
  });
});
```

---

### Example 4: Todo Statistics

```javascript
const todos = state({
  items: [
    { text: 'Buy milk', done: false, priority: 'high' },
    { text: 'Walk dog', done: true, priority: 'medium' },
    { text: 'Read book', done: false, priority: 'low' }
  ]
});

computed(todos, {
  total: function() {
    return this.items.length;
  },
  completed: function() {
    return this.items.filter(t => t.done).length;
  },
  active: function() {
    return this.items.filter(t => !t.done).length;
  },
  completionRate: function() {
    if (this.total === 0) return 0;
    return (this.completed / this.total) * 100;
  },
  highPriority: function() {
    return this.items.filter(t => !t.done && t.priority === 'high').length;
  }
});

// UI updates using bulk updates
effect(() => {
  Elements.update({
    totalCount: { textContent: `${todos.total} tasks` },
    activeCount: { textContent: `${todos.active} active` },
    completedCount: { textContent: `${todos.completed} completed` },
    progressBar: {
      style: { width: `${todos.completionRate}%` }
    },
    progressText: { textContent: `${todos.completionRate.toFixed(0)}%` },
    urgentBadge: {
      textContent: todos.highPriority,
      style: { display: todos.highPriority > 0 ? 'inline' : 'none' }
    }
  });
});
```

---

### Example 5: Search and Filter

```javascript
const products = state({
  items: [
    { name: 'Laptop', price: 999, category: 'electronics', inStock: true },
    { name: 'Mouse', price: 29, category: 'electronics', inStock: false },
    { name: 'Desk', price: 299, category: 'furniture', inStock: true }
  ],
  searchQuery: '',
  categoryFilter: 'all',
  showInStockOnly: false
});

computed(products, {
  filtered: function() {
    return this.items.filter(item => {
      // Search filter
      const matchesSearch = item.name
        .toLowerCase()
        .includes(this.searchQuery.toLowerCase());

      // Category filter
      const matchesCategory = this.categoryFilter === 'all' ||
        item.category === this.categoryFilter;

      // Stock filter
      const matchesStock = !this.showInStockOnly || item.inStock;

      return matchesSearch && matchesCategory && matchesStock;
    });
  },
  resultCount: function() {
    return this.filtered.length;
  },
  averagePrice: function() {
    if (this.filtered.length === 0) return 0;
    const sum = this.filtered.reduce((acc, item) => acc + item.price, 0);
    return sum / this.filtered.length;
  }
});

// UI updates using bulk updates
effect(() => {
  Elements.update({
    productList: {
      innerHTML: products.filtered
        .map(p => `<li>${p.name} - $${p.price}</li>`)
        .join('')
    },
    resultCount: { textContent: `${products.resultCount} products` },
    avgPrice: { textContent: `Avg: $${products.averagePrice.toFixed(2)}` }
  });
});
```

---

### Example 6: Date and Time Calculations

```javascript
const event = state({
  name: 'Product Launch',
  startDate: new Date('2024-12-01'),
  endDate: new Date('2024-12-05'),
  attendees: 150,
  ticketPrice: 99
});

computed(event, {
  durationDays: function() {
    const diff = this.endDate - this.startDate;
    return Math.ceil(diff / (1000 * 60 * 60 * 24));
  },
  daysUntilStart: function() {
    const diff = this.startDate - new Date();
    return Math.ceil(diff / (1000 * 60 * 60 * 24));
  },
  isUpcoming: function() {
    return this.daysUntilStart > 0;
  },
  totalRevenue: function() {
    return this.attendees * this.ticketPrice;
  },
  revenuePerDay: function() {
    return this.totalRevenue / this.durationDays;
  }
});

// UI updates using bulk updates
effect(() => {
  Elements.update({
    eventName: { textContent: event.name },
    duration: { textContent: `${event.durationDays} days` },
    countdown: {
      textContent: event.isUpcoming
        ? `${event.daysUntilStart} days until event`
        : 'Event has passed'
    },
    revenue: { textContent: `$${event.totalRevenue.toLocaleString()}` },
    revenuePerDay: { textContent: `$${event.revenuePerDay.toFixed(2)}/day` }
  });
});
```

---

### Example 7: Score and Ranking

```javascript
const game = state({
  players: [
    { name: 'Alice', score: 1500, wins: 10, losses: 5 },
    { name: 'Bob', score: 1200, wins: 8, losses: 7 },
    { name: 'Charlie', score: 1800, wins: 12, losses: 3 }
  ]
});

computed(game, {
  sortedPlayers: function() {
    return [...this.players].sort((a, b) => b.score - a.score);
  },
  topPlayer: function() {
    return this.sortedPlayers[0];
  },
  averageScore: function() {
    const sum = this.players.reduce((acc, p) => acc + p.score, 0);
    return sum / this.players.length;
  },
  playersWithStats: function() {
    return this.players.map(player => ({
      ...player,
      winRate: (player.wins / (player.wins + player.losses) * 100).toFixed(1),
      totalGames: player.wins + player.losses
    }));
  }
});

// UI updates using bulk updates
effect(() => {
  Elements.update({
    leaderboard: {
      innerHTML: game.sortedPlayers
        .map((p, i) => `<li>${i + 1}. ${p.name} - ${p.score}</li>`)
        .join('')
    },
    topPlayer: { textContent: `Top: ${game.topPlayer.name}` },
    avgScore: { textContent: `Avg: ${game.averageScore.toFixed(0)}` }
  });
});
```

---

### Example 8: Budget Calculator

```javascript
const budget = state({
  income: 5000,
  expenses: [
    { category: 'Rent', amount: 1500 },
    { category: 'Food', amount: 600 },
    { category: 'Transport', amount: 300 }
  ],
  savingsGoalPercent: 20
});

computed(budget, {
  totalExpenses: function() {
    return this.expenses.reduce((sum, exp) => sum + exp.amount, 0);
  },
  remaining: function() {
    return this.income - this.totalExpenses;
  },
  savingsGoal: function() {
    return this.income * (this.savingsGoalPercent / 100);
  },
  canMeetSavingsGoal: function() {
    return this.remaining >= this.savingsGoal;
  },
  percentSaved: function() {
    return (this.remaining / this.income) * 100;
  },
  expensesByCategory: function() {
    return this.expenses.reduce((acc, exp) => {
      acc[exp.category] = exp.amount;
      return acc;
    }, {});
  }
});

// UI updates using bulk updates
effect(() => {
  Elements.update({
    income: { textContent: `$${budget.income}` },
    expenses: { textContent: `$${budget.totalExpenses}` },
    remaining: {
      textContent: `$${budget.remaining}`,
      style: { color: budget.remaining >= 0 ? 'green' : 'red' }
    },
    savingsGoal: { textContent: `Goal: $${budget.savingsGoal}` },
    savingsStatus: {
      textContent: budget.canMeetSavingsGoal ? '✓ On track' : '✗ Below goal',
      style: { color: budget.canMeetSavingsGoal ? 'green' : 'orange' }
    },
    percentSaved: { textContent: `${budget.percentSaved.toFixed(1)}% saved` }
  });
});
```

---

### Example 9: Inventory Management

```javascript
const inventory = state({
  products: [
    { sku: 'LAP001', name: 'Laptop', stock: 5, reorderPoint: 10, price: 999 },
    { sku: 'MOU001', name: 'Mouse', stock: 25, reorderPoint: 20, price: 29 },
    { sku: 'KEY001', name: 'Keyboard', stock: 8, reorderPoint: 15, price: 79 }
  ]
});

computed(inventory, {
  totalValue: function() {
    return this.products.reduce((sum, p) => sum + (p.stock * p.price), 0);
  },
  lowStockProducts: function() {
    return this.products.filter(p => p.stock < p.reorderPoint);
  },
  lowStockCount: function() {
    return this.lowStockProducts.length;
  },
  wellStockedCount: function() {
    return this.products.filter(p => p.stock >= p.reorderPoint).length;
  },
  averageStockLevel: function() {
    const sum = this.products.reduce((acc, p) => acc + p.stock, 0);
    return sum / this.products.length;
  },
  needsReorder: function() {
    return this.lowStockCount > 0;
  }
});

// UI updates using bulk updates
effect(() => {
  Elements.update({
    totalValue: { textContent: `$${inventory.totalValue.toLocaleString()}` },
    lowStockCount: {
      textContent: `${inventory.lowStockCount} items low`,
      style: { color: inventory.needsReorder ? 'red' : 'green' }
    },
    lowStockList: {
      innerHTML: inventory.lowStockProducts
        .map(p => `<li>${p.name} (${p.stock} left)</li>`)
        .join(''),
      style: { display: inventory.needsReorder ? 'block' : 'none' }
    },
    wellStocked: { textContent: `${inventory.wellStockedCount} well-stocked` },
    avgStock: { textContent: `Avg: ${inventory.averageStockLevel.toFixed(1)} units` }
  });
});
```

---

### Example 10: Temperature Converter

```javascript
const temp = state({
  celsius: 0
});

computed(temp, {
  fahrenheit: function() {
    return (this.celsius * 9/5) + 32;
  },
  kelvin: function() {
    return this.celsius + 273.15;
  },
  description: function() {
    if (this.celsius < 0) return '❄️ Freezing';
    if (this.celsius < 10) return '🥶 Cold';
    if (this.celsius < 20) return '😊 Cool';
    if (this.celsius < 30) return '☀️ Warm';
    return '🔥 Hot';
  }
});

// UI updates using bulk updates
effect(() => {
  Elements.update({
    celsius: { textContent: `${temp.celsius}°C` },
    fahrenheit: { textContent: `${temp.fahrenheit.toFixed(1)}°F` },
    kelvin: { textContent: `${temp.kelvin.toFixed(2)}K` },
    description: { textContent: temp.description }
  });
});

// Input using bulk event binding
Elements.update({
  celsiusInput: {
    addEventListener: ['input', (e) => {
      temp.celsius = parseFloat(e.target.value) || 0;
    }]
  }
});
```

---

## Advanced Usage: Chained Computed

Computed properties can depend on other computed properties:

```javascript
const sales = state({
  items: [
    { product: 'A', qty: 10, unitPrice: 50 },
    { product: 'B', qty: 5, unitPrice: 100 }
  ],
  discountPercent: 10,
  taxRate: 0.08
});

computed(sales, {
  // First level: subtotal
  subtotal: function() {
    return this.items.reduce((sum, item) => sum + (item.qty * item.unitPrice), 0);
  },

  // Second level: uses subtotal
  discount: function() {
    return this.subtotal * (this.discountPercent / 100);
  },

  // Second level: uses subtotal
  afterDiscount: function() {
    return this.subtotal - this.discount;
  },

  // Third level: uses afterDiscount
  tax: function() {
    return this.afterDiscount * this.taxRate;
  },

  // Fourth level: uses afterDiscount and tax
  total: function() {
    return this.afterDiscount + this.tax;
  }
});

// Change items
sales.items[0].qty = 20;
// ✨ Entire chain recalculates in correct order
```

---

## Performance: Caching in Action

```javascript
const data = state({ value: 5 });

computed(data, {
  expensive: function() {
    console.log('Computing...'); // See when it runs
    let result = 0;
    for (let i = 0; i < 1000000; i++) {
      result += this.value;
    }
    return result;
  }
});

// First access
console.log(data.expensive); // Logs "Computing...", then result
// Second access
console.log(data.expensive); // Returns cached result (no log)
// Third access
console.log(data.expensive); // Returns cached result (no log)

// Change dependency
data.value = 10;

// Next access
console.log(data.expensive); // Logs "Computing..." again, recalculates
// Subsequent access
console.log(data.expensive); // Returns cached result
```

---

## Common Patterns

### Pattern 1: Validation Computed

```javascript
computed(form, {
  errors: function() {
    const errors = {};
    if (!this.email.includes('@')) errors.email = 'Invalid email';
    if (this.password.length < 8) errors.password = 'Too short';
    return errors;
  },
  isValid: function() {
    return Object.keys(this.errors).length === 0;
  }
});
```

### Pattern 2: Filtered Lists

```javascript
computed(state, {
  activeTodos: function() {
    return this.todos.filter(t => !t.done);
  },
  completedTodos: function() {
    return this.todos.filter(t => t.done);
  },
  visibleTodos: function() {
    if (this.filter === 'active') return this.activeTodos;
    if (this.filter === 'completed') return this.completedTodos;
    return this.todos;
  }
});
```

### Pattern 3: Aggregations

```javascript
computed(data, {
  sum: function() {
    return this.items.reduce((acc, val) => acc + val, 0);
  },
  average: function() {
    return this.sum / this.items.length;
  },
  min: function() {
    return Math.min(...this.items);
  },
  max: function() {
    return Math.max(...this.items);
  }
});
```

---

## Key Takeaways

✅ **Automatic recalculation** - Updates when dependencies change
✅ **Cached results** - Only recalculates when needed
✅ **Clean syntax** - Access like properties, not functions
✅ **Chainable** - Computed can depend on other computed
✅ **Efficient** - Eliminates redundant calculations
✅ **Reactive** - Works with effects and bindings

---

## What's Next?

- **`watch()`** - React to state changes with callbacks
- **`effect()`** - Reactive side effects
- **`state()`** - Create reactive state

---

## Summary

`computed()` **adds cached computed properties to state objects**. Computed properties automatically recalculate when their dependencies change and cache results for performance.

**The magic formula:**
```
computed(state, {
  prop: function() { return calculation; }
}) =
  Reactive property that auto-updates and caches
──────────────────────────────────────────────
Spreadsheet-like formulas for your state
```

Think of it as **spreadsheet formulas** — define a calculation once, it automatically updates when inputs change, and results are cached for efficiency. Perfect for derived values, validations, filtered lists, and any calculation based on reactive state.