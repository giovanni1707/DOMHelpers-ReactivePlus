# `builder.state` - Access Created Reactive State

**Quick Start (30 seconds)**
```javascript
const builder = ReactiveUtils.reactive({
  count: 0,
  name: 'Counter'
});

// Access state directly via builder.state
console.log(builder.state.count); // 0
console.log(builder.state.name); // 'Counter'

// Modify state
builder.state.count = 5;
console.log(builder.state.count); // 5

// Continue building
builder
  .computed({
    double: function() { return this.count * 2; }
  })
  .build();

// State is the same object
console.log(builder.state.double); // 10
```

---

## **What is `builder.state`?**

`builder.state` is a **property** that provides direct access to the reactive state object created by the builder, allowing you to read and modify state while still building the component.

**Key characteristics:**
- **Direct Access**: Access state before calling `build()`
- **Reactive**: State is fully reactive
- **Mutable**: Can modify state during build process
- **Same Reference**: Same object returned by `build()`
- **Always Available**: Available immediately after `reactive()` call

---

## **Syntax**

```javascript
const builder = ReactiveUtils.reactive(initialState);

// Access state
const value = builder.state.propertyName;

// Modify state
builder.state.propertyName = newValue;
```

### **Returns**
- **Type**: Reactive state object (Proxy)

---

## **How it works**

```javascript
function reactive(initialState) {
  const state = createReactive(initialState);
  const cleanups = [];

  const builder = {
    state,  // Direct reference to reactive state
    // ... other methods
  };

  return builder;
}
```

**What happens:**
1. `reactive()` creates reactive state
2. State stored as `builder.state` property
3. All builder methods operate on this state
4. `build()` returns the same state object

---

## **Examples**

### **Example 1: Basic State Access**
```javascript
const builder = ReactiveUtils.reactive({
  count: 0,
  message: 'Hello'
});

console.log(builder.state.count); // 0
console.log(builder.state.message); // 'Hello'

builder.state.count = 10;
console.log(builder.state.count); // 10
```

### **Example 2: Modify During Build**
```javascript
const builder = ReactiveUtils.reactive({ count: 0 });

// Modify state before build
builder.state.count = 5;

builder.computed({
  double: function() { return this.count * 2; }
});

const app = builder.build();

console.log(app.count); // 5
console.log(app.double); // 10
```

### **Example 3: Initialize from External Data**
```javascript
const userData = await fetch('/api/user').then(r => r.json());

const builder = ReactiveUtils.reactive({
  user: null,
  loading: false
});

// Set initial data
builder.state.user = userData;
builder.state.loading = false;

const app = builder
  .computed({
    userName: function() { return this.user?.name || 'Guest'; }
  })
  .build();

console.log(app.userName);
```

### **Example 4: Conditional Setup**
```javascript
const isDevelopment = process.env.NODE_ENV === 'development';

const builder = ReactiveUtils.reactive({
  debug: isDevelopment,
  version: '1.0.0'
});

if (builder.state.debug) {
  builder.effect(() => {
    console.log('Debug mode:', builder.state);
  });
}

const app = builder.build();
```

### **Example 5: State Validation**
```javascript
const builder = ReactiveUtils.reactive({
  email: '',
  age: 0
});

// Validate before building
function validateState() {
  const errors = [];

  if (!builder.state.email.includes('@')) {
    errors.push('Invalid email');
  }

  if (builder.state.age < 18) {
    errors.push('Age must be 18+');
  }

  return errors;
}

builder.state.email = 'user@example.com';
builder.state.age = 25;

const errors = validateState();
if (errors.length === 0) {
  const app = builder.build();
  console.log('App created successfully');
}
```

### **Example 6: Factory Pattern**
```javascript
function createUser(data) {
  const builder = ReactiveUtils.reactive({
    firstName: '',
    lastName: '',
    email: ''
  });

  // Initialize with provided data
  Object.assign(builder.state, data);

  return builder
    .computed({
      fullName: function() {
        return `${this.firstName} ${this.lastName}`;
      }
    })
    .build();
}

const user = createUser({
  firstName: 'John',
  lastName: 'Doe',
  email: 'john@example.com'
});

console.log(user.fullName); // 'John Doe'
```

### **Example 7: Progressive Enhancement**
```javascript
const builder = ReactiveUtils.reactive({ count: 0 });

// Add initial computed
builder.computed({
  double: function() { return this.count * 2; }
});

// Check state before adding more
if (builder.state.count < 100) {
  builder.computed({
    isLow: function() { return this.count < 10; }
  });
}

const app = builder.build();
```

### **Example 8: State Snapshot**
```javascript
const builder = ReactiveUtils.reactive({
  x: 10,
  y: 20,
  z: 30
});

// Take snapshot before building
const snapshot = { ...builder.state };

builder.state.x = 100;

const app = builder.build();

console.log('Original:', snapshot); // { x: 10, y: 20, z: 30 }
console.log('Current:', { x: app.x, y: app.y, z: app.z }); // { x: 100, y: 20, z: 30 }
```

### **Example 9: Debugging Helper**
```javascript
function debugBuilder(name, builder) {
  console.log(`[${name}] State:`, builder.state);
  console.log(`[${name}] Keys:`, Object.keys(builder.state));

  return builder;
}

const app = debugBuilder('MyApp',
  ReactiveUtils.reactive({ count: 0, name: 'App' })
    .computed({ double: function() { return this.count * 2; } })
)
.build();
```

### **Example 10: State Migration**
```javascript
const oldState = {
  counter: 5, // Old property name
  userName: 'John'
};

const builder = ReactiveUtils.reactive({
  count: 0, // New property name
  name: ''
});

// Migrate old state to new structure
builder.state.count = oldState.counter;
builder.state.name = oldState.userName;

const app = builder.build();

console.log(app.count); // 5
console.log(app.name); // 'John'
```

---

## **Common Patterns**

### **Pattern 1: Read State**
```javascript
const value = builder.state.propertyName;
```

### **Pattern 2: Write State**
```javascript
builder.state.propertyName = newValue;
```

### **Pattern 3: Bulk Update**
```javascript
Object.assign(builder.state, updates);
```

### **Pattern 4: Conditional Logic**
```javascript
if (builder.state.someCondition) {
  builder.computed({ ... });
}
```

### **Pattern 5: State Snapshot**
```javascript
const snapshot = { ...builder.state };
```

---

## **State Reference**

```javascript
const builder = ReactiveUtils.reactive({ count: 0 });

// All reference the same object
const stateRef1 = builder.state;
const app = builder.build();
const stateRef2 = app;

console.log(stateRef1 === app); // true
console.log(stateRef2 === builder.state); // true
```

---

## **When to Use**

| Scenario | Use builder.state |
|----------|-------------------|
| Initialize state before build | ✓ Yes |
| Modify state during setup | ✓ Yes |
| Conditional configuration | ✓ Yes |
| State validation | ✓ Yes |
| Debug builder state | ✓ Yes |
| State migration | ✓ Yes |
| Access computed before build | ✗ No (add computed first) |

---

## **vs. Built State**

```javascript
const builder = ReactiveUtils.reactive({ count: 0 });

// Before build
builder.state.count = 5;

// After build
const app = builder.build();
app.count = 10;

// Same object
console.log(builder.state === app); // true
console.log(builder.state.count); // 10
```

---

## **Best Practices**

1. **Use for initialization**
   ```javascript
   builder.state.data = await fetchData();
   ```

2. **Validate before building**
   ```javascript
   if (isValid(builder.state)) {
     const app = builder.build();
   }
   ```

3. **Don't rely on after build**
   ```javascript
   const app = builder.build();
   // Use app directly, not builder.state
   ```

4. **Snapshot if needed**
   ```javascript
   const initial = { ...builder.state };
   ```

5. **Clear pattern**
   ```javascript
   // Setup phase
   builder.state.count = 0;
   builder.computed({ ... });

   // Build phase
   const app = builder.build();
   ```

---

## **Key Takeaways**

1. **Direct Access**: Access reactive state before `build()`
2. **Reactive**: State is fully reactive with Proxy
3. **Same Reference**: `builder.state` === built state
4. **Mutable**: Can modify during build process
5. **Always Available**: Available immediately after `reactive()`
6. **Initialization**: Useful for setting initial values
7. **Validation**: Can validate before building
8. **Configuration**: Conditional setup based on state
9. **Migration**: Useful for state migration
10. **Debug**: Inspect state during build

---

## **Summary**

`builder.state` is a property that provides direct access to the reactive state object created by the builder, allowing you to read and modify state while constructing your component. The state is fully reactive and is the same object that will be returned by `build()`, meaning any changes made via `builder.state` will be reflected in the built component. This property is particularly useful for initializing state with external data, validating state before building, implementing conditional configuration based on current state values, and debugging during the build process. Use `builder.state` during the setup phase to prepare your component, then call `build()` to complete the construction. After building, work directly with the returned state object rather than `builder.state`, as they reference the same underlying Proxy object.
