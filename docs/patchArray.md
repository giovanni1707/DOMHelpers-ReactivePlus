# `patchArray(state, key)` - Manually Patch Array Property for Reactivity

**Quick Start (30 seconds)**
```javascript
// Create reactive state
const app = ReactiveUtils.state({
  items: [1, 2, 3]
});

// Array methods might not trigger reactivity if not auto-patched
app.items.push(4);  // May not trigger effects

// Manually patch the array to make it reactive
ReactiveUtils.patchArray(app, 'items');

// Now array methods trigger reactivity
ReactiveUtils.effect(() => {
  console.log('Items count:', app.items.length);
});
// Output: "Items count: 4"

app.items.push(5);
// Output: "Items count: 5" - effect runs automatically!

app.items.pop();
// Output: "Items count: 4" - effect runs again!
```

---

## **What is `patchArray(state, key)`?**

`patchArray()` is a **utility function** that manually patches an array property on a reactive state object to ensure array mutation methods (push, pop, splice, etc.) trigger reactivity and re-run effects.

**Key characteristics:**
- **Manual Patching**: Explicitly patches array for reactivity
- **Mutation Methods**: Patches push, pop, shift, unshift, splice, sort, reverse, fill, copyWithin
- **Usually Automatic**: Modern `state()` auto-patches arrays
- **Edge Cases**: Use when arrays added after state creation
- **Side-Effect**: Modifies array methods directly
- **Returns Nothing**: Void function (undefined)
- **One-Time**: Only needs to be called once per array

---

## **Syntax**

```javascript
ReactiveUtils.patchArray(state, key)
```

### **Parameters**

**`state`** (object, required)
- Reactive state object created with `state()` or `reactive()`

**`key`** (string, required)
- Property name of the array to patch

### **Returns**
- **Type**: `undefined`
- Side-effect function (modifies array in place)

---

## **How it works**

```javascript
// When you call patchArray()
const state = ReactiveUtils.state({ items: [1, 2, 3] });
ReactiveUtils.patchArray(state, 'items');

// Internally:
// 1. Gets the array
const arr = state.items;

// 2. Checks if already patched
if (arr.__patched) return;

// 3. Marks as patched
Object.defineProperty(arr, '__patched', {
  value: true,
  enumerable: false
});

// 4. Patches each mutation method
['push', 'pop', 'shift', 'unshift', 'splice',
 'sort', 'reverse', 'fill', 'copyWithin'].forEach(method => {
  const original = Array.prototype[method];

  Object.defineProperty(arr, method, {
    value: function(...args) {
      // Call original method
      const result = original.apply(this, args);

      // Trigger reactivity by reassigning array
      const updatedArray = [...this];
      state.items = updatedArray;

      return result;
    }
  });
});
```

**What happens:**
1. Retrieves array from state
2. Checks if already patched (avoids double-patching)
3. Marks array with `__patched` flag
4. Wraps each mutation method
5. Original method called first
6. Array reassigned to trigger reactivity
7. Returns original method result

---

## **Examples**

### **Example 1: Basic Patching**
```javascript
const state = ReactiveUtils.state({
  numbers: [1, 2, 3]
});

// Patch the array
ReactiveUtils.patchArray(state, 'numbers');

// Now mutations trigger effects
ReactiveUtils.effect(() => {
  console.log('Numbers:', state.numbers);
});
// Output: "Numbers: [1, 2, 3]"

state.numbers.push(4);
// Output: "Numbers: [1, 2, 3, 4]"

state.numbers.pop();
// Output: "Numbers: [1, 2, 3]"
```

### **Example 2: Dynamic Array Addition**
```javascript
const state = ReactiveUtils.state({
  name: 'App'
});

// Add array property after state creation
state.todos = [];

// Patch the dynamically added array
ReactiveUtils.patchArray(state, 'todos');

// Now it's reactive
ReactiveUtils.effect(() => {
  console.log(`${state.todos.length} todos`);
});
// Output: "0 todos"

state.todos.push({ title: 'Task 1', done: false });
// Output: "1 todos"
```

### **Example 3: Multiple Arrays**
```javascript
const state = ReactiveUtils.state({
  users: [],
  posts: [],
  comments: []
});

// Patch all arrays
ReactiveUtils.patchArray(state, 'users');
ReactiveUtils.patchArray(state, 'posts');
ReactiveUtils.patchArray(state, 'comments');

// All are now reactive
ReactiveUtils.effect(() => {
  console.log('Users:', state.users.length);
  console.log('Posts:', state.posts.length);
  console.log('Comments:', state.comments.length);
});

state.users.push({ name: 'Alice' });
state.posts.push({ title: 'Hello' });
state.comments.push({ text: 'Nice!' });
```

### **Example 4: Todo List**
```javascript
const app = ReactiveUtils.state({
  todos: []
});

// Ensure array is patched
ReactiveUtils.patchArray(app, 'todos');

// Reactive rendering
ReactiveUtils.effect(() => {
  const html = app.todos.map(todo => `
    <li>
      <input type="checkbox" ${todo.done ? 'checked' : ''}>
      ${todo.title}
    </li>
  `).join('');

  document.getElementById('todos').innerHTML = html;
});

// Add todo - UI updates automatically
function addTodo(title) {
  app.todos.push({ title, done: false });
}

// Remove todo - UI updates automatically
function removeTodo(index) {
  app.todos.splice(index, 1);
}

// Toggle todo - need to reassign for nested changes
function toggleTodo(index) {
  app.todos[index].done = !app.todos[index].done;
  app.todos = [...app.todos];  // Trigger reactivity
}
```

### **Example 5: Shopping Cart**
```javascript
const cart = ReactiveUtils.state({
  items: []
});

ReactiveUtils.patchArray(cart, 'items');

// Reactive cart display
ReactiveUtils.effect(() => {
  const total = cart.items.reduce((sum, item) => sum + item.price, 0);
  document.getElementById('cartTotal').textContent = `$${total.toFixed(2)}`;
  document.getElementById('cartCount').textContent = cart.items.length;
});

function addToCart(product) {
  cart.items.push({
    id: product.id,
    name: product.name,
    price: product.price
  });
  // Effect runs, UI updates
}

function removeFromCart(index) {
  cart.items.splice(index, 1);
  // Effect runs, UI updates
}

function clearCart() {
  cart.items.splice(0, cart.items.length);
  // or: cart.items = [];
  // Effect runs, UI updates
}
```

### **Example 6: All Mutation Methods**
```javascript
const state = ReactiveUtils.state({
  data: [1, 2, 3, 4, 5]
});

ReactiveUtils.patchArray(state, 'data');

ReactiveUtils.effect(() => {
  console.log('Data:', state.data.join(', '));
});

// All these trigger the effect:
state.data.push(6);          // Add to end
state.data.pop();            // Remove from end
state.data.shift();          // Remove from start
state.data.unshift(0);       // Add to start
state.data.splice(2, 1, 99); // Replace element
state.data.sort();           // Sort array
state.data.reverse();        // Reverse array
state.data.fill(0, 0, 2);    // Fill portion
state.data.copyWithin(0, 1); // Copy portion
```

### **Example 7: Conditional Patching**
```javascript
const state = ReactiveUtils.state({
  items: []
});

// Check if patching is needed
if (!state.items.__patched) {
  ReactiveUtils.patchArray(state, 'items');
  console.log('Array patched');
} else {
  console.log('Array already patched');
}
```

### **Example 8: Legacy Code Migration**
```javascript
// Old code using plain array
let items = [1, 2, 3];

// Migrate to reactive state
const state = ReactiveUtils.state({
  items: items
});

// Patch for reactivity
ReactiveUtils.patchArray(state, 'items');

// Now reactive
ReactiveUtils.effect(() => {
  updateUI(state.items);
});

// Old code still works
state.items.push(4);  // Triggers effect
```

### **Example 9: Lazy Loading**
```javascript
const state = ReactiveUtils.state({
  data: null
});

// Later: load data asynchronously
async function loadData() {
  const response = await fetch('/api/data');
  const items = await response.json();

  // Set array data
  state.data = items;

  // Patch for reactivity
  if (Array.isArray(state.data)) {
    ReactiveUtils.patchArray(state, 'data');
  }
}

ReactiveUtils.effect(() => {
  if (state.data) {
    console.log(`Loaded ${state.data.length} items`);
  }
});

loadData();
```

### **Example 10: Debug Helper**
```javascript
function debugPatchArray(state, key) {
  console.log(`Patching array: ${key}`);

  const arr = state[key];
  if (!Array.isArray(arr)) {
    console.error(`${key} is not an array`);
    return;
  }

  if (arr.__patched) {
    console.warn(`${key} already patched`);
    return;
  }

  ReactiveUtils.patchArray(state, key);
  console.log(`${key} patched successfully`);
}

const state = ReactiveUtils.state({ items: [] });
debugPatchArray(state, 'items');
// Output: "Patching array: items"
// Output: "items patched successfully"
```

---

## **Common Patterns**

### **Pattern 1: Immediate Patch**
```javascript
const state = ReactiveUtils.state({ items: [] });
ReactiveUtils.patchArray(state, 'items');
```

### **Pattern 2: Conditional Patch**
```javascript
if (!state.items.__patched) {
  ReactiveUtils.patchArray(state, 'items');
}
```

### **Pattern 3: Multiple Arrays**
```javascript
['users', 'posts', 'comments'].forEach(key => {
  ReactiveUtils.patchArray(state, key);
});
```

### **Pattern 4: Dynamic Addition**
```javascript
state.newArray = [];
ReactiveUtils.patchArray(state, 'newArray');
```

### **Pattern 5: Migration Helper**
```javascript
function makeReactive(plainObj) {
  const state = ReactiveUtils.state(plainObj);
  Object.keys(plainObj).forEach(key => {
    if (Array.isArray(plainObj[key])) {
      ReactiveUtils.patchArray(state, key);
    }
  });
  return state;
}
```

---

## **When to Use**

| Scenario | Use patchArray() |
|----------|------------------|
| Arrays added after state creation | ✓ Yes |
| Dynamically created arrays | ✓ Yes |
| Legacy code migration | ✓ Yes |
| Array methods not reactive | ✓ Yes |
| Debugging reactivity | ✓ Yes |
| Initial state arrays | ✗ No (auto-patched) |
| Using collection() | ✗ No (auto-patched) |
| Modern reactive systems | ⚠ Rarely needed |

---

## **Array Methods Patched**

The following Array.prototype methods are patched:

### **Add/Remove**
- `push(...items)` - Add to end
- `pop()` - Remove from end
- `shift()` - Remove from start
- `unshift(...items)` - Add to start
- `splice(start, deleteCount, ...items)` - Insert/remove at position

### **Transform**
- `sort([compareFunction])` - Sort in place
- `reverse()` - Reverse in place
- `fill(value, [start], [end])` - Fill with value
- `copyWithin(target, start, [end])` - Copy within array

### **Not Patched (Don't Mutate)**
These don't need patching because they don't mutate:
- `map()`, `filter()`, `reduce()`, `slice()`, `concat()`, etc.

---

## **Limitations**

### **Nested Objects Not Auto-Reactive**
```javascript
const state = ReactiveUtils.state({
  todos: [{ title: 'Task', done: false }]
});

ReactiveUtils.patchArray(state, 'todos');

// This doesn't trigger reactivity
state.todos[0].done = true;

// Need to reassign
state.todos[0].done = true;
state.todos = [...state.todos];  // Triggers reactivity
```

### **Direct Assignment Still Works**
```javascript
// Both trigger reactivity
state.items.push(4);      // Patched method
state.items = [...state.items, 4];  // Direct assignment
```

### **Already Patched Arrays**
```javascript
// Patching twice is safe but unnecessary
ReactiveUtils.patchArray(state, 'items');
ReactiveUtils.patchArray(state, 'items');  // No-op
```

---

## **Best Practices**

1. **Let auto-patching work first**
   ```javascript
   // Usually don't need manual patching
   const state = ReactiveUtils.state({ items: [] });
   // Arrays auto-patched in modern systems
   ```

2. **Patch after dynamic addition**
   ```javascript
   state.newArray = [];
   ReactiveUtils.patchArray(state, 'newArray');
   ```

3. **Check before patching**
   ```javascript
   if (!state.items.__patched) {
     ReactiveUtils.patchArray(state, 'items');
   }
   ```

4. **Patch all arrays in object**
   ```javascript
   Object.keys(state).forEach(key => {
     if (Array.isArray(state[key])) {
       ReactiveUtils.patchArray(state, key);
     }
   });
   ```

5. **Document why patching**
   ```javascript
   // Patch because array added after state creation
   state.items = [];
   ReactiveUtils.patchArray(state, 'items');
   ```

---

## **Aliases**

This function has multiple names:

```javascript
// All the same function
ReactiveUtils.patchArray(state, 'items');
patchReactiveArray(state, 'items');  // Global (legacy)
Collections.patchArray(state, 'items');  // Collections namespace
Elements.patchArray(state, 'items');  // Elements namespace
```

**Recommended**: Use `ReactiveUtils.patchArray()` for consistency.

---

## **Troubleshooting**

### **Array Methods Still Not Reactive**
```javascript
// Make sure you're using the right key
ReactiveUtils.patchArray(state, 'items');  // ✓ Correct
ReactiveUtils.patchArray(state, 'item');   // ✗ Wrong key

// Make sure state is reactive
const state = ReactiveUtils.state({ items: [] });  // ✓ Reactive
const state = { items: [] };  // ✗ Not reactive
```

### **Error: Invalid State or Key**
```javascript
// State must exist
ReactiveUtils.patchArray(null, 'items');  // ✗ Error

// Key must exist
ReactiveUtils.patchArray(state, 'missing');  // ✗ Error

// Must be an array
state.notArray = 'string';
ReactiveUtils.patchArray(state, 'notArray');  // ✗ Error
```

---

## **Key Takeaways**

1. **Manual Patching**: Explicitly patches array for reactivity
2. **Mutation Methods**: Patches push, pop, splice, sort, reverse, fill, copyWithin, shift, unshift
3. **Usually Automatic**: Modern systems auto-patch arrays
4. **Dynamic Arrays**: Use for arrays added after state creation
5. **One-Time**: Only needs to be called once per array
6. **Safe**: Checks for already-patched arrays
7. **Side-Effect**: Modifies array methods in place
8. **Returns Nothing**: Void function
9. **Legacy Compatible**: Works with old code
10. **Rarely Needed**: Most arrays auto-patch

---

## **Summary**

`patchArray(state, key)` is a utility function that manually patches an array property on a reactive state object to ensure array mutation methods (push, pop, splice, etc.) trigger reactivity and re-run effects. When called, it wraps each array mutation method to reassign the array after the operation, triggering reactive updates. This is typically handled automatically by modern reactive state systems, but the function is useful for arrays added dynamically after state creation, legacy code migration, or debugging reactivity issues. The function checks if the array is already patched to avoid double-patching, marks the array with a hidden `__patched` flag, and modifies the array methods directly. Use this function when array methods aren't triggering effects as expected, when adding arrays to state after creation, or when working with legacy code that doesn't auto-patch arrays. In modern code with proper initialization, manual patching is rarely needed.
