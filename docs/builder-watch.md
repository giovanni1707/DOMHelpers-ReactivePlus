# `builder.watch(defs)` - Add Watchers

**Quick Start (30 seconds)**
```javascript
const app = ReactiveUtils.reactive({
  count: 0,
  name: 'John',
  status: 'idle'
})
.watch({
  count: (newVal, oldVal) => {
    console.log(`Count changed: ${oldVal} -> ${newVal}`);
  },
  name: (newVal, oldVal) => {
    console.log(`Name changed: ${oldVal} -> ${newVal}`);
  }
})
.build();

app.count = 5; // Logs: "Count changed: 0 -> 5"
app.name = 'Jane'; // Logs: "Name changed: John -> Jane"
```

---

## **What is `builder.watch(defs)`?**

`builder.watch(defs)` is a **builder method** that adds watchers to reactive state properties, executing callbacks whenever watched properties change.

**Key characteristics:**
- **Property Watching**: Watch specific properties for changes
- **Old & New Values**: Callback receives both old and new values
- **Multiple Watchers**: Add multiple watchers at once
- **Chainable**: Returns builder for method chaining
- **Cleanup**: Watchers automatically cleaned up on destroy

---

## **Syntax**

```javascript
builder.watch(definitions)
```

### **Parameters**
- **`definitions`** (Object): Object mapping property names to callback functions

**Callback signature:**
```javascript
(newValue, oldValue) => {
  // React to change
}
```

### **Returns**
- **Type**: Builder instance (for chaining)

---

## **How it works**

```javascript
builder.watch(defs) {
  Object.entries(defs).forEach(([key, callback]) => {
    cleanups.push(addWatch(state, key, callback));
  });
  return this;
}
```

**What happens:**
1. Iterates through watcher definitions
2. Adds watch for each property
3. Stores cleanup functions
4. Returns builder for chaining
5. Callbacks execute when property changes

---

## **Examples**

### **Example 1: Basic Property Watch**
```javascript
const app = ReactiveUtils.reactive({ count: 0 })
  .watch({
    count: (newVal, oldVal) => {
      console.log(`Count: ${oldVal} -> ${newVal}`);
      console.log(`Increased by: ${newVal - oldVal}`);
    }
  })
  .build();

app.count = 5; // Logs: "Count: 0 -> 5", "Increased by: 5"
app.count = 10; // Logs: "Count: 5 -> 10", "Increased by: 5"
```

### **Example 2: Multiple Watchers**
```javascript
const user = ReactiveUtils.reactive({
  name: 'John',
  email: 'john@example.com',
  age: 25
})
.watch({
  name: (newVal, oldVal) => {
    console.log(`Name updated: ${oldVal} -> ${newVal}`);
  },
  email: (newVal, oldVal) => {
    console.log(`Email updated: ${oldVal} -> ${newVal}`);
  },
  age: (newVal, oldVal) => {
    if (newVal >= 18 && oldVal < 18) {
      console.log('User is now an adult!');
    }
  }
})
.build();

user.name = 'Jane';
user.email = 'jane@example.com';
user.age = 18;
```

### **Example 3: Validation on Change**
```javascript
const form = ReactiveUtils.reactive({
  email: '',
  emailError: ''
})
.watch({
  email: (newVal) => {
    if (!newVal.includes('@')) {
      form.emailError = 'Invalid email format';
    } else {
      form.emailError = '';
    }
  }
})
.build();

form.email = 'invalid'; // Sets emailError
form.email = 'valid@example.com'; // Clears emailError
```

### **Example 4: Side Effects**
```javascript
const settings = ReactiveUtils.reactive({
  theme: 'light',
  fontSize: 14,
  language: 'en'
})
.watch({
  theme: (newVal) => {
    document.body.classList.remove('light', 'dark');
    document.body.classList.add(newVal);
    localStorage.setItem('theme', newVal);
  },
  fontSize: (newVal) => {
    document.body.style.fontSize = `${newVal}px`;
  },
  language: (newVal) => {
    localStorage.setItem('language', newVal);
    // Reload translations...
  }
})
.build();

settings.theme = 'dark'; // Updates DOM and localStorage
```

### **Example 5: Logging Changes**
```javascript
const debugLog = [];

const app = ReactiveUtils.reactive({
  x: 0,
  y: 0,
  status: 'ready'
})
.watch({
  x: (newVal, oldVal) => {
    debugLog.push({
      prop: 'x',
      from: oldVal,
      to: newVal,
      timestamp: Date.now()
    });
  },
  y: (newVal, oldVal) => {
    debugLog.push({
      prop: 'y',
      from: oldVal,
      to: newVal,
      timestamp: Date.now()
    });
  }
})
.build();

app.x = 10;
app.y = 20;

console.log(debugLog);
// [
//   { prop: 'x', from: 0, to: 10, timestamp: ... },
//   { prop: 'y', from: 0, to: 20, timestamp: ... }
// ]
```

### **Example 6: Conditional Actions**
```javascript
const game = ReactiveUtils.reactive({
  score: 0,
  level: 1,
  lives: 3
})
.watch({
  score: (newVal, oldVal) => {
    // Level up every 100 points
    if (Math.floor(newVal / 100) > Math.floor(oldVal / 100)) {
      game.level++;
      console.log(`Level up! Now at level ${game.level}`);
    }
  },
  lives: (newVal) => {
    if (newVal === 0) {
      console.log('Game Over!');
      game.score = 0;
      game.level = 1;
      game.lives = 3;
    }
  }
})
.build();

game.score = 150; // Triggers level up
```

### **Example 7: API Sync**
```javascript
const userProfile = ReactiveUtils.reactive({
  name: 'John',
  bio: '',
  avatar: ''
})
.watch({
  name: async (newVal) => {
    await fetch('/api/user/update', {
      method: 'PATCH',
      body: JSON.stringify({ name: newVal })
    });
    console.log('Name synced to server');
  },
  bio: async (newVal) => {
    await fetch('/api/user/update', {
      method: 'PATCH',
      body: JSON.stringify({ bio: newVal })
    });
    console.log('Bio synced to server');
  }
})
.build();

userProfile.name = 'Jane'; // Syncs to API
```

### **Example 8: History Tracking**
```javascript
const history = [];

const editor = ReactiveUtils.reactive({
  content: '',
  title: ''
})
.watch({
  content: (newVal, oldVal) => {
    if (oldVal !== '') {
      history.push({
        type: 'content',
        value: oldVal,
        timestamp: Date.now()
      });
    }
  }
})
.build();

function undo() {
  const last = history.pop();
  if (last && last.type === 'content') {
    editor.content = last.value;
  }
}
```

### **Example 9: Debounced Watch**
```javascript
let searchTimeout;

const search = ReactiveUtils.reactive({
  query: '',
  results: []
})
.watch({
  query: (newVal) => {
    clearTimeout(searchTimeout);

    searchTimeout = setTimeout(async () => {
      if (newVal.length > 2) {
        const response = await fetch(`/api/search?q=${newVal}`);
        search.results = await response.json();
        console.log(`Found ${search.results.length} results`);
      }
    }, 300);
  }
})
.build();

search.query = 'reactive'; // Searches after 300ms delay
```

### **Example 10: Cascading Updates**
```javascript
const calculator = ReactiveUtils.reactive({
  basePrice: 100,
  quantity: 1,
  discount: 0,
  subtotal: 100,
  total: 100
})
.watch({
  basePrice: () => {
    calculator.subtotal = calculator.basePrice * calculator.quantity;
  },
  quantity: () => {
    calculator.subtotal = calculator.basePrice * calculator.quantity;
  },
  subtotal: () => {
    calculator.total = calculator.subtotal * (1 - calculator.discount);
  },
  discount: () => {
    calculator.total = calculator.subtotal * (1 - calculator.discount);
  }
})
.build();

calculator.quantity = 3; // Updates subtotal, then total
calculator.discount = 0.1; // Updates total
```

---

## **Common Patterns**

### **Pattern 1: Simple Watch**
```javascript
.watch({
  count: (newVal, oldVal) => {
    console.log(`Changed: ${oldVal} -> ${newVal}`);
  }
})
```

### **Pattern 2: Multiple Watches**
```javascript
.watch({
  prop1: (newVal) => { /* ... */ },
  prop2: (newVal) => { /* ... */ }
})
```

### **Pattern 3: Validation**
```javascript
.watch({
  email: (newVal) => {
    state.emailError = validateEmail(newVal);
  }
})
```

### **Pattern 4: Persistence**
```javascript
.watch({
  data: (newVal) => {
    localStorage.setItem('data', JSON.stringify(newVal));
  }
})
```

### **Pattern 5: Side Effects**
```javascript
.watch({
  theme: (newVal) => {
    document.body.className = newVal;
  }
})
```

---

## **Watch vs Effect**

| Feature | watch() | effect() |
|---------|---------|----------|
| Target | Specific property | Any accessed property |
| Trigger | When property changes | When any dependency changes |
| Old Value | ✓ Yes | ✗ No |
| Selective | ✓ Yes | ✗ No (watches all accessed) |
| Use Case | Respond to specific changes | Auto-sync derived state |

```javascript
// Watch - specific property
.watch({
  count: (newVal, oldVal) => {
    // Only when count changes
  }
})

// Effect - auto-track
.effect(() => {
  // When any accessed property changes
  console.log(state.count, state.name);
})
```

---

## **Best Practices**

1. **Keep watchers simple**
   ```javascript
   .watch({
     data: (newVal) => {
       updateUI(newVal);
     }
   })
   ```

2. **Avoid infinite loops**
   ```javascript
   // Bad - infinite loop
   .watch({
     count: (newVal) => {
       state.count = newVal + 1; // Triggers watch again!
     }
   })
   ```

3. **Use for side effects**
   ```javascript
   .watch({
     user: (newVal) => {
       localStorage.setItem('user', JSON.stringify(newVal));
     }
   })
   ```

4. **Debounce expensive operations**
   ```javascript
   let timeout;
   .watch({
     query: (newVal) => {
       clearTimeout(timeout);
       timeout = setTimeout(() => search(newVal), 300);
     }
   })
   ```

5. **Clean up in watchers if needed**
   ```javascript
   let subscription;
   .watch({
     topic: (newVal, oldVal) => {
       if (subscription) subscription.unsubscribe();
       subscription = subscribe(newVal);
     }
   })
   ```

---

## **Key Takeaways**

1. **Property-Specific**: Watch specific properties
2. **Old & New**: Callback gets both values
3. **Multiple**: Add many watchers at once
4. **Chainable**: Returns builder for chaining
5. **Side Effects**: Perfect for side effects
6. **Auto-Cleanup**: Cleaned up on destroy
7. **Async Support**: Callbacks can be async
8. **Validation**: Great for validation logic
9. **Persistence**: Sync to storage/APIs
10. **Cascading**: Can trigger other updates

---

## **Summary**

`builder.watch(defs)` is a builder method that adds watchers to specific reactive properties, executing callbacks whenever those properties change. Each watcher callback receives the new and old values, enabling precise change detection and response logic. Watchers are ideal for side effects like validation, persistence, API synchronization, DOM updates, and logging. The method accepts an object mapping property names to callback functions, allowing multiple watchers to be added at once. Unlike effects which auto-track all accessed properties, watchers monitor specific properties and provide both old and new values. Watchers are automatically cleaned up when the component is destroyed. Use watchers when you need to respond to specific property changes with side effects, while keeping the callbacks simple and avoiding circular dependencies that could cause infinite loops.
