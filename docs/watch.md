# `watch()` - Watch Specific Properties

## Quick Start (30 seconds)

```javascript
// Create state
const user = state({
  name: 'Alice',
  email: 'alice@example.com',
  status: 'online'
});

// Watch specific properties
watch(user, {
  name: (newVal, oldVal) => {
    console.log(`Name changed from ${oldVal} to ${newVal}`);
  },
  status: (newVal) => {
    console.log(`Status is now: ${newVal}`);
  }
});

user.name = 'Bob';
// Console: "Name changed from Alice to Bob"

user.status = 'away';
// Console: "Status is now: away"
```

**That's it.** Watch specific state properties and run callbacks when they change. Perfect for logging, side effects, and reacting to specific changes.

---

## What is `watch()`?

`watch()` **watches specific properties and runs callbacks when they change**. Unlike `effect()` which automatically tracks all accessed properties, `watch()` lets you explicitly specify which properties to monitor.

Think of it as **a targeted listener** — you tell it exactly what to watch and what to do when that thing changes.

**In practical terms:** Use `watch()` when you need to react to specific property changes with callbacks, like saving to localStorage, making API calls, or logging changes.

---

## Syntax

```javascript
// Watch specific properties
watch(stateObject, watchDefs);

// Returns cleanup function
const unwatch = watch(stateObject, watchDefs);

// Watch definitions
const watchDefs = {
  propertyName: (newValue, oldValue) => {
    // Callback runs when property changes
  }
};

// Example
const settings = state({
  theme: 'light',
  language: 'en',
  fontSize: 16
});

const unwatch = watch(settings, {
  theme: (newTheme, oldTheme) => {
    console.log(`Theme changed: ${oldTheme} → ${newTheme}`);
    document.body.className = `theme-${newTheme}`;
  },
  language: (newLang) => {
    console.log(`Language changed to: ${newLang}`);
    loadTranslations(newLang);
  },
  fontSize: (newSize) => {
    document.documentElement.style.fontSize = `${newSize}px`;
  }
});

// Later: stop watching
unwatch();
```

**Parameters:**
- `stateObject` - Reactive state object to watch
- `watchDefs` - Object mapping property names to callback functions
- Callback signature: `(newValue, oldValue) => void`

**Returns:**
- Cleanup function to stop all watchers

---

## Why Does This Exist?

### The Problem Without watch()

Reacting to specific changes manually requires lots of code:

```javascript
// ❌ Vanilla JavaScript - manual change tracking
let settingsData = {
  theme: 'light',
  language: 'en',
  fontSize: 16,
  autoSave: true
};

// Must track old values manually
let oldTheme = settingsData.theme;
let oldLanguage = settingsData.language;
let oldFontSize = settingsData.fontSize;

// Manual setters for every property
function setTheme(newTheme) {
  const old = settingsData.theme;
  settingsData.theme = newTheme;

  // Manual callback
  console.log(`Theme changed: ${old} → ${newTheme}`);
  document.body.className = `theme-${newTheme}`;
  localStorage.setItem('theme', newTheme);
}

function setLanguage(newLang) {
  const old = settingsData.language;
  settingsData.language = newLang;

  // Manual callback
  console.log(`Language changed to: ${newLang}`);
  loadTranslations(newLang);
  localStorage.setItem('language', newLang);
}

function setFontSize(newSize) {
  const old = settingsData.fontSize;
  settingsData.fontSize = newSize;

  // Manual callback
  document.documentElement.style.fontSize = `${newSize}px`;
  localStorage.setItem('fontSize', newSize);
}

// Must use setters everywhere
setTheme('dark');
setLanguage('es');
setFontSize(18);

// Direct assignment doesn't trigger callbacks!
settingsData.theme = 'light'; // ❌ No callback runs!
```

**Problems:**
❌ **Manual setters** - Need function for every property
❌ **Easy to bypass** - Direct assignment doesn't work
❌ **Scattered logic** - Change handlers everywhere
❌ **Hard to maintain** - Lots of boilerplate

### The Solution with `watch()`

```javascript
// ✅ DOM Helpers + Reactive State with watch() - automatic callbacks
const settings = state({
  theme: 'light',
  language: 'en',
  fontSize: 16,
  autoSave: true
});

// Watch specific properties
watch(settings, {
  theme: (newTheme, oldTheme) => {
    console.log(`Theme changed: ${oldTheme} → ${newTheme}`);
    document.body.className = `theme-${newTheme}`;
    localStorage.setItem('theme', newTheme);
  },
  language: (newLang, oldLang) => {
    console.log(`Language changed to: ${newLang}`);
    loadTranslations(newLang);
    localStorage.setItem('language', newLang);
  },
  fontSize: (newSize, oldSize) => {
    console.log(`Font size: ${oldSize}px → ${newSize}px`);
    document.documentElement.style.fontSize = `${newSize}px`;
    localStorage.setItem('fontSize', newSize);
  }
});

// Just change properties - watchers run automatically!
settings.theme = 'dark';
// ✨ Watcher runs: logs, updates DOM, saves to localStorage

settings.language = 'es';
// ✨ Watcher runs: logs, loads translations, saves to localStorage

settings.fontSize = 18;
// ✨ Watcher runs: logs, updates font size, saves to localStorage
```

**Benefits:**
✅ **Automatic callbacks** - Watchers run on every change
✅ **Clean syntax** - Just assign properties normally
✅ **Organized** - All watchers defined together
✅ **Old and new values** - Both provided to callback
✅ **Easy cleanup** - Single unwatch() call

---

## Mental Model: Property Alarm

Think of `watch()` like **setting alarms on specific items**:

**Without watch() (Manual Checking):**
```
┌─────────────────────────────┐
│  Manual Checking            │
│                             │
│  function setTheme(val) {   │
│    oldVal = theme           │
│    theme = val              │
│    onThemeChange(val, old)  │
│  }                          │
│                             │
│  function setLang(val) {    │
│    oldVal = lang            │
│    lang = val               │
│    onLangChange(val, old)   │
│  }                          │
│                             │
│  Must wrap everything!      │
└─────────────────────────────┘
```

**With watch() (Automatic Alarms):**
```
┌─────────────────────────────┐
│  Automatic Alarms           │
│                             │
│  watch(state, {             │
│    theme: onChange,         │
│    lang: onChange           │
│  })                         │
│                             │
│  state.theme = 'dark'       │
│  🔔 Alarm! → onChange()     │
│                             │
│  state.lang = 'es'          │
│  🔔 Alarm! → onChange()     │
│                             │
│  Set and forget!            │
└─────────────────────────────┘
```

You **set alarms once**, then they **trigger automatically** when values change.

---

## Basic Usage

### Example 1: Auto-save to localStorage

```javascript
const document = state({
  title: '',
  content: '',
  lastSaved: null
});

// Auto-save when title or content changes
watch(document, {
  title: (newTitle) => {
    localStorage.setItem('doc-title', newTitle);
    document.lastSaved = new Date();
  },
  content: (newContent) => {
    localStorage.setItem('doc-content', newContent);
    document.lastSaved = new Date();
  }
});

// Type in the document
document.title = 'My Document';
// ✨ Auto-saved to localStorage

document.content = 'Hello world';
// ✨ Auto-saved to localStorage
```

---

### Example 2: Logging Changes

```javascript
const game = state({
  score: 0,
  level: 1,
  lives: 3
});

// Log all changes
watch(game, {
  score: (newScore, oldScore) => {
    const diff = newScore - oldScore;
    console.log(`Score ${diff > 0 ? '+' : ''}${diff}: ${oldScore} → ${newScore}`);
  },
  level: (newLevel, oldLevel) => {
    console.log(`Level up! ${oldLevel} → ${newLevel}`);
    playSound('levelup.mp3');
  },
  lives: (newLives, oldLives) => {
    if (newLives < oldLives) {
      console.log(`Lost a life! ${newLives} remaining`);
      playSound('hurt.mp3');
    }
  }
});

game.score = 100;  // Logs: "Score +100: 0 → 100"
game.level = 2;    // Logs: "Level up! 1 → 2" + plays sound
game.lives = 2;    // Logs: "Lost a life! 2 remaining" + plays sound
```

---

### Example 3: API Sync

```javascript
const profile = state({
  name: 'Alice',
  bio: 'Developer',
  avatar: 'avatar.jpg'
});

// Sync changes to server
watch(profile, {
  name: async (newName) => {
    await fetch('/api/profile', {
      method: 'PATCH',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ name: newName })
    });
    console.log('Name saved to server');
  },
  bio: async (newBio) => {
    await fetch('/api/profile', {
      method: 'PATCH',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ bio: newBio })
    });
    console.log('Bio saved to server');
  }
});

profile.name = 'Bob';
// ✨ Sends PATCH request to server
```

---

### Example 4: Dependent State Updates

```javascript
const cart = state({
  items: [],
  selectedShipping: 'standard',
  shippingCost: 5
});

// Update shipping cost when method changes
watch(cart, {
  selectedShipping: (method) => {
    const costs = {
      standard: 5,
      express: 15,
      overnight: 30
    };
    cart.shippingCost = costs[method];
  },
  items: (newItems) => {
    console.log(`Cart now has ${newItems.length} items`);

    // Free shipping on 5+ items
    if (newItems.length >= 5 && cart.selectedShipping === 'standard') {
      cart.shippingCost = 0;
    }
  }
});

cart.selectedShipping = 'express';
// ✨ Shipping cost automatically updates to 15

cart.items = [1, 2, 3, 4, 5];
// ✨ Logs item count, sets free shipping
```

---

### Example 5: Validation on Change

```javascript
const form = state({
  email: '',
  emailError: '',
  password: '',
  passwordError: ''
});

// Validate on change
watch(form, {
  email: (newEmail) => {
    if (newEmail && !newEmail.includes('@')) {
      form.emailError = 'Invalid email format';
    } else {
      form.emailError = '';
    }
  },
  password: (newPassword) => {
    if (newPassword && newPassword.length < 8) {
      form.passwordError = 'Password must be at least 8 characters';
    } else {
      form.passwordError = '';
    }
  }
});

form.email = 'invalid';
// ✨ Sets emailError

form.email = 'valid@example.com';
// ✨ Clears emailError

form.password = 'short';
// ✨ Sets passwordError
```

---

### Example 6: Analytics Tracking

```javascript
const page = state({
  currentRoute: '/home',
  searchQuery: '',
  selectedCategory: 'all'
});

// Track user actions
watch(page, {
  currentRoute: (newRoute, oldRoute) => {
    analytics.pageView({
      from: oldRoute,
      to: newRoute,
      timestamp: new Date()
    });
  },
  searchQuery: (query) => {
    if (query) {
      analytics.search({
        query,
        timestamp: new Date()
      });
    }
  },
  selectedCategory: (category) => {
    analytics.filterChange({
      filter: 'category',
      value: category,
      timestamp: new Date()
    });
  }
});

page.currentRoute = '/products';
// ✨ Sends pageView event to analytics

page.searchQuery = 'laptop';
// ✨ Sends search event to analytics
```

---

### Example 7: Theme System

```javascript
const theme = state({
  mode: 'light',
  primaryColor: '#007bff',
  fontSize: 16
});

// Apply theme changes
watch(theme, {
  mode: (newMode) => {
    document.body.classList.remove('light-mode', 'dark-mode');
    document.body.classList.add(`${newMode}-mode`);

    // Update meta tag
    const metaTheme = document.querySelector('meta[name="theme-color"]');
    metaTheme.content = newMode === 'dark' ? '#1a1a1a' : '#ffffff';
  },
  primaryColor: (newColor) => {
    document.documentElement.style.setProperty('--primary-color', newColor);
  },
  fontSize: (newSize) => {
    document.documentElement.style.setProperty('--base-font-size', `${newSize}px`);
  }
});

theme.mode = 'dark';
// ✨ Applies dark mode, updates meta tag

theme.primaryColor = '#ff0000';
// ✨ Updates CSS variable
```

---

### Example 8: Notification System

```javascript
const notifications = state({
  unreadCount: 0,
  latestMessage: null
});

// React to notifications
watch(notifications, {
  unreadCount: (newCount, oldCount) => {
    // Update favicon badge
    updateFaviconBadge(newCount);

    // Update page title
    document.title = newCount > 0
      ? `(${newCount}) Messages`
      : 'Messages';

    // Show desktop notification on new message
    if (newCount > oldCount) {
      showDesktopNotification('You have new messages');
    }
  },
  latestMessage: (message) => {
    if (message) {
      // Play sound
      playNotificationSound();

      // Show toast
      showToast(`New message from ${message.sender}`);
    }
  }
});

notifications.unreadCount = 5;
// ✨ Updates favicon, title, shows notification

notifications.latestMessage = { sender: 'Alice', text: 'Hello!' };
// ✨ Plays sound, shows toast
```

---

### Example 9: Undo/Redo System

```javascript
const editor = state({
  content: '',
  history: [],
  historyIndex: -1
});

// Track changes for undo/redo
watch(editor, {
  content: (newContent, oldContent) => {
    // Only track if content actually changed
    if (newContent !== oldContent) {
      // Remove any future history
      editor.history = editor.history.slice(0, editor.historyIndex + 1);

      // Add to history
      editor.history.push(newContent);
      editor.historyIndex++;

      // Limit history size
      if (editor.history.length > 50) {
        editor.history.shift();
        editor.historyIndex--;
      }
    }
  }
});

function undo() {
  if (editor.historyIndex > 0) {
    editor.historyIndex--;
    editor.content = editor.history[editor.historyIndex];
  }
}

function redo() {
  if (editor.historyIndex < editor.history.length - 1) {
    editor.historyIndex++;
    editor.content = editor.history[editor.historyIndex];
  }
}
```

---

### Example 10: Multi-language System

```javascript
const i18n = state({
  currentLanguage: 'en',
  translations: {}
});

// Load translations when language changes
watch(i18n, {
  currentLanguage: async (newLang, oldLang) => {
    console.log(`Switching language: ${oldLang} → ${newLang}`);

    try {
      // Show loading
      document.body.classList.add('loading-translations');

      // Fetch translations
      const response = await fetch(`/translations/${newLang}.json`);
      i18n.translations = await response.json();

      // Update HTML lang attribute
      document.documentElement.lang = newLang;

      // Save preference
      localStorage.setItem('language', newLang);

      console.log(`Loaded ${Object.keys(i18n.translations).length} translations`);
    } catch (error) {
      console.error('Failed to load translations:', error);
    } finally {
      document.body.classList.remove('loading-translations');
    }
  }
});

i18n.currentLanguage = 'es';
// ✨ Loads Spanish translations, updates DOM
```

---

## watch() vs effect()

### Using effect() (Auto-track all dependencies)
```javascript
const user = state({ name: 'Alice', age: 30 });

effect(() => {
  // Automatically tracks both name AND age
  console.log(`${user.name} is ${user.age}`);
});

user.name = 'Bob';  // Effect runs
user.age = 31;      // Effect runs
```

### Using watch() (Specific properties only)
```javascript
const user = state({ name: 'Alice', age: 30 });

watch(user, {
  // Only watches name
  name: (newName) => {
    console.log(`Name changed to ${newName}`);
  }
});

user.name = 'Bob';  // Watcher runs
user.age = 31;      // Watcher does NOT run
```

**When to use watch():**
✅ Need old and new values
✅ React to specific properties only
✅ Side effects (API calls, localStorage, logging)
✅ Don't need DOM updates

**When to use effect():**
✅ Auto-track multiple dependencies
✅ DOM updates
✅ Complex reactive logic
✅ Don't care about old values

---

## Cleanup

```javascript
// Create watcher
const unwatch = watch(state, {
  theme: (newTheme) => {
    console.log('Theme:', newTheme);
  }
});

// Later: stop watching
unwatch();

// Changes no longer trigger watcher
state.theme = 'dark'; // No callback runs
```

---

## Common Patterns

### Pattern 1: Debounced Save

```javascript
let saveTimeout;

watch(document, {
  content: (newContent) => {
    clearTimeout(saveTimeout);
    saveTimeout = setTimeout(() => {
      saveToServer(newContent);
    }, 1000);
  }
});
```

### Pattern 2: Conditional Watching

```javascript
watch(settings, {
  autoSave: (isEnabled) => {
    if (isEnabled) {
      // Start auto-save interval
    } else {
      // Stop auto-save interval
    }
  }
});
```

### Pattern 3: Chain Updates

```javascript
watch(cart, {
  items: (items) => {
    // Update count
    cart.itemCount = items.length;

    // Update total
    cart.total = items.reduce((sum, item) => sum + item.price, 0);
  }
});
```

---

## Key Takeaways

✅ **Targeted watching** - Watch specific properties
✅ **Old and new values** - Both provided to callback
✅ **Side effects** - Perfect for logging, API calls, storage
✅ **Cleanup** - Returns unwatch function
✅ **Explicit** - Clear which properties are watched

---

## What's Next?

- **`safeWatch()`** - Watch with error handling
- **`effect()`** - Auto-tracking reactive effects
- **`computed()`** - Cached computed properties

---

## Summary

`watch()` **watches specific properties and runs callbacks when they change**. Perfect for side effects, logging, and reacting to specific state changes.

**The magic formula:**
```
watch(state, {
  prop: (newVal, oldVal) => { /* callback */ }
}) =
  Targeted property listener with old/new values
───────────────────────────────────────────────
Set alarms on specific properties
```

Think of it as **setting alarms** — you specify exactly what to watch and what to do when it changes. The alarm triggers automatically every time that property changes, giving you both old and new values to work with.
