# `resume()` - Resume Reactivity

## Quick Start (30 seconds)

```javascript
// Create state
const app = state({
  count: 0,
  status: 'active'
});

// Effect that logs changes
effect(() => {
  console.log('Count:', app.count, 'Status:', app.status);
});
// Output: Count: 0 Status: active

// Pause reactivity
pause();

// Make changes while paused
app.count = 10;
app.status = 'updated';
// No output yet

// Resume WITHOUT flush - effects stay queued
resume();

// Still no output...

// Resume WITH flush - run all queued effects
resume(true);
// Output: Count: 10 Status: updated
```

**That's it.** Resume reactivity after pausing. Pass `true` to flush queued effects, or omit to just decrement the pause counter.

---

## What is `resume()`?

`resume()` **decrements the pause counter and optionally runs all queued effects**. It's the counterpart to `pause()` that restores normal reactive behavior.

Think of it as **the play button after pause** — when you're done making changes, resume tells the system to start running effects again.

**In practical terms:** After calling `pause()`, use `resume(true)` to flush all pending updates and run effects with the final state.

---

## Syntax

```javascript
// Resume without flushing (just decrement counter)
resume();

// Resume and flush queued effects
resume(true);

// Typical pattern
pause();
// ... make changes ...
resume(true);  // Resume and run effects
```

**Parameters:**
- `flush` (optional) - If `true`, runs all queued effects when pause depth reaches 0

**Returns:**
- `undefined`

**Important:**
- Decrements the **pause counter**
- Effects only run when counter reaches **0** AND `flush` is **true**
- Must call `resume()` **same number of times** as `pause()`
- Safe to call even when **not paused**
- Pass `true` on the **final resume** to flush effects

---

## Why Does This Exist?

### The Problem Without resume()

After pausing, you need a way to resume and run effects:

```javascript
// ❌ Without resume - effects never run
const app = state({
  count: 0
});

effect(() => {
  console.log('Count:', app.count);
});

pause();
app.count = 10;
app.count = 20;
// Effects are queued but never run!
```

**Problems:**
❌ **Effects never run** - No way to trigger queued effects
❌ **State out of sync** - UI doesn't reflect state changes
❌ **Memory leak** - Queued effects pile up
❌ **Broken reactivity** - System stays paused forever

### The Solution with `resume()`

```javascript
// ✅ Resume flushes queued effects
const app = state({
  count: 0
});

effect(() => {
  console.log('Count:', app.count);
});

pause();
app.count = 10;
app.count = 20;
resume(true);  // Flush and run effect
// Output: Count: 20
```

**Benefits:**
✅ **Controlled execution** - Run effects when ready
✅ **Flush pending updates** - All queued effects execute
✅ **Restore reactivity** - System returns to normal
✅ **Clean state** - No memory leaks
✅ **Manual timing** - Choose when effects run

---

## Mental Model: Play Button

Think of `resume()` like **pressing play after pause**:

**Pause/Resume Cycle:**
```
┌─────────────────────────────┐
│  Normal (Depth: 0)          │
│  Effects run immediately    │
└─────────────────────────────┘
         ↓ pause()
┌─────────────────────────────┐
│  Paused (Depth: 1)          │
│  Effects queued             │
│  state.a = 1 (queued)       │
│  state.b = 2 (queued)       │
└─────────────────────────────┘
         ↓ resume()
┌─────────────────────────────┐
│  Check depth                │
│  Depth: 1 → 0               │
│  flush = true?              │
└─────────────────────────────┘
         ↓ Yes
┌─────────────────────────────┐
│  Run Queued Effects         │
│  Effect runs with:          │
│  a = 1, b = 2               │
└─────────────────────────────┘
         ↓
┌─────────────────────────────┐
│  Normal (Depth: 0)          │
│  Effects run immediately    │
└─────────────────────────────┘
```

`resume()` is **the play button** — unpause and run effects.

---

## How Does It Work?

`resume()` manages the pause counter:

```
Call resume(flush)
    ↓
Decrement batch depth
    ↓
Clamp to minimum 0
    ↓
Check depth
    ↓
depth = 0?
    ↓
   Yes
    ↓
flush = true?
    ↓
   Yes → Run all queued effects
    No → Just resume, keep queue
    ↓
Reactivity restored
```

**Key behaviors:**
- Decrements **pause counter**
- Counter **can't go negative** (clamped to 0)
- Only runs effects when depth is **0** and flush is **true**
- Multiple `pause()` calls require multiple `resume()` calls
- Safe to call **more times than pause()** (just stays at 0)

---

## Basic Usage

### Example 1: Simple Pause/Resume

```javascript
const counter = state({ count: 0 });

effect(() => {
  console.log('Count:', counter.count);
});
// Output: Count: 0

pause();

counter.count = 5;
counter.count = 10;

resume(true);
// Output: Count: 10 (runs once with final value)
```

---

### Example 2: Resume Without Flush

```javascript
const app = state({ value: 0 });

effect(() => {
  console.log('Value:', app.value);
});
// Output: Value: 0

pause();
app.value = 10;

resume();  // Resume but don't flush
// No output yet - effects still queued

app.value = 20;  // Still paused? No, depth is 0
// Output: Value: 20 (runs immediately, depth is 0)
```

---

### Example 3: Nested Pause/Resume

```javascript
const app = state({ x: 0 });

effect(() => {
  console.log('X:', app.x);
});

pause();    // Depth: 1
pause();    // Depth: 2

app.x = 10;

resume();   // Depth: 1 (still paused)
// No output

resume(true);  // Depth: 0 and flush
// Output: X: 10
```

---

### Example 4: Bulk Data Load

```javascript
const data = state({
  users: [],
  posts: [],
  comments: []
});

effect(() => {
  renderDashboard(data);
});

async function loadAll() {
  pause();

  data.users = await fetch('/api/users').then(r => r.json());
  data.posts = await fetch('/api/posts').then(r => r.json());
  data.comments = await fetch('/api/comments').then(r => r.json());

  resume(true);  // Flush - dashboard renders once
}
```

---

### Example 5: Conditional Flush

```javascript
const form = state({
  values: {},
  errors: {}
});

effect(() => {
  updateFormUI(form);
});

function validateAndUpdate(values) {
  pause();

  form.values = values;

  const errors = validate(values);
  form.errors = errors;

  if (Object.keys(errors).length === 0) {
    resume(true);  // Valid - flush updates
  } else {
    resume(false); // Invalid - don't flush
  }
}
```

---

### Example 6: Error Handling

```javascript
const app = state({
  data: null,
  error: null
});

effect(() => {
  renderApp(app);
});

async function loadData() {
  pause();

  try {
    app.data = await fetchData();
    app.error = null;
    resume(true);  // Success - flush
  } catch (e) {
    app.error = e.message;
    resume(true);  // Error - still flush to show error
  }
}
```

---

### Example 7: Animation Loop

```javascript
const animation = state({
  frame: 0,
  x: 0,
  y: 0
});

effect(() => {
  renderFrame(animation);
});

function animate() {
  requestAnimationFrame(() => {
    pause();

    animation.frame++;
    animation.x = Math.sin(animation.frame * 0.1) * 100;
    animation.y = Math.cos(animation.frame * 0.1) * 100;

    resume(true);  // Flush - render frame

    animate();  // Next frame
  });
}
```

---

### Example 8: Debounced Updates

```javascript
const search = state({
  query: '',
  results: []
});

effect(() => {
  displayResults(search.results);
});

let timeout;

function handleInput(value) {
  clearTimeout(timeout);

  pause();
  search.query = value;

  timeout = setTimeout(() => {
    search.results = performSearch(search.query);
    resume(true);  // Flush after delay
  }, 300);
}
```

---

### Example 9: Batch Import

```javascript
const data = state({
  records: [],
  count: 0
});

effect(() => {
  updateProgress(data.count);
});

function importRecords(records) {
  pause();

  records.forEach(record => {
    data.records.push(processRecord(record));
    data.count++;
  });

  resume(true);  // Flush - progress updates once at end
}
```

---

### Example 10: Transaction Rollback

```javascript
const db = state({
  records: [],
  lastBackup: []
});

effect(() => {
  saveToStorage(db.records);
});

function transaction(operations) {
  const backup = [...db.records];

  pause();

  try {
    operations.forEach(op => op(db));
    resume(true);  // Success - commit
  } catch (e) {
    db.records = backup;
    resume(false); // Failure - rollback, don't flush
    throw e;
  }
}
```

---

## Advanced Usage: Multiple Pause Levels

```javascript
const app = state({ value: 0 });

effect(() => {
  console.log('Value:', app.value);
});

function outer() {
  pause();  // Depth: 1

  app.value = 10;

  inner();

  app.value = 30;

  resume();  // Depth: 0, but no flush
}

function inner() {
  pause();  // Depth: 2

  app.value = 20;

  resume();  // Depth: 1, still paused
}

pause();  // Depth: 1
outer();  // Depth increases and decreases
// Depth still 1 here

resume(true);  // Depth: 0 and flush
// Output: Value: 30
```

---

## Common Patterns

### Pattern 1: Standard Pause/Resume

```javascript
pause();
// ... make changes ...
resume(true);
```

### Pattern 2: Try/Finally Safety

```javascript
pause();
try {
  // ... make changes ...
} finally {
  resume(true);  // Always resume
}
```

### Pattern 3: Conditional Flush

```javascript
pause();
const success = doWork();
resume(success);  // Flush only on success
```

### Pattern 4: Nested Operations

```javascript
function outer() {
  pause();
  // ...
  inner();  // May also pause/resume
  // ...
  resume();
}
```

---

## When to Use resume()

**✅ Use resume() when:**
- After calling `pause()`
- Ready to run queued effects
- Completing a batch of changes
- Restoring normal reactivity

**❌ Don't need resume() when:**
- Haven't called `pause()`
- Using `batch()` instead (auto-resumes)
- Want effects to keep queuing

---

## resume() vs batch()

| Feature | resume() | batch() |
|---------|----------|---------|
| Control | Manual | Automatic |
| Scope | Cross-function | Single function |
| Nesting | Manual tracking | Auto-managed |
| Flush | Optional | Automatic |
| Use case | Complex flows | Simple batching |

---

## Key Takeaways

✅ **Decrements counter** - Undoes pause
✅ **Optional flush** - Control when effects run
✅ **Restores reactivity** - System returns to normal
✅ **Nestable** - Works with multiple pause levels
✅ **Safe** - Can't go negative
✅ **Flexible** - Flush only when ready

---

## What's Next?

- **`pause()`** - Pause reactivity temporarily
- **`batch()`** - Automatic batching in functions
- **`untrack()`** - Run code without tracking

---

## Summary

`resume()` **decrements the pause counter and optionally runs all queued effects**. It's the counterpart to `pause()` that restores reactive behavior.

**The magic formula:**
```
pause();
// Make changes
resume(true);
  =
Pause → Change → Resume → Flush
────────────────────────────────
Manual reactivity control
```

Think of it as **the play button** — after pausing to make changes, resume restores normal reactivity and optionally flushes all queued effects. Essential counterpart to `pause()` for manual batching control.
