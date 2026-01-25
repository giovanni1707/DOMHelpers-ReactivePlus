# `collection.items` - Reactive Array Property

## Quick Start (30 seconds)

```javascript
// Create collection
const todos = collection([
  { id: 1, text: 'Buy milk', done: false },
  { id: 2, text: 'Walk dog', done: true }
]);

// Access items directly
console.log(todos.items);
// Output: [{ id: 1, ...}, { id: 2, ...}]

// Items is reactive
effect(() => {
  console.log('Todo count:', todos.items.length);
});
// Output: Todo count: 2

// Modify items - effect runs
todos.items.push({ id: 3, text: 'Read book', done: false });
// Output: Todo count: 3

// Direct assignment also works
todos.items = [{ id: 1, text: 'New task', done: false }];
// Output: Todo count: 1
```

**That's it.** The `items` property holds the reactive array for collections. Access and modify it like a normal array, with full reactivity.

---

## What is `collection.items`?

`collection.items` **is the reactive array property that holds the collection's data**. Created by the `collection()` function, it's a normal JavaScript array wrapped in reactivity.

Think of it as **the data container** — where the actual items live in a collection, fully reactive and ready to use.

**In practical terms:** When you create a collection, `items` is the property you read and modify to work with the collection's data.

---

## Syntax

```javascript
// Create collection
const myCollection = collection([item1, item2, item3]);

// Access items
const allItems = myCollection.items;

// Read items
console.log(myCollection.items[0]);
console.log(myCollection.items.length);

// Modify items
myCollection.items.push(newItem);
myCollection.items[0] = updatedItem;
myCollection.items = newArray;

// Use in effects
effect(() => {
  console.log('Items:', myCollection.items);
});
```

**Type:**
- Array (reactive)

**Properties:**
- Standard array properties (length, etc.)
- Reactive — changes trigger effects

**Important:**
- This is a **regular array** with reactivity
- All array methods work: `push`, `pop`, `map`, `filter`, etc.
- Modifying items **triggers effects**
- Can be replaced entirely with assignment

---

## Why Does This Exist?

### The Problem Without collection.items

Managing reactive arrays requires manual setup:

```javascript
// ❌ Manual reactive array setup
const todos = state({
  list: []  // Where should the array go? What should it be called?
});

effect(() => {
  console.log('Todos:', todos.list);
});

// Need to access nested property
todos.list.push(newTodo);
console.log(todos.list.length);

// Confusing naming - is it "list", "data", "items", "array"?
```

**Problems:**
❌ **Inconsistent naming** - Each developer picks different names
❌ **Nested access** - Always need `state.array`
❌ **No convention** - No standard way to access collection data
❌ **Manual setup** - Need to create state structure yourself

### The Solution with `collection.items`

```javascript
// ✅ Standardized collection with items property
const todos = collection([]);

effect(() => {
  console.log('Todos:', todos.items);
});

// Standard property name
todos.items.push(newTodo);
console.log(todos.items.length);

// Everyone knows it's "items" - consistent!
```

**Benefits:**
✅ **Standard naming** - Always use `items`
✅ **Consistent API** - Every collection works the same
✅ **Simple access** - Direct property access
✅ **Convention** - Clear, predictable structure
✅ **Fully reactive** - Built-in reactivity support

---

## Mental Model: The Data Container

Think of `collection.items` like **a labeled box**:

**Manual State (Any Name):**
```
┌─────────────────────────────┐
│  State Object               │
│  ┌───────────────────────┐  │
│  │ todos.??? = [...]     │  │
│  │ (what's it called?)   │  │
│  └───────────────────────┘  │
│                             │
│  Inconsistent naming        │
└─────────────────────────────┘
```

**Collection with items (Standard):**
```
┌─────────────────────────────┐
│  Collection Object          │
│  ┌───────────────────────┐  │
│  │ .items = [...]        │  │
│  │ (always "items")      │  │
│  └───────────────────────┘  │
│                             │
│  Standard, predictable      │
└─────────────────────────────┘
```

`collection.items` is **the labeled box** — you always know where the data is.

---

## How Does It Work?

Collections create a state object with `items` property:

```
Call collection([...])
    ↓
Create reactive state:
  {
    items: [...]  // Your data here
  }
    ↓
Add collection methods
    ↓
Return reactive object
    ↓
Access items:
  collection.items → Tracked
    ↓
Modify items:
  collection.items.push() → Triggers effects
```

**Key behaviors:**
- Created by `collection()` function
- Holds the **actual array** data
- Fully **reactive** property
- Can be **read** or **written**
- Supports all **array operations**

---

## Basic Usage

### Example 1: Simple Todo List

```javascript
const todos = collection([
  { id: 1, text: 'Buy milk', done: false }
]);

// Access items
console.log(todos.items);
// [{ id: 1, text: 'Buy milk', done: false }]

// Add item
todos.items.push({ id: 2, text: 'Walk dog', done: false });

// Update item
todos.items[0].done = true;

// Remove item
todos.items.splice(0, 1);

// Replace all
todos.items = [
  { id: 3, text: 'New task', done: false }
];
```

---

### Example 2: Reactive Rendering

```javascript
const products = collection([
  { id: 1, name: 'Laptop', price: 999 },
  { id: 2, name: 'Mouse', price: 29 }
]);

// Render when items change
effect(() => {
  const html = products.items
    .map(p => `<li>${p.name}: $${p.price}</li>`)
    .join('');

  document.getElementById('products').innerHTML = html;
});

// Add product - DOM updates automatically
products.items.push({ id: 3, name: 'Keyboard', price: 79 });
```

---

### Example 3: Computed Properties

```javascript
const cart = collection([
  { name: 'Laptop', price: 999, qty: 1 }
]);

// Computed based on items
computed(cart, {
  total: function() {
    return this.items.reduce((sum, item) => sum + item.price * item.qty, 0);
  },
  count: function() {
    return this.items.length;
  }
});

console.log(cart.total); // 999
console.log(cart.count); // 1

cart.items.push({ name: 'Mouse', price: 29, qty: 2 });
console.log(cart.total); // 1057
console.log(cart.count); // 2
```

---

### Example 4: Filtering

```javascript
const tasks = collection([
  { id: 1, text: 'Task 1', done: false },
  { id: 2, text: 'Task 2', done: true },
  { id: 3, text: 'Task 3', done: false }
]);

// Filter items
effect(() => {
  const active = tasks.items.filter(t => !t.done);
  console.log('Active tasks:', active.length);
});

// Mark task done - effect runs
tasks.items[0].done = true;
// Output: Active tasks: 1
```

---

### Example 5: Sorting

```javascript
const users = collection([
  { name: 'Charlie', age: 35 },
  { name: 'Alice', age: 28 },
  { name: 'Bob', age: 42 }
]);

// Sort items
const sortedByName = [...users.items].sort((a, b) =>
  a.name.localeCompare(b.name)
);

console.log(sortedByName);
// [{ name: 'Alice', ...}, { name: 'Bob', ...}, { name: 'Charlie', ...}]

// Sort in-place (triggers reactivity)
users.items.sort((a, b) => a.age - b.age);
```

---

### Example 6: Array Methods

```javascript
const numbers = collection([1, 2, 3, 4, 5]);

// Map
const doubled = numbers.items.map(n => n * 2);
console.log(doubled); // [2, 4, 6, 8, 10]

// Filter
const evens = numbers.items.filter(n => n % 2 === 0);
console.log(evens); // [2, 4]

// Reduce
const sum = numbers.items.reduce((s, n) => s + n, 0);
console.log(sum); // 15

// Find
const found = numbers.items.find(n => n > 3);
console.log(found); // 4

// Every/Some
const allPositive = numbers.items.every(n => n > 0);
const hasNegative = numbers.items.some(n => n < 0);
```

---

### Example 7: Direct Assignment

```javascript
const messages = collection([
  { id: 1, text: 'Hello' }
]);

// Replace entire array
messages.items = [
  { id: 2, text: 'New message' },
  { id: 3, text: 'Another message' }
];

// Effects see the new array
effect(() => {
  console.log('Message count:', messages.items.length);
});
// Output: Message count: 2
```

---

### Example 8: Watching Changes

```javascript
const inventory = collection([
  { sku: 'A001', stock: 10 },
  { sku: 'A002', stock: 5 }
]);

// Watch for low stock
effect(() => {
  const lowStock = inventory.items.filter(item => item.stock < 10);

  if (lowStock.length > 0) {
    console.log('Low stock alert:', lowStock);
  }
});

// Update stock - effect runs
inventory.items[1].stock = 3;
// Output: Low stock alert: [{ sku: 'A002', stock: 3 }]
```

---

### Example 9: Pagination

```javascript
const allItems = collection(Array.from({ length: 100 }, (_, i) => ({
  id: i + 1,
  name: `Item ${i + 1}`
})));

const pagination = state({
  page: 1,
  pageSize: 10
});

// Computed page of items
computed(pagination, {
  pageItems: function() {
    const start = (this.page - 1) * this.pageSize;
    const end = start + this.pageSize;
    return allItems.items.slice(start, end);
  }
});

console.log(pagination.pageItems.length); // 10

pagination.page = 2;
console.log(pagination.pageItems[0].id); // 11
```

---

### Example 10: Nested Collections

```javascript
const categories = collection([
  {
    name: 'Electronics',
    items: [
      { name: 'Laptop', price: 999 },
      { name: 'Mouse', price: 29 }
    ]
  },
  {
    name: 'Furniture',
    items: [
      { name: 'Desk', price: 299 },
      { name: 'Chair', price: 199 }
    ]
  }
]);

// Access nested items
const electronicsItems = categories.items[0].items;
console.log(electronicsItems.length); // 2

// Modify nested items
categories.items[0].items.push({ name: 'Keyboard', price: 79 });
```

---

## Advanced Usage: Custom Collection Methods

```javascript
const tasks = collection([]);

// Add custom methods
tasks.$addTask = function(text) {
  this.items.push({
    id: Date.now(),
    text,
    done: false,
    createdAt: new Date()
  });
};

tasks.$completeTask = function(id) {
  const task = this.items.find(t => t.id === id);
  if (task) task.done = true;
};

tasks.$removeDone = function() {
  this.items = this.items.filter(t => !t.done);
};

// Use custom methods
tasks.$addTask('Buy milk');
tasks.$addTask('Walk dog');

const firstId = tasks.items[0].id;
tasks.$completeTask(firstId);

tasks.$removeDone();
console.log(tasks.items.length); // 1
```

---

## Common Patterns

### Pattern 1: Read Items

```javascript
const items = collection.items;
const firstItem = collection.items[0];
const length = collection.items.length;
```

### Pattern 2: Add Items

```javascript
collection.items.push(newItem);
collection.items.unshift(newItem);
collection.items.splice(index, 0, newItem);
```

### Pattern 3: Remove Items

```javascript
collection.items.pop();
collection.items.shift();
collection.items.splice(index, 1);
collection.items = collection.items.filter(predicate);
```

### Pattern 4: Update Items

```javascript
collection.items[index] = updatedItem;
collection.items[index].property = newValue;
Object.assign(collection.items[index], updates);
```

### Pattern 5: Replace All

```javascript
collection.items = newArray;
```

---

## Key Takeaways

✅ **Standard property** - Always named `items`
✅ **Reactive array** - Changes trigger effects
✅ **Full array API** - All array methods work
✅ **Direct access** - Read and write easily
✅ **Conventional** - Predictable, consistent
✅ **Built-in** - Created automatically by `collection()`

---

## What's Next?

- **`collection()`** - Create reactive collections
- **`collection.$add()`** - Add items with method
- **`collection.$remove()`** - Remove items with method

---

## Summary

`collection.items` **is the reactive array property that holds collection data**. It's a standard, predictable way to access and modify collection contents with full reactivity.

**The magic formula:**
```
const col = collection([...]);
col.items → access data
col.items.push(...) → modify data
  =
Standard array property
─────────────────────
Always "items", always reactive
```

Think of it as **the data container** — the standard, conventional place where collection data lives. Always called `items`, always reactive, always a plain JavaScript array. Perfect for consistent, predictable collection access across your application.
