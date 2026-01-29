# `pause()` - Pause Reactivity

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

// Make changes - no effects run
app.count = 10;
app.status = 'paused';
// No output!

// Resume reactivity
resume(true);  // Pass true to flush pending updates
// Output: Count: 10 Status: paused
```

**That's it.** Pause reactive effect execution, make multiple changes, then resume. All pending updates run when you resume.

---

## What is `pause()`?

`pause()` **temporarily disables reactive effect execution**. While paused, state changes still happen but effects don't run. This is like an extended batch mode that you control manually.

Think of it as **the pause button for reactivity** — freeze effect execution, make as many changes as you need, then resume when ready.

**In practical terms:** Use `pause()` when you need to make many changes across different parts of your code and want effects to run only once when you're done.

---

## Syntax

```javascript
// Pause reactivity
pause();

// Make changes - effects queued but don't run
state.prop1 = value1;
state.prop2 = value2;

// Resume and optionally flush
resume();        // Resume without flushing
resume(true);    // Resume and run queued effects
```

**Parameters:**
- None

**Returns:**
- `undefined`

**Important:**
- Each `pause()` increments a **depth counter**
- Must call `resume()` same number of times
- Effects are **queued** while paused
- Use `resume(true)` to **flush** queued effects
- Can be called **multiple times** (nesting)

---

## Why Does This Exist?

### The Problem Without pause()

When making changes across multiple function calls, effects run multiple times:

```javascript
// ❌ Effects run after each change
const app = state({
  users: [],
  posts: [],
  comments: [],
  loading: true
});

effect(() => {
  console.log('Rendering dashboard...');
  renderDashboard(app);
});

function loadUsers() {
  app.users = fetchUsers();
  // Effect runs
}

function loadPosts() {
  app.posts = fetchPosts();
  // Effect runs
}

function loadComments() {
  app.comments = fetchComments();
  // Effect runs
}

async function loadAll() {
  await loadUsers();    // Effect runs
  await loadPosts();    // Effect runs
  await loadComments(); // Effect runs
  app.loading = false;  // Effect runs
}
// Effect runs 4 times!
```

**Problems:**
❌ **Multiple renders** - Effect runs after each function
❌ **Can't use batch()** - Changes across different functions
❌ **Performance waste** - Redundant effect executions
❌ **Incomplete state** - Effects see intermediate states
❌ **Code separation** - Hard to batch changes across modules

### The Solution with `pause()`

```javascript
// ✅ Pause before changes, resume after
const app = state({
  users: [],
  posts: [],
  comments: [],
  loading: true
});

effect(() => {
  console.log('Rendering dashboard...');
  renderDashboard(app);
});

function loadUsers() {
  app.users = fetchUsers();
  // Effect queued
}

function loadPosts() {
  app.posts = fetchPosts();
  // Effect queued
}

function loadComments() {
  app.comments = fetchComments();
  // Effect queued
}

async function loadAll() {
  pause();  // Pause reactivity

  await loadUsers();
  await loadPosts();
  await loadComments();
  app.loading = false;

  resume(true);  // Resume and flush
  // Effect runs once with all data!
}
```

**Benefits:**
✅ **Single effect run** - All changes, one execution
✅ **Cross-function batching** - Works across different functions
✅ **Manual control** - Pause and resume when you want
✅ **Performance** - Eliminates redundant executions
✅ **Complete state** - Effects see final state only

---

## Mental Model: Pause Button

Think of `pause()` like **pressing pause on a video player**:

**Without pause() (Always Running):**
```
┌─────────────────────────────┐
│  Reactivity Always Running  │
│                             │
│  state.a = 1                │
│  → Effect runs              │
│                             │
│  state.b = 2                │
│  → Effect runs              │
│                             │
│  state.c = 3                │
│  → Effect runs              │
│                             │
│  Effects run continuously   │
└─────────────────────────────┘
```

**With pause() (Manual Control):**
```
┌─────────────────────────────┐
│  Manual Pause/Resume        │
│                             │
│  pause()                    │
│  → Reactivity paused        │
│                             │
│  state.a = 1  (queued)      │
│  state.b = 2  (queued)      │
│  state.c = 3  (queued)      │
│                             │
│  resume(true)               │
│  → Flush queued updates     │
│  → Effect runs once         │
└─────────────────────────────┘
```

`pause()` is **the pause button** — stop effects, make changes, then resume.

---

## How Does It Work?

`pause()` increments a counter to disable effects:

```
Call pause()
    ↓
Increment batch depth
    ↓
Reactivity paused
    ↓
State changes:
  state.a = 1  → Change applied, effect queued
  state.b = 2  → Change applied, effect queued
    ↓
Call resume(flush)
    ↓
Decrement batch depth
    ↓
Depth = 0?
    ↓
   Yes
    ↓
flush = true?
    ↓
   Yes → Run all queued effects
```

**Key behaviors:**
- Increments **batch depth counter**
- State changes **still happen** while paused
- Effects are **queued** but don't run
- Must call `resume()` to **decrement counter**
- When depth reaches **0** and flush is **true**, effects run
- Supports **nesting** (multiple pause/resume pairs)

---

## Basic Usage

### Example 1: Bulk Data Loading

```javascript
const app = state({
  data: null,
  metadata: null,
  config: null,
  ready: false
});

effect(() => {
  if (app.ready) {
    renderApp(app);
  }
});

async function initialize() {
  pause();

  app.data = await fetchData();
  app.metadata = await fetchMetadata();
  app.config = await fetchConfig();
  app.ready = true;

  resume(true);
  // Effect runs once when all data is loaded
}
```

---

### Example 2: Form Population

```javascript
const form = state({
  firstName: '',
  lastName: '',
  email: '',
  address: '',
  city: '',
  zip: ''
});

effect(() => {
  updateFormUI(form);
});

function populateForm(userData) {
  pause();

  form.firstName = userData.firstName;
  form.lastName = userData.lastName;
  form.email = userData.email;
  form.address = userData.address;
  form.city = userData.city;
  form.zip = userData.zip;

  resume(true);
  // Form UI updates once with all fields filled
}
```

---

### Example 3: Game State Reset

```javascript
const game = state({
  score: 0,
  level: 1,
  health: 100,
  enemies: [],
  powerUps: [],
  paused: false
});

effect(() => {
  renderGame(game);
});

function resetGame() {
  pause();

  game.score = 0;
  game.level = 1;
  game.health = 100;
  game.enemies = [];
  game.powerUps = [];
  game.paused = false;

  resume(true);
  // Game renders once with reset state
}
```

---

### Example 4: Import CSV

```javascript
const data = state({
  records: [],
  imported: 0,
  failed: 0,
  status: 'idle'
});

effect(() => {
  updateImportProgress(data);
});

function importCSV(csvData) {
  const rows = parseCSV(csvData);

  pause();

  data.status = 'importing';
  data.records = [];
  data.imported = 0;
  data.failed = 0;

  rows.forEach((row, index) => {
    try {
      data.records.push(processRow(row));
      data.imported++;
    } catch (e) {
      console.error(`Row ${index} failed:`, e);
      data.failed++;
    }
  });

  data.status = 'complete';

  resume(true);
  // Progress updates once at the end
}
```

---

### Example 5: Multi-Step Wizard

```javascript
const wizard = state({
  step: 1,
  step1Data: {},
  step2Data: {},
  step3Data: {},
  completed: false
});

effect(() => {
  renderWizard(wizard);
});

function completeWizard(step1, step2, step3) {
  pause();

  wizard.step1Data = step1;
  wizard.step2Data = step2;
  wizard.step3Data = step3;
  wizard.step = 4;
  wizard.completed = true;

  resume(true);
  // Wizard updates to final state once
}
```

---

### Example 6: Canvas Drawing

```javascript
const canvas = state({
  shapes: [],
  selectedId: null,
  zoom: 1,
  pan: { x: 0, y: 0 }
});

effect(() => {
  redrawCanvas(canvas);
});

function loadDrawing(savedDrawing) {
  pause();

  canvas.shapes = savedDrawing.shapes;
  canvas.zoom = savedDrawing.zoom;
  canvas.pan = savedDrawing.pan;
  canvas.selectedId = null;

  resume(true);
  // Canvas redraws once with all shapes
}
```

---

### Example 7: Batch API Calls

```javascript
const api = state({
  users: [],
  projects: [],
  tasks: [],
  lastSync: null
});

effect(() => {
  updateDashboard(api);
});

async function syncAll() {
  pause();

  const [users, projects, tasks] = await Promise.all([
    fetch('/api/users').then(r => r.json()),
    fetch('/api/projects').then(r => r.json()),
    fetch('/api/tasks').then(r => r.json())
  ]);

  api.users = users;
  api.projects = projects;
  api.tasks = tasks;
  api.lastSync = new Date();

  resume(true);
  // Dashboard updates once with all data
}
```

---

### Example 8: Settings Migration

```javascript
const settings = state({
  theme: 'light',
  fontSize: 16,
  notifications: true,
  autoSave: true,
  version: 1
});

effect(() => {
  applySettings(settings);
  saveToStorage(settings);
});

function migrateSettings(oldSettings) {
  pause();

  // Migrate from old version
  settings.theme = oldSettings.darkMode ? 'dark' : 'light';
  settings.fontSize = oldSettings.textSize || 16;
  settings.notifications = oldSettings.enableNotifications ?? true;
  settings.autoSave = oldSettings.autoSave ?? true;
  settings.version = 2;

  resume(true);
  // Settings apply and save once
}
```

---

### Example 9: Undo System

```javascript
const editor = state({
  content: '',
  cursor: 0,
  selection: null,
  canUndo: false,
  canRedo: false
});

effect(() => {
  renderEditor(editor);
  updateToolbar(editor);
});

function restoreSnapshot(snapshot) {
  pause();

  editor.content = snapshot.content;
  editor.cursor = snapshot.cursor;
  editor.selection = snapshot.selection;
  editor.canUndo = undoStack.length > 0;
  editor.canRedo = redoStack.length > 0;

  resume(true);
  // Editor updates once with restored state
}
```

---

### Example 10: Complex Calculation

```javascript
const simulation = state({
  particles: [],
  forces: [],
  energy: 0,
  time: 0,
  running: false
});

effect(() => {
  renderSimulation(simulation);
});

function stepSimulation(deltaTime) {
  pause();

  // Update particles
  simulation.particles.forEach(p => {
    p.x += p.vx * deltaTime;
    p.y += p.vy * deltaTime;
  });

  // Update forces
  simulation.forces = calculateForces(simulation.particles);

  // Update energy
  simulation.energy = calculateEnergy(simulation.particles);

  // Update time
  simulation.time += deltaTime;

  resume(true);
  // Simulation renders once with all updates
}
```

---

## Advanced Usage: Nested Pause/Resume

```javascript
const app = state({
  data: {}
});

effect(() => {
  console.log('Updated');
});

function outerOperation() {
  pause();  // Depth: 1

  app.data.a = 1;

  innerOperation();

  app.data.d = 4;

  resume();  // Depth: 0, but no flush
}

function innerOperation() {
  pause();  // Depth: 2

  app.data.b = 2;
  app.data.c = 3;

  resume();  // Depth: 1
}

outerOperation();
// No effects run yet (no flush)

resume(true);
// Now effects run (depth 0 with flush)
```

---

## Common Patterns

### Pattern 1: Pause During Async

```javascript
async function loadData() {
  pause();

  const data = await fetchData();
  state.data = data;
  state.loading = false;

  resume(true);
}
```

### Pattern 2: Pause Around Loops

```javascript
pause();

items.forEach(item => {
  state.items.push(processItem(item));
});

resume(true);
```

### Pattern 3: Conditional Resume

```javascript
pause();

try {
  // Make changes
  state.value = compute();
  resume(true);  // Flush on success
} catch (e) {
  resume(false); // Don't flush on error
  throw e;
}
```

### Pattern 4: Module-Level Pause

```javascript
// moduleA.js
export function updateA(state) {
  state.a = 1;
}

// moduleB.js
export function updateB(state) {
  state.b = 2;
}

// main.js
pause();
updateA(state);
updateB(state);
resume(true);
```

---

## When to Use pause()

**✅ Use pause() when:**
- Making changes across multiple functions
- Long-running operations with many updates
- Importing/loading bulk data
- Resetting complex state
- Need manual control over batching

**❌ Don't use pause() when:**
- Single function has all changes (use `batch()`)
- Few changes that can use `set()`
- Effects should run after each change
- Simple updates

---

## Key Takeaways

✅ **Manual control** - Pause and resume when you want
✅ **Cross-function** - Works across different functions
✅ **Nestable** - Supports multiple pause/resume pairs
✅ **Queue effects** - Changes happen, effects queued
✅ **Optional flush** - Choose when to run effects
✅ **Performance** - One effect run for many changes

---

## What's Next?

- **`resume()`** - Resume reactivity with optional flush
- **`batch()`** - Automatic batching in single function
- **`untrack()`** - Run code without tracking dependencies

---

## Summary

`pause()` **temporarily disables reactive effect execution**. While paused, state changes happen but effects are queued. Call `resume(true)` to run all queued effects.

**The magic formula:**
```
pause();
// Make many changes
state.a = 1;
state.b = 2;
resume(true);
  =
Manual batch mode
─────────────────
Pause, change, resume
```

Think of it as **the pause button for reactivity** — temporarily freeze effect execution, make all your changes, then resume when ready. Perfect for bulk operations, data loading, and changes across multiple functions.
