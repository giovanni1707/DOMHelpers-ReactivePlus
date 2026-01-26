# `proxy.set(key, value, options)` - Set Value in Reactive Storage

**Quick Start (30 seconds)**
```javascript
// Create reactive storage
const storage = ReactiveUtils.reactiveStorage('localStorage', 'myApp');

// Set a value (triggers reactivity)
storage.set('theme', 'dark');
storage.set('user', { name: 'John', age: 30 });

// Reactive effect automatically responds
ReactiveUtils.effect(() => {
  const theme = storage.get('theme');
  console.log('Theme changed:', theme);
});

// Set with expiration (expires in 1 hour)
storage.set('tempData', { value: 123 }, { expires: 3600 });
```

---

## **What is `proxy.set(key, value, options)`?**

`proxy.set()` is a **method on the reactive storage proxy** returned by `reactiveStorage()` that stores a value in browser storage and triggers reactive updates in any effects or computed properties tracking that storage.

**Key characteristics:**
- **Sets Value**: Stores value in localStorage or sessionStorage
- **Triggers Reactivity**: Notifies all tracking effects
- **JSON Serialization**: Automatically converts to JSON
- **Expiration Support**: Optional expiration time
- **Cross-Tab**: Changes trigger storage events
- **Returns Boolean**: true on success, false on failure
- **Safe**: Handles quota exceeded errors

---

## **Syntax**

```javascript
proxy.set(key, value, options)
```

### **Parameters**
- **`key`** (string): Storage key (without namespace prefix)
- **`value`** (any): Value to store (must be JSON-serializable)
- **`options`** (object, optional): Configuration options
  - **`expires`** (number): Expiration time in seconds

### **Returns**
- **Type**: `boolean`
- **`true`**: Successfully stored
- **`false`**: Storage failed (quota exceeded, etc.)

---

## **How it works**

```javascript
// When you call proxy.set()
const storage = ReactiveUtils.reactiveStorage('localStorage', 'app');
storage.set('user', { name: 'John' });

// Internally:
// 1. Wraps value with metadata
const data = {
  value: { name: 'John' },
  timestamp: Date.now()
};

// 2. Serializes to JSON
const serialized = JSON.stringify(data);

// 3. Stores with full key
localStorage.setItem('app:user', serialized);

// 4. Increments reactive version
reactiveState._version++;
reactiveState._keys = new Set(store.keys());

// 5. All tracking effects re-run
```

**What happens:**
1. Validates and serializes value to JSON
2. Wraps value with timestamp and optional expiration
3. Stores in browser storage with namespaced key
4. Increments internal reactive version number
5. Updates internal keys Set
6. Triggers all effects tracking this storage
7. Fires storage event (cross-tab sync)

---

## **Examples**

### **Example 1: Basic Set**
```javascript
const storage = ReactiveUtils.reactiveStorage('localStorage');

// Set simple values
storage.set('count', 42);
storage.set('name', 'Alice');
storage.set('active', true);

// Set objects
storage.set('user', {
  id: 123,
  name: 'John',
  email: 'john@example.com'
});

// Set arrays
storage.set('items', [1, 2, 3, 4, 5]);
```

### **Example 2: Reactive Updates**
```javascript
const prefs = ReactiveUtils.reactiveStorage('localStorage', 'prefs');

// Set initial value
prefs.set('theme', 'light');

// Effect tracks storage
ReactiveUtils.effect(() => {
  const theme = prefs.get('theme');
  document.body.className = theme + '-theme';
});

// Update triggers effect
prefs.set('theme', 'dark');
// Document body class updates automatically to 'dark-theme'
```

### **Example 3: Set with Expiration**
```javascript
const cache = ReactiveUtils.reactiveStorage('localStorage', 'cache');

// Expires in 1 hour (3600 seconds)
cache.set('userData', {
  name: 'Jane',
  lastFetch: Date.now()
}, { expires: 3600 });

// Expires in 5 minutes
cache.set('tempToken', 'abc123', { expires: 300 });

// After expiration, get() returns null
setTimeout(() => {
  const token = cache.get('tempToken'); // null
  console.log('Token expired');
}, 301000);
```

### **Example 4: Update Counter**
```javascript
const storage = ReactiveUtils.reactiveStorage('localStorage', 'app');

// Initialize counter
if (!storage.has('clicks')) {
  storage.set('clicks', 0);
}

// Reactive display
ReactiveUtils.effect(() => {
  const clicks = storage.get('clicks');
  querySelector('#counter').textContent = clicks;
});

// Increment counter
querySelector('#btn').addEventListener('click', () => {
  const current = storage.get('clicks') || 0;
  storage.set('clicks', current + 1);
});
```

### **Example 5: Shopping Cart**
```javascript
const cart = ReactiveUtils.reactiveStorage('localStorage', 'cart');

// Initialize empty cart
cart.set('items', []);
cart.set('total', 0);

function addToCart(product) {
  const items = cart.get('items') || [];
  items.push(product);

  const total = items.reduce((sum, item) => sum + item.price, 0);

  cart.set('items', items);
  cart.set('total', total);
}

// Reactive cart display
ReactiveUtils.effect(() => {
  const items = cart.get('items') || [];
  const total = cart.get('total') || 0;

  querySelector('#cartCount').textContent = items.length;
  querySelector('#cartTotal').textContent = `$${total.toFixed(2)}`;
});

// Add items
addToCart({ name: 'Product A', price: 29.99 });
addToCart({ name: 'Product B', price: 19.99 });
```

### **Example 6: User Preferences**
```javascript
const prefs = ReactiveUtils.reactiveStorage('localStorage', 'preferences');

const defaultPrefs = {
  theme: 'light',
  fontSize: 14,
  notifications: true,
  language: 'en'
};

// Initialize with defaults
Object.entries(defaultPrefs).forEach(([key, value]) => {
  if (!prefs.has(key)) {
    prefs.set(key, value);
  }
});

// Settings panel
ReactiveUtils.effect(() => {
  const theme = prefs.get('theme');
  const fontSize = prefs.get('fontSize');

  document.body.className = `${theme}-theme`;
  document.body.style.fontSize = `${fontSize}px`;
});

// Update settings
function updateTheme(newTheme) {
  prefs.set('theme', newTheme);
}

function updateFontSize(size) {
  prefs.set('fontSize', size);
}
```

### **Example 7: Form Auto-Save**
```javascript
const formData = ReactiveUtils.reactiveStorage('sessionStorage', 'formDraft');

// Save form on input
querySelectorAll('input, textarea').forEach(field => {
  field.addEventListener('input', () => {
    const form = {};
    querySelectorAll('input, textarea').forEach(f => {
      form[f.name] = f.value;
    });
    formData.set('draft', form);
  });
});

// Load saved draft
window.addEventListener('load', () => {
  const draft = formData.get('draft');
  if (draft) {
    Object.entries(draft).forEach(([name, value]) => {
      const field = querySelector(`[name="${name}"]`);
      if (field) field.value = value;
    });
  }
});
```

### **Example 8: Cross-Tab Messaging**
```javascript
const messages = ReactiveUtils.reactiveStorage('localStorage', 'messages');

// Tab 1: Listen for messages
ReactiveUtils.effect(() => {
  const msgs = messages.get('inbox') || [];
  console.log('Messages:', msgs);
  displayMessages(msgs);
});

// Tab 2: Send message
function sendMessage(text) {
  const msgs = messages.get('inbox') || [];
  msgs.push({
    text: text,
    timestamp: Date.now(),
    from: 'Tab 2'
  });
  messages.set('inbox', msgs);
}

// Message sent in Tab 2 appears in Tab 1 instantly
```

### **Example 9: Last Visited Pages**
```javascript
const history = ReactiveUtils.reactiveStorage('localStorage', 'history');

function trackPage(url, title) {
  const pages = history.get('visited') || [];

  // Add to history (max 50 items)
  pages.unshift({ url, title, timestamp: Date.now() });
  if (pages.length > 50) {
    pages.length = 50;
  }

  history.set('visited', pages);
}

// Reactive history sidebar
ReactiveUtils.effect(() => {
  const pages = history.get('visited') || [];
  const html = pages.slice(0, 10).map(page => `
    <li><a href="${page.url}">${page.title}</a></li>
  `).join('');
  querySelector('#history').innerHTML = html;
});

// Track page visits
trackPage('/about', 'About Us');
trackPage('/products', 'Products');
```

### **Example 10: Real-Time Sync**
```javascript
const sync = ReactiveUtils.reactiveStorage('localStorage', 'sync');

// Set sync status
sync.set('lastSync', Date.now());
sync.set('pendingChanges', 0);

// Monitor sync status
ReactiveUtils.effect(() => {
  const lastSync = sync.get('lastSync');
  const pending = sync.get('pendingChanges') || 0;

  const indicator = querySelector('#syncStatus');
  if (pending > 0) {
    indicator.textContent = `${pending} pending changes`;
    indicator.className = 'syncing';
  } else {
    const elapsed = Date.now() - lastSync;
    indicator.textContent = `Synced ${Math.floor(elapsed / 1000)}s ago`;
    indicator.className = 'synced';
  }
});

// Trigger sync
function syncData() {
  const pending = sync.get('pendingChanges') || 0;
  sync.set('pendingChanges', pending + 1);

  // Perform sync...

  sync.set('pendingChanges', 0);
  sync.set('lastSync', Date.now());
}
```

---

## **Common Patterns**

### **Pattern 1: Simple Set**
```javascript
storage.set('key', value);
```

### **Pattern 2: Set with Expiration**
```javascript
storage.set('key', value, { expires: 3600 });
```

### **Pattern 3: Update Pattern**
```javascript
const current = storage.get('key') || defaultValue;
storage.set('key', modify(current));
```

### **Pattern 4: Batch Updates**
```javascript
['key1', 'key2', 'key3'].forEach(key => {
  storage.set(key, values[key]);
});
```

### **Pattern 5: Conditional Set**
```javascript
if (!storage.has('key')) {
  storage.set('key', defaultValue);
}
```

---

## **When to Use**

| Scenario | Use proxy.set() |
|----------|-----------------|
| Store user preferences | ✓ Yes |
| Cache API responses | ✓ Yes |
| Save form drafts | ✓ Yes |
| Track user actions | ✓ Yes |
| Cross-tab messaging | ✓ Yes |
| Temporary data | ✓ Yes (with expires) |
| Sensitive data | ⚠ Use sessionStorage |
| Large files | ✗ No (size limits) |

---

## **Options**

### **`expires` (number)**
Expiration time in seconds. After this time, `get()` returns null.

```javascript
// Expires in 1 hour
storage.set('temp', data, { expires: 3600 });

// Expires in 5 minutes
storage.set('token', 'abc', { expires: 300 });

// No expiration (default)
storage.set('permanent', data);
```

---

## **Error Handling**

### **Quota Exceeded**
```javascript
const result = storage.set('largeData', hugeObject);
if (!result) {
  console.error('Storage quota exceeded');
  // Clear old data or notify user
}
```

### **Serialization Errors**
```javascript
// Circular references fail
const obj = {};
obj.self = obj;
storage.set('circular', obj); // Returns false, logs error

// Functions not serializable
storage.set('func', () => {}); // Returns false
```

### **Check Success**
```javascript
const success = storage.set('key', value);
if (success) {
  console.log('Saved successfully');
} else {
  console.error('Save failed');
}
```

---

## **Best Practices**

1. **Check serialization**
   ```javascript
   try {
     JSON.stringify(value);
     storage.set('key', value);
   } catch (e) {
     console.error('Value not serializable');
   }
   ```

2. **Use expiration for temporary data**
   ```javascript
   storage.set('session', data, { expires: 7200 }); // 2 hours
   ```

3. **Namespace your storage**
   ```javascript
   const storage = ReactiveUtils.reactiveStorage('localStorage', 'myApp');
   ```

4. **Don't store sensitive data**
   ```javascript
   // Bad: passwords, tokens in localStorage
   // Good: use sessionStorage or secure cookies
   ```

5. **Check storage availability**
   ```javascript
   if (ReactiveUtils.isStorageAvailable('localStorage')) {
     storage.set('key', value);
   }
   ```

6. **Batch updates efficiently**
   ```javascript
   // Good: batch related updates
   storage.set('user', userData);
   storage.set('prefs', userPrefs);
   storage.set('lastLogin', Date.now());
   ```

---

## **vs. Native localStorage.setItem()**

| Feature | `proxy.set()` | `localStorage.setItem()` |
|---------|---------------|-------------------------|
| Triggers reactivity | ✓ Yes | ✗ No |
| JSON handling | ✓ Automatic | ✗ Manual |
| Expiration | ✓ Built-in | ✗ Manual |
| Namespace | ✓ Automatic | ✗ Manual |
| Error handling | ✓ Returns boolean | ✗ Throws errors |

```javascript
// proxy.set() - reactive, automatic
const storage = ReactiveUtils.reactiveStorage('localStorage');
storage.set('user', { name: 'John' });
// Triggers effects, handles JSON, namespacing

// localStorage.setItem() - manual
localStorage.setItem('user', JSON.stringify({ name: 'John' }));
// No reactivity, manual JSON, no namespace
```

---

## **Key Takeaways**

1. **Stores Value**: Saves data in browser storage
2. **Triggers Reactivity**: Notifies all tracking effects
3. **JSON Automatic**: Handles serialization automatically
4. **Expiration Support**: Optional expiration time
5. **Returns Boolean**: Indicates success/failure
6. **Cross-Tab**: Triggers storage events
7. **Namespaced**: Keys automatically prefixed
8. **Safe**: Handles quota errors gracefully
9. **Metadata**: Stores timestamp automatically
10. **Reactive Updates**: Effects re-run on changes

---

## **Summary**

`proxy.set(key, value, options)` is a method on the reactive storage proxy returned by `reactiveStorage()` that stores a value in browser storage (localStorage or sessionStorage) and triggers reactive updates in any effects or computed properties tracking that storage. When called, it automatically serializes the value to JSON, wraps it with metadata (timestamp and optional expiration), stores it with a namespaced key, and increments an internal reactive version number that triggers all tracking effects to re-run. The method returns true on success or false if storage fails (quota exceeded, serialization error, etc.). Use `proxy.set()` when you need to store data in browser storage and want UI components or other reactive logic to automatically update when that data changes. It's perfect for user preferences, form auto-save, cached API responses, cross-tab messaging, and any scenario where multiple parts of your application need to react to storage changes. Unlike native localStorage.setItem(), proxy.set() provides automatic JSON handling, built-in expiration support, namespace management, and most importantly, reactive updates that keep your UI synchronized with storage changes.

---

## **Advanced: Reactive Bindings with Global Shortcuts**

These examples demonstrate the power of combining reactive storage with **Elements** (ID access), **bindings()** (declarative DOM sync), **ClassName auto-iteration** (no forEach), and **conditions** (reactive rendering).

### **Example: Elements + bindings() - Declarative Sync**
```javascript
const storage = ReactiveUtils.reactiveStorage('localStorage', 'ui');

// Set theme
storage.set('theme', 'dark');
storage.set('fontSize', 16);

// Declarative bindings - automatically sync storage to DOM
bindings({
  // Elements: ID-based access (fastest)
  '#themeDisplay': () => storage.get('theme'),
  '#fontSizeValue': () => storage.get('fontSize') + 'px',

  // Elements with multiple properties
  '#statusBar': {
    textContent: () => `Theme: ${storage.get('theme')}`,
    className: () => `status ${storage.get('theme')}-mode`
  }
});

// Change storage - bindings update automatically
storage.set('theme', 'light');  // #themeDisplay shows "light"
storage.set('fontSize', 18);     // #fontSizeValue shows "18px"
```

**Why this is powerful:**
- **Elements** (via `#id`) = Fast, direct access by ID
- **bindings()** = Declarative, no manual effects
- Storage changes → UI updates automatically
- No manual querySelector or updateAll needed

### **Example: ClassName Auto-Iteration + Array-Based Updates**
```javascript
const storage = ReactiveUtils.reactiveStorage('localStorage', 'items');

// Set array data
storage.set('products', [
  { name: 'Product A', price: 29.99 },
  { name: 'Product B', price: 39.99 },
  { name: 'Product C', price: 49.99 }
]);

ReactiveUtils.effect(() => {
  const products = storage.get('products') || [];

  // Array-based update - NO forEach needed!
  // ClassName automatically distributes array values to matching elements
  ClassName('product-name').update({
    textContent: products.map(p => p.name)  // ['Product A', 'Product B', 'Product C']
  });

  ClassName('product-price').update({
    textContent: products.map(p => `$${p.price}`)
  });
});

// Index access (enhanced with Proxy)
ClassName('product-card')[0]    // First product card
ClassName('product-card')[1]    // Second product card
ClassName('product-card')[-1]   // Last product card

// Update one product - all cards update automatically
const products = storage.get('products');
products[0].price = 24.99;
storage.set('products', products);
```

**Why this is elegant:**
- **No forEach** - ClassName auto-iterates
- **Array-based updates** - Maps array values to elements
- **Index access** - Direct element access with `[0]`, `[-1]`
- Changing storage updates all cards at once

### **Example: Conditions + whenState - Reactive Class Rendering**
```javascript
const storage = ReactiveUtils.reactiveStorage('localStorage', 'app');

// Set initial state
storage.set('status', 'loading');
storage.set('userLevel', 5);

// Reactive conditions - apply classes based on storage values
ReactiveUtils.effect(() => {
  const status = storage.get('status');
  const level = storage.get('userLevel');

  // Pattern matching on status
  whenState(status, {
    'loading': 'spinner-active',
    'success': 'status-success',
    'error': 'status-error'
  }, '#statusIndicator');

  // Numeric range matching
  whenState(level, {
    '0-3': 'level-beginner',
    '4-7': 'level-intermediate',
    '8-10': 'level-expert'
  }, '#userBadge');
});

// Update storage - conditions apply automatically
storage.set('status', 'success');  // #statusIndicator gets .status-success
storage.set('userLevel', 8);       // #userBadge gets .level-expert
```

**Why conditions are powerful:**
- **Pattern matching** - No manual if/else chains
- **Declarative** - Define what, not how
- **Reactive** - Updates automatically on storage changes
- **Flexible** - Supports strings, numbers, ranges, regex

### **Example: updateAll() + Computed - Mixed State & DOM Updates**
```javascript
const storage = ReactiveUtils.reactiveStorage('localStorage', 'dashboard');
const appState = ReactiveUtils.state({
  counter: 0,
  lastUpdate: null
});

// Add computed property
appState.$computed('display', function() {
  return `Count: ${this.counter} (Updated: ${this.lastUpdate || 'Never'})`;
});

// Set storage
storage.set('theme', 'dark');
storage.set('notifications', 5);

// updateAll() - Update BOTH storage AND DOM in one call
function updateDashboard() {
  updateAll({
    // Update storage
    theme: 'light',
    notifications: 3,

    // Update DOM directly
    '#dashboard': {
      className: 'dashboard light-mode',
      dataset: { theme: 'light' }
    },

    // Update with Elements (ID-based)
    '#counter': { textContent: appState.counter },
    '#display': { textContent: appState.display }  // Uses computed
  });
}

// Bindings for automatic sync
bindings({
  '#notifications': () => storage.get('notifications') || 0,
  '#theme': () => storage.get('theme'),
  '#computed': () => appState.display  // Computed property
});

// Increment counter
appState.counter++;
appState.lastUpdate = new Date().toLocaleTimeString();
```

**The power of this approach:**
- **updateAll()** - Mixed storage + DOM updates
- **Computed properties** - Derived values with `$computed()`
- **bindings()** - Declarative DOM sync
- **Elements** - Fast ID-based access
- Everything stays in sync automatically

### **Example: Full Reactive System - Storage + State + Collections + Bindings**
```javascript
const storage = ReactiveUtils.reactiveStorage('localStorage', 'app');

// Reactive collections for array data
const todos = ReactiveUtils.collection([]);

// Reactive state for app data
const appState = ReactiveUtils.state({
  filter: 'all',
  theme: 'light'
});

// Load todos from storage
const savedTodos = storage.get('todos') || [];
savedTodos.forEach(todo => todos.add(todo));

// Watch collection changes - auto-save to storage
ReactiveUtils.watch(todos, () => {
  storage.set('todos', todos.items);
});

// Declarative bindings
bindings({
  // Elements - ID-based (fastest)
  '#todoCount': () => todos.length,
  '#activeCount': () => todos.filter(t => !t.done).length,

  // Computed display
  '#filterStatus': () => `Showing: ${appState.filter}`,

  // Theme indicator
  '#themeDisplay': () => appState.theme
});

// Conditions - reactive class application
ReactiveUtils.effect(() => {
  whenState(appState.theme, {
    'light': 'theme-light',
    'dark': 'theme-dark'
  }, 'body');

  whenState(todos.length, {
    '0': 'empty-state',
    '1-5': 'few-items',
    '>5': 'many-items'
  }, '#todoList');
});

// Render todos with ClassName array-based updates
ReactiveUtils.effect(() => {
  const filtered = todos.filter(t => {
    if (appState.filter === 'active') return !t.done;
    if (appState.filter === 'completed') return t.done;
    return true;
  });

  // Array-based update - NO forEach!
  ClassName('todo-item').update({
    textContent: filtered.map(t => t.text),
    dataset: { id: filtered.map(t => t.id) }
  });
});

// Add todo
function addTodo(text) {
  todos.add({ id: Date.now(), text, done: false });
  // Collection watch auto-saves to storage
}

// Toggle theme
function toggleTheme() {
  appState.theme = appState.theme === 'light' ? 'dark' : 'light';
  storage.set('theme', appState.theme);  // Persist theme
}
```

**The complete reactive system:**
- **Storage** - Persistent reactive data (localStorage/sessionStorage)
- **Collections** - Reactive arrays with `.add()`, `.remove()`, `.filter()`
- **State** - Reactive objects with computed properties
- **bindings()** - Declarative DOM synchronization
- **whenState** - Conditional rendering with pattern matching
- **Elements** - Fast ID-based DOM access
- **ClassName** - Auto-iteration with array-based updates
- **watch** - React to changes (auto-save to storage)
- Everything updates automatically when data changes!

