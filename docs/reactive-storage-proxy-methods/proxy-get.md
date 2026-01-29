# `proxy.get(key)` - Get Value from Reactive Storage

**Quick Start (30 seconds)**
```javascript
// Create reactive storage
const storage = ReactiveUtils.reactiveStorage('localStorage', 'myApp');

// Set values
storage.set('theme', 'dark');
storage.set('user', { name: 'John', age: 30 });

// Get values (tracks dependency)
const theme = storage.get('theme');
console.log(theme); // 'dark'

const user = storage.get('user');
console.log(user); // { name: 'John', age: 30 }

// Reactive effect automatically tracks get()
ReactiveUtils.effect(() => {
  const currentTheme = storage.get('theme');
  console.log('Theme:', currentTheme);
  // Effect re-runs when theme changes
});
```

---

## **What is `proxy.get(key)`?**

`proxy.get()` is a **method on the reactive storage proxy** returned by `reactiveStorage()` that retrieves a value from browser storage and **tracks the access as a reactive dependency** so that effects re-run when the value changes.

**Key characteristics:**
- **Retrieves Value**: Reads from localStorage or sessionStorage
- **Tracks Dependency**: Effects re-run when value changes
- **JSON Deserialization**: Automatically parses JSON
- **Expiration Check**: Returns null if data expired
- **Returns Value**: The stored value or null if not found
- **Reactive**: Creates dependency in tracking contexts
- **Cross-Tab**: Responds to changes from other tabs

---

## **Syntax**

```javascript
proxy.get(key)
```

### **Parameters**
- **`key`** (string): Storage key (without namespace prefix)

### **Returns**
- **Type**: `any` or `null`
- Returns the stored value if found and not expired
- Returns `null` if key doesn't exist or data expired

---

## **How it works**

```javascript
// When you call proxy.get() inside an effect
const storage = ReactiveUtils.reactiveStorage('localStorage', 'app');

ReactiveUtils.effect(() => {
  const user = storage.get('user');
  console.log(user);
});

// Internally:
// 1. Proxy intercepts get() call
// 2. Tracks reactive dependency (_version)
const _ = reactiveState._version;

// 3. Retrieves from storage
const fullKey = namespace ? `${namespace}:${key}` : key;
const item = localStorage.getItem(fullKey);

// 4. Parses JSON
const data = JSON.parse(item);

// 5. Checks expiration
if (data.expires && Date.now() > data.expires) {
  localStorage.removeItem(fullKey);
  return null;
}

// 6. Returns value
return data.value;
```

**What happens:**
1. Proxy intercepts the get() call
2. Accesses reactive state _version (creates dependency)
3. Constructs full storage key with namespace
4. Retrieves item from browser storage
5. Parses JSON to object
6. Checks if data has expired
7. Removes expired data if necessary
8. Returns value or null
9. Effect now "subscribes" to changes to this key

---

## **Examples**

### **Example 1: Basic Get**
```javascript
const storage = ReactiveUtils.reactiveStorage('localStorage');

// Set values
storage.set('name', 'Alice');
storage.set('age', 30);
storage.set('active', true);

// Get values
console.log(storage.get('name'));   // 'Alice'
console.log(storage.get('age'));    // 30
console.log(storage.get('active')); // true

// Get non-existent key
console.log(storage.get('missing')); // null
```

### **Example 2: Reactive Effect**
```javascript
const prefs = ReactiveUtils.reactiveStorage('localStorage', 'prefs');

// Set initial theme
prefs.set('theme', 'light');

// Effect tracks get()
ReactiveUtils.effect(() => {
  const theme = prefs.get('theme');
  document.body.className = theme + '-theme';
  console.log('Theme applied:', theme);
});
// Logs: "Theme applied: light"

// Change triggers effect
prefs.set('theme', 'dark');
// Logs: "Theme applied: dark"
// Body class updated to 'dark-theme'
```

### **Example 3: Get with Fallback**
```javascript
const storage = ReactiveUtils.reactiveStorage('localStorage', 'app');

// Get with default value
const count = storage.get('count') || 0;
const items = storage.get('items') || [];
const config = storage.get('config') || { debug: false };

console.log(count);  // 0 if not found
console.log(items);  // [] if not found
console.log(config); // { debug: false } if not found
```

### **Example 4: Complex Objects**
```javascript
const storage = ReactiveUtils.reactiveStorage('localStorage', 'data');

// Store complex object
storage.set('user', {
  id: 123,
  name: 'John Doe',
  email: 'john@example.com',
  preferences: {
    theme: 'dark',
    notifications: true
  },
  tags: ['admin', 'verified']
});

// Retrieve complex object
const user = storage.get('user');
console.log(user.name);                    // 'John Doe'
console.log(user.preferences.theme);       // 'dark'
console.log(user.tags);                    // ['admin', 'verified']
```

### **Example 5: Expired Data**
```javascript
const cache = ReactiveUtils.reactiveStorage('localStorage', 'cache');

// Set with 5 second expiration
cache.set('tempData', { value: 'expires soon' }, { expires: 5 });

// Immediately available
console.log(cache.get('tempData')); // { value: 'expires soon' }

// After 6 seconds
setTimeout(() => {
  console.log(cache.get('tempData')); // null (expired)
}, 6000);
```

### **Example 6: Reactive Counter**
```javascript
const storage = ReactiveUtils.reactiveStorage('localStorage', 'app');

// Initialize counter
storage.set('clicks', 0);

// Display counter reactively
ReactiveUtils.effect(() => {
  const clicks = storage.get('clicks');
  querySelector('#counter').textContent = `Clicks: ${clicks}`;
});

// Increment on click
querySelector('#btn').addEventListener('click', () => {
  const current = storage.get('clicks');
  storage.set('clicks', current + 1);
  // Effect runs automatically, display updates
});
```

### **Example 7: User Preferences Panel**
```javascript
const prefs = ReactiveUtils.reactiveStorage('localStorage', 'preferences');

// Initialize defaults
if (!prefs.has('fontSize')) prefs.set('fontSize', 14);
if (!prefs.has('theme')) prefs.set('theme', 'light');
if (!prefs.has('language')) prefs.set('language', 'en');

// Apply preferences reactively
ReactiveUtils.effect(() => {
  const fontSize = prefs.get('fontSize');
  const theme = prefs.get('theme');
  const language = prefs.get('language');

  document.body.style.fontSize = fontSize + 'px';
  document.body.className = theme + '-theme';
  document.documentElement.lang = language;
});

// Changes apply immediately
prefs.set('fontSize', 16);
prefs.set('theme', 'dark');
```

### **Example 8: Shopping Cart**
```javascript
const cart = ReactiveUtils.reactiveStorage('localStorage', 'cart');

// Initialize empty cart
if (!cart.has('items')) cart.set('items', []);

// Reactive cart display
ReactiveUtils.effect(() => {
  const items = cart.get('items');
  const total = items.reduce((sum, item) => sum + item.price, 0);

  querySelector('#cartCount').textContent = items.length;
  querySelector('#cartTotal').textContent = `$${total.toFixed(2)}`;

  const html = items.map(item => `
    <div class="cart-item">
      <span>${item.name}</span>
      <span>$${item.price}</span>
    </div>
  `).join('');
  querySelector('#cartItems').innerHTML = html;
});

// Add item - display updates automatically
function addToCart(product) {
  const items = cart.get('items');
  items.push(product);
  cart.set('items', items);
}
```

### **Example 9: Cross-Tab Messaging**
```javascript
const messages = ReactiveUtils.reactiveStorage('localStorage', 'messages');

// Tab 1: Listen for messages
ReactiveUtils.effect(() => {
  const inbox = messages.get('inbox') || [];
  console.log('Received messages:', inbox);

  const html = inbox.map(msg => `
    <div class="message">
      <strong>${msg.from}:</strong> ${msg.text}
      <small>${new Date(msg.timestamp).toLocaleTimeString()}</small>
    </div>
  `).join('');
  querySelector('#messages').innerHTML = html;
});

// Tab 2: Send message
function sendMessage(text) {
  const inbox = messages.get('inbox') || [];
  inbox.push({
    from: 'Tab 2',
    text: text,
    timestamp: Date.now()
  });
  messages.set('inbox', inbox);
}

// Message sent in Tab 2 appears in Tab 1 instantly
```

### **Example 10: Form Auto-Load**
```javascript
const formData = ReactiveUtils.reactiveStorage('sessionStorage', 'formDraft');

// Load saved form data on page load
window.addEventListener('load', () => {
  const draft = formData.get('draft');

  if (draft) {
    // Restore form fields
    Object.entries(draft).forEach(([name, value]) => {
      const field = querySelector(`[name="${name}"]`);
      if (field) field.value = value;
    });

    console.log('Form draft restored');
  }
});

// Auto-save on input
querySelectorAll('input, textarea').forEach(field => {
  field.addEventListener('input', () => {
    const form = {};
    querySelectorAll('input, textarea').forEach(f => {
      form[f.name] = f.value;
    });
    formData.set('draft', form);
  });
});
```

---

## **Common Patterns**

### **Pattern 1: Simple Get**
```javascript
const value = storage.get('key');
```

### **Pattern 2: Get with Default**
```javascript
const value = storage.get('key') || defaultValue;
```

### **Pattern 3: Reactive Get**
```javascript
ReactiveUtils.effect(() => {
  const value = storage.get('key');
  updateUI(value);
});
```

### **Pattern 4: Check Before Get**
```javascript
if (storage.has('key')) {
  const value = storage.get('key');
  process(value);
}
```

### **Pattern 5: Get Multiple Keys**
```javascript
const user = storage.get('user');
const prefs = storage.get('prefs');
const session = storage.get('session');
```

---

## **When to Use**

| Scenario | Use proxy.get() |
|----------|-----------------|
| Load user preferences | ✓ Yes |
| Retrieve cached data | ✓ Yes |
| Reactive UI updates | ✓ Yes |
| Form data loading | ✓ Yes |
| Cross-tab data sync | ✓ Yes |
| Check for saved state | ✓ Yes |
| One-time read | ⚠ Consider native API |
| Non-reactive code | ⚠ Consider native API |

---

## **Reactivity Details**

### **Creates Dependency**
```javascript
ReactiveUtils.effect(() => {
  const value = storage.get('key');
  // Effect now depends on 'key'
  // Re-runs when 'key' changes
});
```

### **Multiple Gets**
```javascript
ReactiveUtils.effect(() => {
  const a = storage.get('a');
  const b = storage.get('b');
  // Effect depends on both 'a' and 'b'
  // Re-runs when either changes
});
```

### **Conditional Get**
```javascript
ReactiveUtils.effect(() => {
  if (condition) {
    const value = storage.get('key');
    // Dependency only tracked when condition is true
  }
});
```

---

## **Return Values**

### **Successful Get**
```javascript
storage.set('name', 'Alice');
const name = storage.get('name'); // 'Alice'
```

### **Key Not Found**
```javascript
const missing = storage.get('nonexistent'); // null
```

### **Expired Data**
```javascript
storage.set('temp', 'value', { expires: 1 });
// After 2 seconds
const expired = storage.get('temp'); // null
```

### **Complex Types**
```javascript
storage.set('obj', { a: 1, b: [2, 3] });
const obj = storage.get('obj'); // { a: 1, b: [2, 3] }

storage.set('arr', [1, 2, 3]);
const arr = storage.get('arr'); // [1, 2, 3]

storage.set('bool', true);
const bool = storage.get('bool'); // true

storage.set('num', 42);
const num = storage.get('num'); // 42
```

---

## **Best Practices**

1. **Provide defaults**
   ```javascript
   const count = storage.get('count') || 0;
   const items = storage.get('items') || [];
   ```

2. **Check existence first**
   ```javascript
   if (storage.has('key')) {
     const value = storage.get('key');
     process(value);
   }
   ```

3. **Use in effects for reactive UI**
   ```javascript
   ReactiveUtils.effect(() => {
     const theme = storage.get('theme');
     applyTheme(theme);
   });
   ```

4. **Validate retrieved data**
   ```javascript
   const user = storage.get('user');
   if (user && user.id && user.name) {
     displayUser(user);
   }
   ```

5. **Handle null gracefully**
   ```javascript
   const config = storage.get('config');
   if (!config) {
     config = getDefaultConfig();
     storage.set('config', config);
   }
   ```

6. **Don't mutate returned objects**
   ```javascript
   // Bad: mutates cached object
   const items = storage.get('items');
   items.push(newItem); // Doesn't trigger reactivity

   // Good: create new object
   const items = storage.get('items') || [];
   const updated = [...items, newItem];
   storage.set('items', updated);
   ```

---

## **vs. Native localStorage.getItem()**

| Feature | `proxy.get()` | `localStorage.getItem()` |
|---------|--------------|--------------------------|
| Tracks reactivity | ✓ Yes | ✗ No |
| JSON parsing | ✓ Automatic | ✗ Manual |
| Expiration check | ✓ Automatic | ✗ Manual |
| Namespace | ✓ Automatic | ✗ Manual |
| Returns null if missing | ✓ Yes | ✓ Yes |

```javascript
// proxy.get() - reactive, automatic
const storage = ReactiveUtils.reactiveStorage('localStorage');
const user = storage.get('user');
// Tracks dependency, parses JSON, checks expiration

// localStorage.getItem() - manual
const userStr = localStorage.getItem('user');
const user = userStr ? JSON.parse(userStr) : null;
// No reactivity, manual JSON parsing
```

---

## **Key Takeaways**

1. **Retrieves Value**: Reads from browser storage
2. **Tracks Dependency**: Creates reactive dependency
3. **JSON Automatic**: Parses JSON automatically
4. **Returns Value/Null**: Value if found, null if not
5. **Expiration Aware**: Checks and removes expired data
6. **Reactive**: Effects re-run on changes
7. **Cross-Tab**: Responds to other tabs
8. **Namespaced**: Handles namespace automatically
9. **Type Preserved**: Returns original JavaScript types
10. **Safe**: Never throws, returns null on error

---

## **Summary**

`proxy.get(key)` is a method on the reactive storage proxy returned by `reactiveStorage()` that retrieves a value from browser storage and tracks the access as a reactive dependency, causing effects and computed properties to automatically re-run when the stored value changes. When called, it constructs the namespaced storage key, retrieves the item from localStorage or sessionStorage, automatically parses the JSON, checks if the data has expired, and returns the value or null if not found or expired. The key feature is that when used inside an effect or computed property, get() creates a reactive dependency, so the effect automatically re-runs whenever that storage key is updated via set(). This makes it perfect for reactive UI updates, user preferences, cached data, cross-tab synchronization, and any scenario where UI components need to automatically reflect storage changes. Unlike native localStorage.getItem(), proxy.get() provides automatic JSON deserialization, expiration handling, namespace management, and most importantly, reactive dependency tracking that keeps your application state synchronized with storage.

---

## **Advanced: Reactive UI with Global Shortcuts**

Combining `proxy.get()` with global shortcuts creates powerful reactive patterns.

### **Example: Auto-Sync UI with ClassName**
```javascript
const prefs = ReactiveUtils.reactiveStorage('localStorage', 'prefs');

// Initialize preferences
prefs.set('darkMode', true);
prefs.set('compactView', false);
prefs.set('showSidebar', true);

// Reactive UI sync - runs on any preference change
ReactiveUtils.effect(() => {
  const darkMode = prefs.get('darkMode');
  const compactView = prefs.get('compactView');
  const showSidebar = prefs.get('showSidebar');

  // Update all layout containers
  ClassName('layout-container').updateAll(el => {
    el.classList.toggle('dark-mode', darkMode);
    el.classList.toggle('compact', compactView);
    el.classList.toggle('sidebar-visible', showSidebar);
  });

  // Update all preference displays
  ClassName('pref-display').updateAll(el => {
    const pref = el.dataset.pref;
    el.textContent = prefs.get(pref);
  });
});

// Toggle preference - entire UI updates automatically
prefs.set('darkMode', !prefs.get('darkMode'));
```

### **Example: Form Field Sync with Name**
```javascript
const userData = ReactiveUtils.reactiveStorage('localStorage', 'user');

// Load user data
userData.set('profile', {
  username: 'alice',
  email: 'alice@example.com',
  bio: 'Software developer'
});

// Sync all form fields reactively
ReactiveUtils.effect(() => {
  const profile = userData.get('profile');
  if (!profile) return;

  // Populate all fields by name
  Object.entries(profile).forEach(([fieldName, value]) => {
    Name(fieldName).updateAll(field => {
      field.value = value;
    });
  });
});

// Update profile - all fields update across all forms
const profile = userData.get('profile');
profile.username = 'alice_updated';
userData.set('profile', profile);
```

### **Example: Multi-Tab Reactive Updates**
```javascript
const notifications = ReactiveUtils.reactiveStorage('localStorage', 'notifications');

// Effect runs in ALL open tabs
ReactiveUtils.effect(() => {
  const unread = notifications.get('unread') || [];
  const count = unread.length;

  // Update all notification badges
  ClassName('notification-badge').updateAll(badge => {
    badge.textContent = count;
    badge.style.display = count > 0 ? 'inline-block' : 'none';
    badge.classList.toggle('has-notifications', count > 0);
  });

  // Update all notification lists
  ClassName('notification-list').updateAll(list => {
    list.innerHTML = unread.map(notif => `
      <li class="notification-item">
        <strong>${notif.title}</strong>
        <p>${notif.message}</p>
      </li>
    `).join('');
  });
});

// Add notification in Tab 1
const unread = notifications.get('unread') || [];
unread.push({ title: 'New Message', message: 'You have mail!' });
notifications.set('unread', unread);
// Tab 2 UI updates automatically!
```

**Why this is powerful:**
- Cross-tab reactivity built-in
- `ClassName()` updates all badges in all tabs
- Single source of truth (storage)
- No manual polling or postMessage needed

