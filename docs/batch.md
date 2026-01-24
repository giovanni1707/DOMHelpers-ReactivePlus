# `batch()` - Batch Reactive Updates

## Quick Start (30 seconds)

```javascript
// Create state
const app = state({
  count: 0,
  title: 'App',
  status: 'idle'
});

// Effect runs on any property change
effect(() => {
  console.log(`${app.title}: ${app.count} (${app.status})`);
});

// Without batch - effect runs 3 times
app.count = 10;    // Effect runs: "App: 10 (idle)"
app.title = 'New'; // Effect runs: "New: 10 (idle)"
app.status = 'active'; // Effect runs: "New: 10 (active)"

// With batch - effect runs only once
batch(() => {
  app.count = 10;
  app.title = 'New';
  app.status = 'active';
});
// Effect runs once: "New: 10 (active)"
```

**That's it.** Group multiple state changes together so effects run only once after all updates complete.

---

## What is `batch()`?

`batch()` **groups multiple reactive updates together and defers effect execution until all updates complete**. Instead of running effects after each individual change, they run once after the batch.

Think of it as **transaction mode for state updates** — make multiple changes, commit them all at once, and effects react to the final state.

**In practical terms:** Use `batch()` when making several related state changes to avoid redundant effect executions and improve performance.

---

## Syntax

```javascript
// Batch updates
batch(() => {
  // Multiple state changes here
  state.prop1 = value1;
  state.prop2 = value2;
  state.prop3 = value3;
});

// Can return value
const result = batch(() => {
  state.count = 10;
  state.total = 100;
  return state.total;
});

console.log(result); // 100
```

**Parameters:**
- `fn` - Function containing state updates to batch

**Returns:**
- The return value of the function

**Important:**
- Effects run **after** the function completes
- Updates are **atomic** — all changes visible together
- Can be **nested** — batches stack properly
- **Automatic** in most update functions like `set()` and `updateAll()`

---

## Why Does This Exist?

### The Problem Without batch()

Multiple state changes trigger effects multiple times:

```javascript
// ❌ Without batch - inefficient effect execution
const cart = state({
  items: [],
  count: 0,
  total: 0
});

// Expensive effect
effect(() => {
  console.log('Updating UI...');
  updateCartDisplay(cart.items, cart.count, cart.total);
  // Expensive DOM updates...
});

// Add item
function addItem(item) {
  cart.items.push(item);     // Effect runs (incomplete state)
  cart.count = cart.count + 1;      // Effect runs (incomplete state)
  cart.total = cart.total + item.price; // Effect runs (complete state)
}

// Effect runs 3 times per addItem!
```

**Problems:**
❌ **Multiple effect runs** - Same effect executes multiple times
❌ **Incomplete state** - Effects see intermediate states
❌ **Performance waste** - Redundant computations and DOM updates
❌ **Visual glitches** - UI flashes between intermediate states
❌ **Inconsistent state** - Effects might see partial updates

### The Solution with `batch()`

```javascript
// ✅ With batch - efficient single effect execution
const cart = state({
  items: [],
  count: 0,
  total: 0
});

// Expensive effect
effect(() => {
  console.log('Updating UI...');
  updateCartDisplay(cart.items, cart.count, cart.total);
});

// Add item with batching
function addItem(item) {
  batch(() => {
    cart.items.push(item);
    cart.count = cart.count + 1;
    cart.total = cart.total + item.price;
  });
}

// Effect runs only once with complete final state!
```

**Benefits:**
✅ **Single effect execution** - Runs once with final state
✅ **Atomic updates** - All changes visible together
✅ **Performance** - No redundant computations
✅ **Smooth UI** - No intermediate visual states
✅ **Consistent state** - Effects always see complete updates

---

## Mental Model: Transaction Mode

Think of `batch()` like **a database transaction**:

**Without batch() (Individual Commits):**
```
┌─────────────────────────────┐
│  Individual Updates         │
│                             │
│  state.a = 1                │
│  → Commit                   │
│  → Effects run              │
│                             │
│  state.b = 2                │
│  → Commit                   │
│  → Effects run              │
│                             │
│  state.c = 3                │
│  → Commit                   │
│  → Effects run              │
│                             │
│  Effects run 3 times!       │
└─────────────────────────────┘
```

**With batch() (Transaction):**
```
┌─────────────────────────────┐
│  Batched Transaction        │
│                             │
│  batch(() => {              │
│    state.a = 1  (pending)   │
│    state.b = 2  (pending)   │
│    state.c = 3  (pending)   │
│  })                         │
│  → Commit all               │
│  → Effects run once         │
│                             │
│  Effects run 1 time!        │
└─────────────────────────────┘
```

`batch()` is **transaction mode** — commit all changes together.

---

## How Does It Work?

`batch()` defers effect execution until completion:

```
Call batch(fn)
    ↓
Increment batch depth
    ↓
Execute function
    ↓
State changes happen:
  state.a = 1  → Queue update
  state.b = 2  → Queue update
  state.c = 3  → Queue update
    ↓
Function completes
    ↓
Decrement batch depth
    ↓
Batch depth = 0?
    ↓
   Yes → Flush queued updates
    ↓
Run all effects once
    ↓
Return function result
```

**Key behaviors:**
- Increments **batch depth** on entry
- Queues effects instead of running them
- Decrements **batch depth** on exit
- Flushes queue when depth reaches **0**
- Supports **nesting** (multiple batch calls)
- **Safe for errors** - cleanup in finally block

---

## Basic Usage

### Example 1: Form Updates

```javascript
const form = state({
  firstName: '',
  lastName: '',
  email: '',
  isValid: false
});

effect(() => {
  validateForm(form);
  updateFormUI(form);
});

// Update multiple fields at once
function updateUser(userData) {
  batch(() => {
    form.firstName = userData.firstName;
    form.lastName = userData.lastName;
    form.email = userData.email;
    form.isValid = validateAll(userData);
  });
  // Effect runs once with all fields updated
}
```

---

### Example 2: Shopping Cart

```javascript
const cart = state({
  items: [],
  subtotal: 0,
  tax: 0,
  total: 0
});

effect(() => {
  console.log('Cart updated:', cart.total);
  renderCart(cart);
});

// Add item with all calculations
function addToCart(item) {
  batch(() => {
    cart.items.push(item);
    cart.subtotal = calculateSubtotal(cart.items);
    cart.tax = cart.subtotal * 0.08;
    cart.total = cart.subtotal + cart.tax;
  });
  // UI updates once with final totals
}
```

---

### Example 3: Animation Frame

```javascript
const animation = state({
  x: 0,
  y: 0,
  rotation: 0,
  scale: 1
});

effect(() => {
  renderSprite(animation);
});

// Update all animation properties per frame
function animate(deltaTime) {
  batch(() => {
    animation.x += velocity.x * deltaTime;
    animation.y += velocity.y * deltaTime;
    animation.rotation += rotationSpeed * deltaTime;
    animation.scale = 1 + Math.sin(Date.now() / 1000) * 0.1;
  });
  // Renders once per frame with all properties synced
}
```

---

### Example 4: Data Sync

```javascript
const app = state({
  users: [],
  posts: [],
  comments: [],
  loading: false,
  lastSync: null
});

effect(() => {
  console.log('Data synced:', app.lastSync);
  updateDashboard(app);
});

// Sync all data from server
async function syncData() {
  const [users, posts, comments] = await Promise.all([
    fetchUsers(),
    fetchPosts(),
    fetchComments()
  ]);

  batch(() => {
    app.users = users;
    app.posts = posts;
    app.comments = comments;
    app.loading = false;
    app.lastSync = new Date();
  });
  // Dashboard updates once with all data
}
```

---

### Example 5: Game State

```javascript
const game = state({
  score: 0,
  level: 1,
  health: 100,
  powerUps: []
});

effect(() => {
  updateGameUI(game);
  saveProgress(game);
});

// Level up with multiple changes
function levelUp() {
  batch(() => {
    game.level = game.level + 1;
    game.health = 100;  // Reset health
    game.score = game.score + (game.level * 1000);
    game.powerUps = [];  // Clear power-ups
  });
  // UI and save happen once
}
```

---

### Example 6: Bulk Import

```javascript
const data = state({
  records: [],
  imported: 0,
  failed: 0,
  status: 'idle'
});

effect(() => {
  updateImportStatus(data);
});

// Import many records
function importRecords(csvData) {
  const records = parseCSV(csvData);

  batch(() => {
    data.status = 'importing';
    data.records = [];
    data.imported = 0;
    data.failed = 0;
  });

  // Process records
  batch(() => {
    records.forEach(record => {
      try {
        data.records.push(processRecord(record));
        data.imported++;
      } catch (e) {
        data.failed++;
      }
    });
    data.status = 'complete';
  });
  // Status updates only twice (start and end)
}
```

---

### Example 7: Theme Switch

```javascript
const theme = state({
  mode: 'light',
  primaryColor: '#007bff',
  backgroundColor: '#ffffff',
  textColor: '#000000',
  fontSize: 16
});

effect(() => {
  applyTheme(theme);
});

// Switch theme with all related changes
function switchTheme(newTheme) {
  const colors = newTheme === 'dark'
    ? { bg: '#1a1a1a', text: '#ffffff', primary: '#4dabf7' }
    : { bg: '#ffffff', text: '#000000', primary: '#007bff' };

  batch(() => {
    theme.mode = newTheme;
    theme.backgroundColor = colors.bg;
    theme.textColor = colors.text;
    theme.primaryColor = colors.primary;
  });
  // Theme applies once with all colors matching
}
```

---

### Example 8: Filter and Sort

```javascript
const products = state({
  items: [],
  category: 'all',
  sortBy: 'name',
  sortOrder: 'asc',
  filtered: []
});

effect(() => {
  renderProductList(products.filtered);
});

// Update filters and sorting together
function applyFilters(category, sortBy, sortOrder) {
  batch(() => {
    products.category = category;
    products.sortBy = sortBy;
    products.sortOrder = sortOrder;
    products.filtered = filterAndSort(
      products.items,
      category,
      sortBy,
      sortOrder
    );
  });
  // List renders once with final filtered/sorted data
}
```

---

### Example 9: Undo/Redo

```javascript
const editor = state({
  content: '',
  cursorPosition: 0,
  selection: null,
  canUndo: false,
  canRedo: false
});

effect(() => {
  updateEditorUI(editor);
});

// Undo operation
function undo() {
  const previous = history.getPrevious();

  batch(() => {
    editor.content = previous.content;
    editor.cursorPosition = previous.cursorPosition;
    editor.selection = previous.selection;
    editor.canUndo = history.canUndo();
    editor.canRedo = history.canRedo();
  });
  // Editor updates once with all state restored
}
```

---

### Example 10: Validation

```javascript
const form = state({
  email: '',
  password: '',
  confirmPassword: '',
  errors: {},
  touched: {},
  isValid: false
});

effect(() => {
  updateFormErrors(form);
});

// Validate all fields
function validateForm() {
  batch(() => {
    const errors = {};

    if (!form.email.includes('@')) {
      errors.email = 'Invalid email';
    }
    if (form.password.length < 8) {
      errors.password = 'Too short';
    }
    if (form.password !== form.confirmPassword) {
      errors.confirmPassword = 'Passwords do not match';
    }

    form.errors = errors;
    form.isValid = Object.keys(errors).length === 0;
  });
  // Errors display once after full validation
}
```

---

## Advanced Usage: Nested Batches

```javascript
const app = state({
  users: [],
  posts: [],
  stats: {}
});

effect(() => {
  console.log('App updated');
  renderApp(app);
});

function updateData() {
  batch(() => {
    // Outer batch
    app.users = fetchUsers();

    batch(() => {
      // Inner batch (nested)
      app.posts = fetchPosts();
      app.stats.posts = app.posts.length;
    });

    app.stats.users = app.users.length;
  });
  // Effect runs once after all batches complete
}
```

---

## Common Patterns

### Pattern 1: Related State Updates

```javascript
batch(() => {
  state.field1 = value1;
  state.field2 = value2;
  state.field3 = value3;
});
```

### Pattern 2: Calculated Values

```javascript
batch(() => {
  state.items = newItems;
  state.count = newItems.length;
  state.total = calculateTotal(newItems);
});
```

### Pattern 3: Reset State

```javascript
batch(() => {
  state.data = null;
  state.loading = false;
  state.error = null;
  state.status = 'idle';
});
```

### Pattern 4: Bulk Updates

```javascript
batch(() => {
  items.forEach(item => {
    state.items.push(item);
    state.count++;
  });
});
```

---

## When to Use batch()

**✅ Use batch() when:**
- Making multiple related state changes
- Updating calculated/derived values
- Performance matters (avoiding redundant effects)
- Preventing UI flickers from intermediate states
- Bulk operations on state

**❌ Don't need batch() when:**
- Using `set()` or `updateAll()` (already batched)
- Making a single state change
- Effects should run after each change
- State changes are unrelated

---

## Key Takeaways

✅ **Single execution** - Effects run once after all updates
✅ **Atomic updates** - All changes visible together
✅ **Performance** - Eliminates redundant effect runs
✅ **Smooth UI** - No intermediate visual states
✅ **Nestable** - Batches can be nested safely
✅ **Returns value** - Can return result from batch function

---

## What's Next?

- **`pause()`** - Pause reactivity completely
- **`resume()`** - Resume reactivity with optional flush
- **`untrack()`** - Run code without tracking dependencies

---

## Summary

`batch()` **groups multiple reactive updates together and defers effect execution** until all updates complete. Effects run once with the final state instead of after each individual change.

**The magic formula:**
```
batch(() => {
  state.a = 1;
  state.b = 2;
  state.c = 3;
}) =
  All updates, one effect run
────────────────────────────
Transaction mode for reactivity
```

Think of it as **transaction mode for state** — make all your changes, then commit them together. Essential for performance optimization and preventing intermediate UI states. Perfect for related updates that should be atomic.
