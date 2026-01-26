# `autoSave()` Options - Configuration Reference

**Quick Start (30 seconds)**
```javascript
const state = ReactiveUtils.reactive({
  count: 0,
  user: { name: 'John' }
});

// Configure autoSave with options
ReactiveUtils.autoSave(state, 'appState', {
  storage: 'localStorage',      // or 'sessionStorage'
  namespace: 'myApp',            // Key prefix
  debounce: 1000,                // Wait 1s before saving
  autoLoad: true,                // Load on init
  autoSave: true,                // Auto-save on changes
  sync: true,                    // Cross-tab sync
  expires: 3600,                 // Expire in 1 hour
  onSave: (value) => value,      // Transform before save
  onLoad: (value) => value,      // Transform after load
  onSync: (value) => {},         // Called on cross-tab sync
  onError: (error, type) => {}   // Error handler
});
```

---

## **All Options**

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `storage` | string | `'localStorage'` | Storage type: localStorage or sessionStorage |
| `namespace` | string | `''` | Key prefix for organization |
| `debounce` | number | `0` | Delay in ms before saving |
| `autoLoad` | boolean | `true` | Load data on initialization |
| `autoSave` | boolean | `true` | Auto-save on state changes |
| `sync` | boolean | `false` | Cross-tab synchronization |
| `expires` | number | `null` | Expiration time in seconds |
| `onSave` | function | `null` | Transform before saving |
| `onLoad` | function | `null` | Transform after loading |
| `onSync` | function | `null` | Called on cross-tab sync |
| `onError` | function | `null` | Error handler callback |

---

## **Option 1: `storage`**

### **Description**
Specifies which browser storage to use: `'localStorage'` (persists forever) or `'sessionStorage'` (cleared on tab close).

### **Type**
`string`

### **Default**
`'localStorage'`

### **Values**
- `'localStorage'` - Data persists across browser sessions
- `'sessionStorage'` - Data cleared when tab/window closes

### **Syntax**
```javascript
ReactiveUtils.autoSave(state, 'key', {
  storage: 'localStorage'
});
```

### **Examples**

**Persistent Data (localStorage)**
```javascript
const userPrefs = ReactiveUtils.reactive({ theme: 'dark' });

ReactiveUtils.autoSave(userPrefs, 'preferences', {
  storage: 'localStorage'
});
// Survives browser restart
```

**Session Data (sessionStorage)**
```javascript
const formDraft = ReactiveUtils.reactive({ title: '', content: '' });

ReactiveUtils.autoSave(formDraft, 'draft', {
  storage: 'sessionStorage'
});
// Cleared when tab closes
```

### **When to Use**
- **localStorage**: User preferences, settings, cached data
- **sessionStorage**: Form drafts, temporary data, sensitive session info

---

## **Option 2: `namespace`**

### **Description**
Prefix added to storage keys for organization and collision prevention.

### **Type**
`string`

### **Default**
`''` (no namespace)

### **Syntax**
```javascript
ReactiveUtils.autoSave(state, 'key', {
  namespace: 'myApp'
});
// Stored as: "myApp:key"
```

### **Examples**

**Without Namespace**
```javascript
ReactiveUtils.autoSave(state, 'settings');
// Stored as: "settings"
```

**With Namespace**
```javascript
ReactiveUtils.autoSave(state, 'settings', {
  namespace: 'myApp'
});
// Stored as: "myApp:settings"
```

**Multiple Namespaces**
```javascript
const userState = ReactiveUtils.reactive({});
const appState = ReactiveUtils.reactive({});

ReactiveUtils.autoSave(userState, 'data', { namespace: 'user' });
// Stored as: "user:data"

ReactiveUtils.autoSave(appState, 'data', { namespace: 'app' });
// Stored as: "app:data"
// No collision!
```

### **When to Use**
- Prevent key collisions between features
- Organize storage by feature/module
- Multi-tenant applications
- Library/component isolation

---

## **Option 3: `debounce`**

### **Description**
Delay in milliseconds before saving, prevents excessive saves during rapid changes.

### **Type**
`number`

### **Default**
`0` (no debounce)

### **Syntax**
```javascript
ReactiveUtils.autoSave(state, 'key', {
  debounce: 1000  // Wait 1 second
});
```

### **Examples**

**No Debounce (Immediate Save)**
```javascript
ReactiveUtils.autoSave(state, 'data', {
  debounce: 0
});
// Saves immediately on every change
```

**1 Second Debounce**
```javascript
const editor = ReactiveUtils.reactive({ content: '' });

ReactiveUtils.autoSave(editor, 'document', {
  debounce: 1000
});

editor.content = 'a';  // Starts timer
editor.content = 'ab'; // Resets timer
editor.content = 'abc'; // Resets timer
// Saves 1 second after last change
```

**Form Auto-Save**
```javascript
const form = ReactiveUtils.reactive({
  title: '',
  content: ''
});

ReactiveUtils.autoSave(form, 'draft', {
  debounce: 2000  // Save 2s after user stops typing
});
```

### **When to Use**
- Text editors (500-2000ms)
- Form auto-save (1000-3000ms)
- Frequently changing data
- Performance optimization

### **Recommended Values**
- **0ms**: Critical data that must save immediately
- **500ms**: Fast-typing scenarios
- **1000-2000ms**: Text editors, forms
- **3000-5000ms**: Less critical auto-save

---

## **Option 4: `autoLoad`**

### **Description**
Automatically load data from storage when `autoSave()` is called.

### **Type**
`boolean`

### **Default**
`true`

### **Syntax**
```javascript
ReactiveUtils.autoSave(state, 'key', {
  autoLoad: true
});
```

### **Examples**

**Auto-Load Enabled (Default)**
```javascript
const settings = ReactiveUtils.reactive({ theme: 'light' });

ReactiveUtils.autoSave(settings, 'settings', {
  autoLoad: true
});
// settings.theme automatically loaded from storage
```

**Auto-Load Disabled**
```javascript
const data = ReactiveUtils.reactive({ items: [] });

ReactiveUtils.autoSave(data, 'data', {
  autoLoad: false
});
// data.items stays [], must call ReactiveUtils.load(data) manually
```

**Conditional Loading**
```javascript
const cache = ReactiveUtils.reactive({ data: null });

ReactiveUtils.autoSave(cache, 'cache', {
  autoLoad: false
});

if (shouldUseCache()) {
  ReactiveUtils.load(cache);
}
```

### **When to Use**
- **`true`**: Most scenarios (user prefs, settings, cache)
- **`false`**: Conditional loading, manual control, migrations

---

## **Option 5: `autoSave`**

### **Description**
Automatically save to storage when state changes.

### **Type**
`boolean`

### **Default**
`true`

### **Syntax**
```javascript
ReactiveUtils.autoSave(state, 'key', {
  autoSave: true
});
```

### **Examples**

**Auto-Save Enabled (Default)**
```javascript
const prefs = ReactiveUtils.reactive({ theme: 'dark' });

ReactiveUtils.autoSave(prefs, 'prefs', {
  autoSave: true
});

prefs.theme = 'light';
// Automatically saved to storage
```

**Auto-Save Disabled**
```javascript
const form = ReactiveUtils.reactive({ data: {} });

ReactiveUtils.autoSave(form, 'form', {
  autoSave: false
});

form.data.title = 'New Title';
// NOT saved automatically

// Save manually when ready
ReactiveUtils.save(form);
```

**Toggle Auto-Save**
```javascript
const state = ReactiveUtils.reactive({ value: 0 });

ReactiveUtils.autoSave(state, 'state', {
  autoSave: true
});

// Disable auto-save temporarily
ReactiveUtils.stopAutoSave(state);
state.value = 100;  // Not saved

// Re-enable
ReactiveUtils.startAutoSave(state);
state.value = 200;  // Saved
```

### **When to Use**
- **`true`**: User preferences, settings, most scenarios
- **`false`**: Manual save buttons, batch operations, forms with submit

---

## **Option 6: `sync`**

### **Description**
Enable cross-tab/window synchronization via storage events.

### **Type**
`boolean`

### **Default**
`false`

### **Syntax**
```javascript
ReactiveUtils.autoSave(state, 'key', {
  sync: true
});
```

### **Examples**

**Cross-Tab Sync Enabled**
```javascript
const settings = ReactiveUtils.reactive({ theme: 'dark' });

ReactiveUtils.autoSave(settings, 'settings', {
  sync: true
});

// Tab 1: Change theme
settings.theme = 'light';

// Tab 2: Automatically updates to 'light'
```

**Messaging Between Tabs**
```javascript
const messages = ReactiveUtils.reactive({ inbox: [] });

ReactiveUtils.autoSave(messages, 'messages', {
  sync: true,
  onSync: (newValue) => {
    console.log('New messages from another tab!');
    showNotification('New messages received');
  }
});
```

**Shared Shopping Cart**
```javascript
const cart = ReactiveUtils.reactive({ items: [] });

ReactiveUtils.autoSave(cart, 'cart', {
  sync: true
});

// Changes in one tab reflect in all tabs
```

### **When to Use**
- **Enable**: Multi-tab apps, shared state, messaging
- **Disable**: Independent tabs, performance-critical, single-tab apps

### **Important Notes**
- Only works with `localStorage` (not sessionStorage)
- Sync events fire in OTHER tabs, not the originating tab
- Can cause sync loops if not careful

---

## **Option 7: `expires`**

### **Description**
Expiration time in seconds. Data automatically deleted after this time.

### **Type**
`number` or `null`

### **Default**
`null` (no expiration)

### **Syntax**
```javascript
ReactiveUtils.autoSave(state, 'key', {
  expires: 3600  // 1 hour
});
```

### **Examples**

**1 Hour Expiration**
```javascript
const cache = ReactiveUtils.reactive({ data: null });

ReactiveUtils.autoSave(cache, 'apiCache', {
  expires: 3600  // Expires in 1 hour
});

// After 1 hour, ReactiveUtils.load(cache) returns false
```

**Session Token (30 minutes)**
```javascript
const auth = ReactiveUtils.reactive({ token: null });

ReactiveUtils.autoSave(auth, 'session', {
  expires: 1800  // 30 minutes
});
```

**Common Durations**
```javascript
// 5 minutes
expires: 300

// 1 hour
expires: 3600

// 1 day
expires: 86400

// 1 week
expires: 604800

// No expiration
expires: null
```

### **When to Use**
- Cache with TTL
- Session tokens
- Temporary data
- Time-sensitive information

### **Behavior**
- Data stored with timestamp
- Load checks expiration
- Expired data automatically deleted
- Returns `null` on load if expired

---

## **Option 8: `onSave`**

### **Description**
Callback to transform/validate data before saving to storage.

### **Type**
`function(value) => transformedValue` or `null`

### **Default**
`null`

### **Signature**
```typescript
(value: any) => any
```

### **Syntax**
```javascript
ReactiveUtils.autoSave(state, 'key', {
  onSave: (value) => {
    // Transform value before saving
    return transformedValue;
  }
});
```

### **Examples**

**Remove Sensitive Data**
```javascript
const user = ReactiveUtils.reactive({
  name: 'John',
  email: 'john@example.com',
  password: 'secret123'
});

ReactiveUtils.autoSave(user, 'user', {
  onSave: (value) => {
    // Don't save password
    const { password, ...safe } = value;
    return safe;
  }
});
```

**Add Metadata**
```javascript
ReactiveUtils.autoSave(state, 'data', {
  onSave: (value) => {
    return {
      ...value,
      savedAt: Date.now(),
      version: '1.0.0'
    };
  }
});
```

**Compress Data**
```javascript
ReactiveUtils.autoSave(state, 'largeData', {
  onSave: (value) => {
    return compressData(value);
  },
  onLoad: (value) => {
    return decompressData(value);
  }
});
```

**Validation**
```javascript
ReactiveUtils.autoSave(form, 'form', {
  onSave: (value) => {
    if (!value.email || !value.email.includes('@')) {
      console.error('Invalid email');
      return null;  // Don't save
    }
    return value;
  }
});
```

### **When to Use**
- Remove sensitive fields
- Add metadata
- Validate before save
- Transform/normalize data
- Compress large data

---

## **Option 9: `onLoad`**

### **Description**
Callback to transform/validate data after loading from storage.

### **Type**
`function(value) => transformedValue` or `null`

### **Default**
`null`

### **Signature**
```typescript
(value: any) => any
```

### **Syntax**
```javascript
ReactiveUtils.autoSave(state, 'key', {
  onLoad: (value) => {
    // Transform value after loading
    return transformedValue;
  }
});
```

### **Examples**

**Data Migration**
```javascript
ReactiveUtils.autoSave(state, 'config', {
  onLoad: (value) => {
    // Migrate old format to new
    if (value.version === '1.0') {
      return {
        ...value,
        newField: 'default',
        version: '2.0'
      };
    }
    return value;
  }
});
```

**Add Default Values**
```javascript
ReactiveUtils.autoSave(settings, 'settings', {
  onLoad: (value) => {
    return {
      theme: 'light',
      fontSize: 14,
      ...value  // Override with loaded values
    };
  }
});
```

**Parse Dates**
```javascript
ReactiveUtils.autoSave(state, 'events', {
  onSave: (value) => {
    // Convert Date to ISO string
    return {
      ...value,
      createdAt: value.createdAt.toISOString()
    };
  },
  onLoad: (value) => {
    // Convert ISO string back to Date
    return {
      ...value,
      createdAt: new Date(value.createdAt)
    };
  }
});
```

**Validation**
```javascript
ReactiveUtils.autoSave(state, 'data', {
  onLoad: (value) => {
    if (!isValid(value)) {
      console.warn('Invalid data, using defaults');
      return getDefaults();
    }
    return value;
  }
});
```

### **When to Use**
- Data migration
- Add default values
- Parse special types (Date, etc.)
- Validate loaded data
- Decompress data

---

## **Option 10: `onSync`**

### **Description**
Callback fired when data syncs from another tab/window.

### **Type**
`function(newValue) => void` or `null`

### **Default**
`null`

### **Signature**
```typescript
(newValue: any) => void
```

### **Syntax**
```javascript
ReactiveUtils.autoSave(state, 'key', {
  sync: true,
  onSync: (newValue) => {
    // Handle sync from other tab
  }
});
```

### **Examples**

**Sync Notification**
```javascript
ReactiveUtils.autoSave(state, 'data', {
  sync: true,
  onSync: (newValue) => {
    console.log('Data synced from another tab');
    showNotification('Data updated');
  }
});
```

**Refresh UI**
```javascript
const messages = ReactiveUtils.reactive({ inbox: [] });

ReactiveUtils.autoSave(messages, 'messages', {
  sync: true,
  onSync: (newValue) => {
    playNotificationSound();
    highlightNewMessages(newValue.inbox);
  }
});
```

**Log Sync Events**
```javascript
ReactiveUtils.autoSave(state, 'state', {
  sync: true,
  onSync: (newValue) => {
    console.log('Sync event:', {
      timestamp: Date.now(),
      value: newValue
    });
  }
});
```

### **When to Use**
- Show sync notifications
- Refresh UI on sync
- Log sync events
- Play sounds/animations
- Track multi-tab usage

### **Important Notes**
- Only fires in OTHER tabs (not originating tab)
- Requires `sync: true`
- Only works with localStorage

---

## **Option 11: `onError`**

### **Description**
Callback for handling errors during save/load/sync operations.

### **Type**
`function(error, type) => void` or `null`

### **Default**
`null`

### **Signature**
```typescript
(error: Error, type: string) => void
```

**Error Types:**
- `'save'` - Save operation failed
- `'load'` - Load operation failed
- `'sync'` - Sync operation failed
- `'quota'` - Storage quota exceeded
- `'getValue'` - Get value failed
- `'setValue'` - Set value failed

### **Syntax**
```javascript
ReactiveUtils.autoSave(state, 'key', {
  onError: (error, type) => {
    // Handle error
  }
});
```

### **Examples**

**Basic Error Handling**
```javascript
ReactiveUtils.autoSave(state, 'data', {
  onError: (error, type) => {
    console.error(`Storage error (${type}):`, error.message);
  }
});
```

**Quota Exceeded**
```javascript
ReactiveUtils.autoSave(state, 'largeData', {
  onError: (error, type) => {
    if (type === 'quota') {
      alert('Storage full! Please clear some data.');
      showStorageManager();
    } else {
      console.error('Error:', error);
    }
  }
});
```

**Error Logging**
```javascript
ReactiveUtils.autoSave(state, 'critical', {
  onError: (error, type) => {
    // Log to error tracking service
    logToSentry({
      error: error,
      context: type,
      state: state
    });
  }
});
```

**Fallback Behavior**
```javascript
ReactiveUtils.autoSave(state, 'data', {
  onError: (error, type) => {
    if (type === 'load') {
      console.warn('Load failed, using defaults');
      Object.assign(state, getDefaults());
    }
  }
});
```

**User Notification**
```javascript
ReactiveUtils.autoSave(form, 'draft', {
  onError: (error, type) => {
    if (type === 'save') {
      showErrorNotification('Failed to save draft');
    }
  }
});
```

### **When to Use**
- Error logging
- User notifications
- Fallback logic
- Debug information
- Quota management

---

## **Complete Example**

```javascript
const appState = ReactiveUtils.reactive({
  user: null,
  preferences: {
    theme: 'light',
    fontSize: 14
  },
  cache: []
});

ReactiveUtils.autoSave(appState, 'appState', {
  // Storage configuration
  storage: 'localStorage',
  namespace: 'myApp',

  // Performance
  debounce: 1000,

  // Behavior
  autoLoad: true,
  autoSave: true,
  sync: true,

  // Expiration
  expires: 86400,  // 24 hours

  // Transformation
  onSave: (value) => {
    // Remove cache before saving
    const { cache, ...rest } = value;
    return {
      ...rest,
      savedAt: Date.now()
    };
  },

  onLoad: (value) => {
    // Add defaults if missing
    return {
      cache: [],
      ...value
    };
  },

  // Events
  onSync: (newValue) => {
    console.log('Synced from another tab');
    showNotification('App state updated');
  },

  onError: (error, type) => {
    console.error(`Storage error (${type}):`, error.message);

    if (type === 'quota') {
      alert('Storage full!');
      clearOldData();
    }
  }
});
```

---

## **Summary**

The `autoSave()` function accepts an options object with 11 configuration options that control storage behavior, performance, data transformation, and event handling. Use `storage` to choose between localStorage and sessionStorage, `namespace` to organize keys and prevent collisions, `debounce` to optimize performance during rapid changes, `autoLoad` and `autoSave` to control automatic behavior, `sync` for cross-tab synchronization, `expires` for time-based data expiration, `onSave` and `onLoad` for data transformation and validation, `onSync` to respond to cross-tab updates, and `onError` for robust error handling. These options provide fine-grained control over how reactive state is persisted to browser storage, making it easy to implement user preferences, form auto-save, caching, cross-tab communication, and more with just a few configuration options.

---

## **Advanced: autoSave with Global Shortcuts**

Combining `autoSave()` with global DOM shortcuts creates powerful reactive applications.

### **Example: Theme System with ClassName**
```javascript
const appState = ReactiveUtils.state({
  theme: 'light',
  accentColor: 'blue',
  fontSize: 14
});

// Auto-save with persistence
ReactiveUtils.autoSave(appState, 'appState', {
  storage: 'localStorage',
  namespace: 'myApp'
});

// Reactive UI - automatically syncs on ANY state change
ReactiveUtils.effect(() => {
  // Update ALL themed elements at once
  ClassName('themed').updateAll(el => {
    el.dataset.theme = appState.theme;
    el.dataset.accent = appState.accentColor;
    el.style.fontSize = appState.fontSize + 'px';
  });
});

// Change state - saves AND updates UI
appState.theme = 'dark';  // Saves to localStorage + UI updates
appState.fontSize = 16;   // Saves to localStorage + UI updates
```

**Benefits:**
- State changes auto-save to storage
- UI auto-updates via effect
- `ClassName()` updates all matching elements
- Zero manual DOM manipulation

### **Example: Form Auto-Save with Name**
```javascript
const formState = ReactiveUtils.state({
  title: '',
  content: '',
  tags: [],
  published: false
});

// Auto-save form drafts
ReactiveUtils.autoSave(formState, 'postDraft', {
  storage: 'sessionStorage',
  debounce: 500,  // Save 500ms after last change
  onSave: (value) => {
    console.log('Draft saved:', new Date().toLocaleTimeString());
    return value;
  }
});

// Bind form fields to state
querySelectorAll('[name]').forEach(field => {
  field.addEventListener('input', () => {
    const name = field.name;
    const value = field.type === 'checkbox' ? field.checked : field.value;
    formState[name] = value;
    // Automatically saves after 500ms (debounced)
  });
});

// Restore from storage on load
ReactiveUtils.effect(() => {
  Object.entries(formState).forEach(([name, value]) => {
    Name(name).updateAll(field => {
      if (field.type === 'checkbox') {
        field.checked = value;
      } else {
        field.value = value;
      }
    });
  });
});
```

**Why this scales:**
- Works with ANY form size
- `Name()` handles multiple fields (radios, checkboxes)
- Debounced saves prevent excessive writes
- Auto-restores on page load

### **Example: Dashboard with tagName**
```javascript
const dashboard = ReactiveUtils.state({
  metrics: {
    users: 0,
    posts: 0,
    revenue: 0
  },
  lastUpdate: null
});

// Persist dashboard state
ReactiveUtils.autoSave(dashboard, 'dashboardState', {
  storage: 'localStorage',
  namespace: 'analytics',
  onSave: (value) => {
    value.lastUpdate = Date.now();
    return value;
  }
});

// Update all metric outputs
ReactiveUtils.effect(() => {
  tagName('output').updateAll(el => {
    const metric = el.dataset.metric;
    if (metric && dashboard.metrics[metric] !== undefined) {
      el.textContent = dashboard.metrics[metric].toLocaleString();
    }
  });
});

// Update metrics - saves automatically + UI updates
dashboard.metrics.users = 1234;
dashboard.metrics.posts = 5678;
// All <output> elements update + state saves to localStorage
```

### **Example: Settings Panel with querySelectorAll**
```javascript
const settings = ReactiveUtils.state({
  notifications: true,
  autoSave: true,
  darkMode: false,
  language: 'en'
});

// Persist settings
ReactiveUtils.autoSave(settings, 'userSettings', {
  storage: 'localStorage',
  immediate: true  // Apply saved settings on load
});

// Sync all setting controls
ReactiveUtils.effect(() => {
  querySelectorAll('[data-setting]').forEach(control => {
    const setting = control.dataset.setting;
    const value = settings[setting];

    if (control.type === 'checkbox') {
      control.checked = value;
    } else if (control.tagName === 'SELECT') {
      control.value = value;
    } else {
      control.value = value;
    }
  });

  // Apply settings to UI
  ClassName('app-container').updateAll(el => {
    el.classList.toggle('dark-mode', settings.darkMode);
  });
});

// Update on user interaction
querySelectorAll('[data-setting]').forEach(control => {
  control.addEventListener('change', () => {
    const setting = control.dataset.setting;
    const value = control.type === 'checkbox' ? control.checked : control.value;
    settings[setting] = value;
    // Auto-saves + triggers effect + UI updates
  });
});
```

**The complete flow:**
1. User changes setting control
2. State updates
3. `autoSave()` persists to localStorage
4. Effect runs
5. `querySelectorAll()` updates all controls
6. `ClassName()` applies theme changes
7. Everything stays synchronized

