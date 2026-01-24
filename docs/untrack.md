# `untrack()` - Run Code Without Tracking Dependencies

## Quick Start (30 seconds)

```javascript
// Create state
const app = state({
  count: 0,
  timestamp: null
});

// Effect watches count
effect(() => {
  console.log('Count changed to:', app.count);

  // Update timestamp without creating dependency
  untrack(() => {
    app.timestamp = Date.now();
  });
});

app.count = 1;
// Output: Count changed to: 1

app.timestamp = Date.now();
// No output - timestamp changes don't re-run effect
```

**That's it.** Run code inside effects without creating reactive dependencies. Perfect for side effects that shouldn't trigger re-runs.

---

## What is `untrack()`?

`untrack()` **runs a function without tracking reactive dependencies**. When used inside an effect, any state accessed within `untrack()` won't be tracked, preventing infinite loops and unwanted re-runs.

Think of it as **the blindfold for reactivity** — the code runs but the reactive system doesn't watch what it touches.

**In practical terms:** Use `untrack()` to access state or modify it inside effects without creating dependencies that would cause the effect to re-run.

---

## Syntax

```javascript
// Run function without tracking
const result = untrack(() => {
  // Access or modify state here
  // Won't create dependencies
  return someValue;
});

// Example in effect
effect(() => {
  console.log('Count:', app.count);  // Tracked

  untrack(() => {
    app.lastUpdate = Date.now();  // Not tracked
    logToAnalytics(app.count);     // Reading count here not tracked
  });
});
```

**Parameters:**
- `fn` - Function to run without dependency tracking

**Returns:**
- The return value of the function

**Important:**
- State access inside `untrack()` **doesn't create dependencies**
- State modifications inside `untrack()` **still trigger other effects**
- Can be **nested** - inner `untrack()` has no effect (already untracked)
- Only affects **dependency tracking**, not reactive updates

---

## Why Does This Exist?

### The Problem Without untrack()

Accessing state inside effects creates dependencies that cause infinite loops:

```javascript
// ❌ Without untrack - infinite loop!
const app = state({
  count: 0,
  lastUpdate: null
});

effect(() => {
  console.log('Count:', app.count);

  // Accessing lastUpdate creates dependency
  // Modifying it triggers this effect again!
  app.lastUpdate = Date.now();  // ⚠️ Infinite loop!
});

// Effect runs → reads count → writes lastUpdate →
// Effect runs again → reads lastUpdate → writes lastUpdate →
// Effect runs again → ...forever!
```

**Problems:**
❌ **Infinite loops** - Effect triggers itself
❌ **Can't modify state** - Writing in effects causes re-runs
❌ **Can't read metadata** - Accessing state creates unwanted dependencies
❌ **Verbose workarounds** - Need separate effects or manual tracking

### The Solution with `untrack()`

```javascript
// ✅ With untrack - clean and safe
const app = state({
  count: 0,
  lastUpdate: null
});

effect(() => {
  console.log('Count:', app.count);

  // Modify state without creating dependency
  untrack(() => {
    app.lastUpdate = Date.now();  // ✅ Safe - won't cause re-run
  });
});

app.count = 1;
// Output: Count: 1 (effect runs once)

app.lastUpdate = Date.now();
// No output - lastUpdate changes don't trigger effect
```

**Benefits:**
✅ **No infinite loops** - Safe state modification in effects
✅ **Selective dependencies** - Track only what matters
✅ **Side effects** - Logging, analytics without dependencies
✅ **Metadata updates** - Track timestamps, counters safely
✅ **Clean code** - No workarounds needed

---

## Mental Model: Blindfold for Reactivity

Think of `untrack()` like **wearing a blindfold**:

**Normal Effect (Watching):**
```
┌─────────────────────────────┐
│  Effect Watching            │
│                             │
│  effect(() => {             │
│    Read state.a  👁️         │
│    → Track dependency       │
│                             │
│    Read state.b  👁️         │
│    → Track dependency       │
│  })                         │
│                             │
│  Effect re-runs when        │
│  a or b changes             │
└─────────────────────────────┘
```

**With untrack() (Blindfolded):**
```
┌─────────────────────────────┐
│  Effect with Untrack        │
│                             │
│  effect(() => {             │
│    Read state.a  👁️         │
│    → Track dependency       │
│                             │
│    untrack(() => {          │
│      Read state.b  🙈       │
│      → No tracking!         │
│    })                       │
│  })                         │
│                             │
│  Effect re-runs only when   │
│  a changes (not b)          │
└─────────────────────────────┘
```

`untrack()` is **the blindfold** — reactivity can't see what happens inside.

---

## How Does It Work?

`untrack()` temporarily disables dependency tracking:

```
Call untrack(fn)
    ↓
Save current effect
    ↓
Set currentEffect = null
    ↓
Execute function
  ├─ State access → Not tracked
  ├─ State modification → Still reactive
  └─ No dependencies created
    ↓
Restore previous effect
    ↓
Return function result
```

**Key behaviors:**
- Saves the **current effect** context
- Sets current effect to **null** during execution
- State reads **don't create dependencies**
- State writes **still trigger other effects**
- Restores context in **finally block** (error-safe)
- Can be **nested** (inner untrack has no effect)

---

## Basic Usage

### Example 1: Timestamp Updates

```javascript
const counter = state({
  count: 0,
  lastModified: null
});

// Update timestamp without tracking
effect(() => {
  console.log('Count:', counter.count);

  untrack(() => {
    counter.lastModified = new Date();
  });
});

counter.count = 5;
// Output: Count: 5
// lastModified updated, but doesn't trigger effect
```

---

### Example 2: Analytics Logging

```javascript
const app = state({
  currentPage: 'home',
  visitCount: 0
});

// Log analytics without creating dependencies
effect(() => {
  console.log('Page:', app.currentPage);

  untrack(() => {
    // Read visitCount for logging without tracking it
    logAnalytics({
      page: app.currentPage,
      visits: app.visitCount,
      timestamp: Date.now()
    });
  });
});

app.visitCount = 100;
// No effect re-run - visitCount not tracked
```

---

### Example 3: Debugging

```javascript
const data = state({
  items: [],
  selectedId: null
});

// Debug logging without dependencies
effect(() => {
  const item = data.items.find(i => i.id === data.selectedId);

  console.log('Selected item:', item);

  untrack(() => {
    // Debug logging that won't affect dependencies
    console.debug('Full state:', {
      itemCount: data.items.length,
      allItems: data.items,
      timestamp: Date.now()
    });
  });
});
```

---

### Example 4: Performance Metrics

```javascript
const app = state({
  users: [],
  loadTime: 0
});

effect(() => {
  const startTime = Date.now();

  // Render users
  renderUserList(app.users);

  untrack(() => {
    // Update performance metric without creating dependency
    app.loadTime = Date.now() - startTime;
  });
});

// Check load time without triggering effect
console.log('Load time:', app.loadTime);
```

---

### Example 5: Conditional Computation

```javascript
const config = state({
  enableFeature: true,
  featureConfig: {},
  debugMode: false
});

effect(() => {
  if (config.enableFeature) {
    setupFeature(config.featureConfig);
  }

  // Access debug mode without tracking
  untrack(() => {
    if (config.debugMode) {
      console.log('Feature config:', config.featureConfig);
    }
  });
});

config.debugMode = true;
// No re-run - debugMode not tracked
```

---

### Example 6: Event Counter

```javascript
const events = state({
  clickCount: 0,
  lastClick: null,
  eventLog: []
});

effect(() => {
  console.log('Clicks:', events.clickCount);

  untrack(() => {
    // Update metadata without causing re-run
    events.lastClick = Date.now();
    events.eventLog.push({
      count: events.clickCount,
      time: Date.now()
    });
  });
});
```

---

### Example 7: Computed with Side Effects

```javascript
const cart = state({
  items: [],
  totalCalculations: 0
});

computed(cart, {
  total: function() {
    const sum = this.items.reduce((s, item) => s + item.price, 0);

    // Track computation count without creating dependency
    untrack(() => {
      this.totalCalculations++;
    });

    return sum;
  }
});

console.log(cart.total);  // Increments totalCalculations
console.log(cart.totalCalculations);  // Won't re-compute total
```

---

### Example 8: Caching

```javascript
const cache = state({
  data: {},
  cacheHits: 0,
  cacheMisses: 0
});

effect(() => {
  const key = getRequestKey();
  const cached = cache.data[key];

  if (cached) {
    untrack(() => {
      cache.cacheHits++;
    });
    return cached;
  } else {
    untrack(() => {
      cache.cacheMisses++;
    });
    const data = fetchData(key);
    cache.data[key] = data;
    return data;
  }
});
```

---

### Example 9: Validation Counter

```javascript
const form = state({
  email: '',
  password: '',
  validationCount: 0
});

effect(() => {
  const emailValid = form.email.includes('@');
  const passwordValid = form.password.length >= 8;

  console.log('Valid:', emailValid && passwordValid);

  untrack(() => {
    // Increment counter without triggering re-validation
    form.validationCount++;
  });
});
```

---

### Example 10: History Tracking

```javascript
const editor = state({
  content: '',
  history: [],
  historyIndex: -1
});

effect(() => {
  // Save to history when content changes
  const snapshot = {
    content: editor.content,
    timestamp: Date.now()
  };

  untrack(() => {
    // Update history without creating dependencies
    editor.history.push(snapshot);
    editor.historyIndex = editor.history.length - 1;
  });
});

// Navigate history without triggering effect
editor.historyIndex = 0;  // No effect run
```

---

## Advanced Usage: Complex Side Effects

```javascript
const app = state({
  data: [],
  searchQuery: '',
  searchCount: 0,
  lastSearch: null,
  suggestions: []
});

effect(() => {
  // Main dependency: searchQuery
  const results = search(app.data, app.searchQuery);

  untrack(() => {
    // Update metrics
    app.searchCount++;
    app.lastSearch = Date.now();

    // Generate suggestions based on results
    // (reading results is fine, already in this effect)
    app.suggestions = generateSuggestions(results, app.data);

    // Log analytics
    logSearch({
      query: app.searchQuery,
      resultCount: results.length,
      count: app.searchCount
    });
  });

  return results;
});

// Changing suggestions won't re-run search
app.suggestions = customSuggestions;
```

---

## Common Patterns

### Pattern 1: Update Metadata

```javascript
effect(() => {
  doWork(state.input);

  untrack(() => {
    state.lastUpdate = Date.now();
    state.updateCount++;
  });
});
```

### Pattern 2: Logging Without Tracking

```javascript
effect(() => {
  const result = compute(state.value);

  untrack(() => {
    console.log('Debug:', {
      input: state.value,
      output: result,
      state: state
    });
  });
});
```

### Pattern 3: Conditional Side Effects

```javascript
effect(() => {
  processData(state.data);

  untrack(() => {
    if (state.debugMode) {
      logDebug(state);
    }
  });
});
```

### Pattern 4: Performance Tracking

```javascript
effect(() => {
  const start = Date.now();

  render(state.data);

  untrack(() => {
    state.renderTime = Date.now() - start;
  });
});
```

---

## When to Use untrack()

**✅ Use untrack() when:**
- Updating metadata (timestamps, counters)
- Logging or analytics inside effects
- Reading state for debugging without dependencies
- Preventing infinite loops from state writes
- Conditional side effects that shouldn't re-trigger

**❌ Don't use untrack() when:**
- You want the dependency tracked
- Not inside an effect (no effect to matter)
- The access should trigger re-runs

---

## untrack() vs toRaw()

| Feature | untrack() | toRaw() |
|---------|-----------|---------|
| Purpose | Skip tracking | Get raw object |
| Scope | Function only | Object access |
| Changes reactive? | Yes | No |
| Use in effects | Yes | Yes |
| Prevents loops | Yes | Depends |

---

## Key Takeaways

✅ **No dependencies** - Reads don't create tracking
✅ **Still reactive** - Writes trigger other effects
✅ **Safe in effects** - Prevents infinite loops
✅ **Selective tracking** - Choose what to track
✅ **Side effects** - Logging, metrics, debugging
✅ **Error-safe** - Cleanup in finally block

---

## What's Next?

- **`pause()`** - Pause all reactivity temporarily
- **`batch()`** - Batch multiple updates
- **`toRaw()`** - Get non-reactive object

---

## Summary

`untrack()` **runs a function without tracking reactive dependencies**. State accessed inside won't create dependencies, but state writes still trigger other effects.

**The magic formula:**
```
effect(() => {
  Read state.a     // Tracked
  untrack(() => {
    Read state.b   // Not tracked
    state.c = 1    // Still reactive
  })
})
  =
Selective dependency tracking
────────────────────────────
Access without tracking
```

Think of it as **the blindfold for reactivity** — run code that touches state without the reactive system watching. Essential for metadata updates, logging, debugging, and preventing infinite loops in effects. Perfect for side effects that shouldn't create dependencies.
