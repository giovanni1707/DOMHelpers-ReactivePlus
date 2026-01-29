# `effects()` - Create Multiple Effects at Once

## Quick Start (30 seconds)

```javascript
const counter = state({ count: 0, step: 1 });

// Create multiple effects at once
const cleanup = effects({
  updateDisplay: () => {
    Elements.update({
      count: { textContent: counter.count }
    });
  },
  updateDouble: () => {
    Elements.update({
      double: { textContent: counter.count * 2 }
    });
  },
  logChanges: () => {
    console.log('Count:', counter.count);
  }
});

counter.count++; // All three effects run

// Later: cleanup all at once
cleanup();
```

**That's it.** Create multiple named effects in one call. Cleaner than calling `effect()` multiple times, plus you get a single cleanup function.

---

## What is `effects()`?

`effects()` is **bulk effect creation**. Instead of calling `effect()` multiple times, you define all your effects in one object with descriptive names.

Think of it as **organizing your effects in a labeled collection** — each effect has a name, making your code more readable and maintainable.

**In practical terms:** Use `effects()` when you have multiple related effects and want to create/cleanup them all together.

---

## Syntax

```javascript
// Create multiple effects
const cleanup = effects(effectDefinitions);

// Effect definitions
const effectDefinitions = {
  effectName1: () => { /* effect logic */ },
  effectName2: () => { /* effect logic */ },
  effectName3: () => { /* effect logic */ }
};

// Example
const app = state({ theme: 'light', fontSize: 16 });

const cleanup = effects({
  applyTheme: () => {
    document.body.className = `theme-${app.theme}`;
  },
  applyFontSize: () => {
    document.documentElement.style.fontSize = `${app.fontSize}px`;
  },
  logChanges: () => {
    console.log('Theme:', app.theme, 'Font:', app.fontSize);
  }
});
```

**Parameters:**
- `effectDefinitions` - Object with named effect functions

**Returns:**
- Cleanup function that stops all effects

---

## Why Does This Exist?

### Without effects() - Repetitive

```javascript
// ❌ Vanilla JavaScript - manual DOM updates
let appTheme = 'light';
let appFontSize = 16;

function updateTheme() {
  document.body.className = `theme-${appTheme}`;
}

function updateFontSize() {
  document.documentElement.style.fontSize = `${appFontSize}px`;
}

function updateAll() {
  updateTheme();
  updateFontSize();
  console.log('Theme:', appTheme, 'Font:', appFontSize);
}

appTheme = 'dark';
updateAll(); // Must call manually
```

### With effects() - Organized

```javascript
// ✅ DOM Helpers + effects() - automatic and organized
const app = state({ theme: 'light', fontSize: 16 });

const cleanup = effects({
  applyTheme: () => {
    document.body.className = `theme-${app.theme}`;
  },
  applyFontSize: () => {
    document.documentElement.style.fontSize = `${app.fontSize}px`;
  },
  logChanges: () => {
    console.log('Theme:', app.theme, 'Font:', app.fontSize);
  }
});

app.theme = 'dark';
// ✨ All three effects run automatically
```

---

## Basic Examples

### Example 1: Multi-Display Updates

```javascript
const user = state({ name: 'Alice', status: 'online' });

effects({
  updateHeader: () => {
    Elements.update({
      headerName: { textContent: user.name }
    });
  },
  updateSidebar: () => {
    Elements.update({
      sidebarName: { textContent: user.name },
      sidebarStatus: { textContent: user.status }
    });
  },
  updateTitle: () => {
    document.title = `${user.name} - ${user.status}`;
  }
});
```

### Example 2: Validation Effects

```javascript
const form = state({ email: '', password: '' });

effects({
  validateEmail: () => {
    const valid = form.email.includes('@');
    Elements.update({
      emailError: {
        textContent: valid ? '' : 'Invalid email',
        style: { display: valid ? 'none' : 'block' }
      }
    });
  },
  validatePassword: () => {
    const valid = form.password.length >= 8;
    Elements.update({
      passwordError: {
        textContent: valid ? '' : 'Password too short',
        style: { display: valid ? 'none' : 'block' }
      }
    });
  }
});
```

### Example 3: Dashboard Updates

```javascript
const stats = state({ users: 100, revenue: 5000 });

effects({
  updateUserCount: () => {
    Elements.update({
      userCount: { textContent: stats.users }
    });
  },
  updateRevenue: () => {
    Elements.update({
      revenue: { textContent: `$${stats.revenue.toLocaleString()}` }
    });
  },
  updateGrowth: () => {
    const growth = (stats.revenue / stats.users).toFixed(2);
    Elements.update({
      avgRevenue: { textContent: `$${growth} per user` }
    });
  }
});
```

See full documentation with 10+ comprehensive examples.

---

## effects() vs Multiple effect() Calls

### Using effect() - Individual calls
```javascript
const cleanup1 = effect(() => { /* ... */ });
const cleanup2 = effect(() => { /* ... */ });
const cleanup3 = effect(() => { /* ... */ });

// Cleanup requires multiple calls
cleanup1();
cleanup2();
cleanup3();
```

### Using effects() - Organized
```javascript
const cleanup = effects({
  effect1: () => { /* ... */ },
  effect2: () => { /* ... */ },
  effect3: () => { /* ... */ }
});

// Single cleanup call
cleanup();
```

---

## Key Takeaways

✅ **Bulk creation** - Create multiple effects at once
✅ **Named effects** - Descriptive names for each effect
✅ **Single cleanup** - One function stops all effects
✅ **Organized** - All related effects together

---

## Summary

`effects()` creates **multiple named effects in one call**. Cleaner than individual `effect()` calls, with organized code and single cleanup.
