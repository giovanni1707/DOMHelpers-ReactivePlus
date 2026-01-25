# Collection Query Methods - Complete Reference

**Quick Start (30 seconds)**
```javascript
const collection = ReactiveUtils.collection(['apple', 'banana', 'cherry']);

// Get collection length
console.log(collection.length); // 3

// Get first item
console.log(collection.first); // 'apple'

// Get last item
console.log(collection.last); // 'cherry'

// Reactive - updates automatically
collection.add('date');
console.log(collection.length); // 4
console.log(collection.last); // 'date'
```

---

## **What are Collection Query Methods?**

Collection query methods are **reactive getter properties** that provide read-only access to collection metadata and items.

**The 3 core query getters:**
1. **`length`** - Returns the number of items in the collection
2. **`first`** - Returns the first item (or undefined if empty)
3. **`last`** - Returns the last item (or undefined if empty)

**Key characteristics:**
- **Reactive Getters**: Automatically track dependencies
- **Read-Only**: Cannot be assigned to directly
- **Always Current**: Return live values
- **Computed Values**: Derived from underlying items array
- **Side-Effect Free**: Safe to call repeatedly

---

## **Method Reference**

### **1. `length`**

**Type**: Getter
**Returns**: `number`
**Reactive**: Yes

Returns the current number of items in the collection.

```javascript
const collection = ReactiveUtils.collection([1, 2, 3]);

console.log(collection.length); // 3

collection.add(4);
console.log(collection.length); // 4

collection.remove(1);
console.log(collection.length); // 3
```

**Implementation:**
```javascript
get length() {
  return this.items.length;
}
```

**Use cases:**
- Check if collection is empty
- Loop boundaries
- Conditional rendering
- Validation
- Progress tracking

---

### **2. `first`**

**Type**: Getter
**Returns**: `any | undefined`
**Reactive**: Yes

Returns the first item in the collection, or `undefined` if empty.

```javascript
const collection = ReactiveUtils.collection(['a', 'b', 'c']);

console.log(collection.first); // 'a'

collection.clear();
console.log(collection.first); // undefined

collection.add('x');
console.log(collection.first); // 'x'
```

**Implementation:**
```javascript
get first() {
  return this.items[0];
}
```

**Use cases:**
- Get current/active item
- Display first element
- Priority queues
- Stack peek operations
- Default selections

---

### **3. `last`**

**Type**: Getter
**Returns**: `any | undefined`
**Reactive**: Yes

Returns the last item in the collection, or `undefined` if empty.

```javascript
const collection = ReactiveUtils.collection(['a', 'b', 'c']);

console.log(collection.last); // 'c'

collection.add('d');
console.log(collection.last); // 'd'

collection.pop();
console.log(collection.last); // 'c'
```

**Implementation:**
```javascript
get last() {
  return this.items[this.items.length - 1];
}
```

**Use cases:**
- Get most recent item
- Display latest addition
- Queue rear access
- Tail element operations
- Recent activity tracking

---

## **Examples**

### **Example 1: Empty Collection Check**
```javascript
const tasks = ReactiveUtils.collection();

if (tasks.length === 0) {
  console.log('No tasks yet');
}

tasks.add('Write documentation');

if (tasks.length > 0) {
  console.log(`You have ${tasks.length} task(s)`);
}
```

### **Example 2: First/Last Display**
```javascript
const queue = ReactiveUtils.collection(['Alice', 'Bob', 'Charlie']);

console.log(`Next in line: ${queue.first}`); // Alice
console.log(`Last in line: ${queue.last}`); // Charlie

queue.shift(); // Remove Alice
console.log(`Next in line: ${queue.first}`); // Bob
```

### **Example 3: Reactive UI Updates**
```javascript
const items = ReactiveUtils.collection(['Apple', 'Banana']);

ReactiveUtils.effect(() => {
  const status = items.length > 0
    ? `${items.length} items (${items.first} to ${items.last})`
    : 'Empty';

  document.getElementById('status').textContent = status;
});

items.add('Cherry'); // UI automatically updates
// Display: "3 items (Apple to Cherry)"
```

### **Example 4: Range Display**
```javascript
const numbers = ReactiveUtils.collection([10, 20, 30, 40, 50]);

function getRange() {
  if (numbers.length === 0) return 'Empty';
  if (numbers.length === 1) return `Single value: ${numbers.first}`;

  return `Range: ${numbers.first} to ${numbers.last} (${numbers.length} items)`;
}

console.log(getRange()); // "Range: 10 to 50 (5 items)"
```

### **Example 5: Progress Tracking**
```javascript
const downloads = ReactiveUtils.collection();
const totalFiles = 10;

ReactiveUtils.effect(() => {
  const progress = (downloads.length / totalFiles) * 100;
  console.log(`Progress: ${progress.toFixed(0)}%`);

  if (downloads.length === totalFiles) {
    console.log(`Complete! Last file: ${downloads.last}`);
  }
});

// Simulate downloads
downloads.add('file1.jpg');
downloads.add('file2.jpg');
// ... etc
```

### **Example 6: Stack Operations**
```javascript
const stack = ReactiveUtils.collection();

// Push
stack.add('First');
stack.add('Second');
stack.add('Third');

// Peek at top
console.log(`Top: ${stack.last}`); // Third

// Pop
const popped = stack.pop();
console.log(`Popped: ${popped}`); // Third
console.log(`New top: ${stack.last}`); // Second
```

### **Example 7: Queue Operations**
```javascript
const queue = ReactiveUtils.collection();

// Enqueue
queue.add('Task 1');
queue.add('Task 2');
queue.add('Task 3');

// Peek at front
console.log(`Next task: ${queue.first}`); // Task 1

// Dequeue
const next = queue.shift();
console.log(`Processing: ${next}`); // Task 1
console.log(`Next task: ${queue.first}`); // Task 2
```

### **Example 8: Validation**
```javascript
const cart = ReactiveUtils.collection();

function canCheckout() {
  if (cart.length === 0) {
    console.log('❌ Cart is empty');
    return false;
  }

  if (cart.length > 100) {
    console.log('❌ Too many items (max 100)');
    return false;
  }

  console.log(`✓ Ready to checkout with ${cart.length} items`);
  return true;
}

cart.add({ name: 'Apple', price: 1.5 });
canCheckout(); // ✓ Ready to checkout with 1 items
```

### **Example 9: Activity Log**
```javascript
const activityLog = ReactiveUtils.collection();
const maxLogSize = 100;

function logActivity(message) {
  activityLog.add({
    message,
    timestamp: Date.now()
  });

  // Keep only last 100 entries
  if (activityLog.length > maxLogSize) {
    activityLog.shift();
  }

  console.log(`Logged: ${message}`);
  console.log(`Total logs: ${activityLog.length}`);
  console.log(`Oldest: ${new Date(activityLog.first.timestamp).toISOString()}`);
  console.log(`Newest: ${new Date(activityLog.last.timestamp).toISOString()}`);
}

logActivity('User logged in');
logActivity('User viewed dashboard');
```

### **Example 10: Time Range Display**
```javascript
const events = ReactiveUtils.collection();

function getTimeRange() {
  if (events.length === 0) {
    return 'No events';
  }

  const firstTime = new Date(events.first.time).toLocaleTimeString();
  const lastTime = new Date(events.last.time).toLocaleTimeString();

  if (events.length === 1) {
    return `1 event at ${firstTime}`;
  }

  return `${events.length} events from ${firstTime} to ${lastTime}`;
}

events.add({ name: 'Start', time: Date.now() });
console.log(getTimeRange()); // "1 event at 10:30:45 AM"

setTimeout(() => {
  events.add({ name: 'End', time: Date.now() });
  console.log(getTimeRange()); // "2 events from 10:30:45 AM to 10:30:50 AM"
}, 5000);
```

### **Example 11: Conditional Rendering**
```javascript
const messages = ReactiveUtils.collection();

ReactiveUtils.effect(() => {
  const container = document.getElementById('messages');

  if (messages.length === 0) {
    container.innerHTML = '<p>No messages</p>';
  } else {
    container.innerHTML = `
      <div>
        <p>Showing ${messages.length} messages</p>
        <p>First: ${messages.first.text}</p>
        <p>Last: ${messages.last.text}</p>
      </div>
    `;
  }
});

messages.add({ text: 'Hello' });
messages.add({ text: 'World' });
```

### **Example 12: Pagination Info**
```javascript
const items = ReactiveUtils.collection();
const pageSize = 10;

function getPaginationInfo() {
  const totalPages = Math.ceil(items.length / pageSize);

  return {
    totalItems: items.length,
    totalPages,
    isEmpty: items.length === 0,
    firstItem: items.first,
    lastItem: items.last
  };
}

// Load items
for (let i = 1; i <= 25; i++) {
  items.add({ id: i, name: `Item ${i}` });
}

console.log(getPaginationInfo());
// {
//   totalItems: 25,
//   totalPages: 3,
//   isEmpty: false,
//   firstItem: { id: 1, name: 'Item 1' },
//   lastItem: { id: 25, name: 'Item 25' }
// }
```

### **Example 13: Smart Defaults**
```javascript
const users = ReactiveUtils.collection([
  { id: 1, name: 'Alice' },
  { id: 2, name: 'Bob' },
  { id: 3, name: 'Charlie' }
]);

const state = ReactiveUtils.state({
  selectedUser: null
});

// Set default to first user
if (!state.selectedUser && users.length > 0) {
  state.selectedUser = users.first;
}

console.log(`Selected: ${state.selectedUser.name}`); // Alice
```

### **Example 14: Comparison**
```javascript
const scores = ReactiveUtils.collection([85, 92, 78, 95, 88]);

function analyzeScores() {
  if (scores.length === 0) return 'No scores';

  const lowest = Math.min(...scores.items);
  const highest = Math.max(...scores.items);

  return {
    count: scores.length,
    first: scores.first,
    last: scores.last,
    lowest,
    highest,
    range: highest - lowest
  };
}

console.log(analyzeScores());
// {
//   count: 5,
//   first: 85,
//   last: 88,
//   lowest: 78,
//   highest: 95,
//   range: 17
// }
```

### **Example 15: Undo/Redo**
```javascript
const history = ReactiveUtils.collection();
const maxHistory = 50;

function addToHistory(action) {
  history.add(action);

  if (history.length > maxHistory) {
    history.shift();
  }

  console.log(`History: ${history.length} actions`);
  console.log(`Can undo: ${history.length > 0}`);
  console.log(`Last action: ${history.last?.type}`);
}

function undo() {
  if (history.length === 0) {
    console.log('Nothing to undo');
    return;
  }

  const action = history.pop();
  console.log(`Undoing: ${action.type}`);
  console.log(`Remaining: ${history.length} actions`);
}

addToHistory({ type: 'create', data: 'file.txt' });
addToHistory({ type: 'edit', data: 'Added text' });
addToHistory({ type: 'delete', data: 'Removed line' });

undo(); // Undoing: delete
```

### **Example 16: Batch Size Display**
```javascript
const batch = ReactiveUtils.collection();

ReactiveUtils.effect(() => {
  const size = batch.length;

  if (size === 0) {
    console.log('Batch empty');
  } else if (size < 10) {
    console.log(`Small batch: ${size} items`);
  } else if (size < 100) {
    console.log(`Medium batch: ${size} items`);
  } else {
    console.log(`Large batch: ${size} items`);
  }

  console.log(`From: ${batch.first?.id || 'N/A'} to ${batch.last?.id || 'N/A'}`);
});

// Add items
for (let i = 1; i <= 15; i++) {
  batch.add({ id: i, value: i * 10 });
}
// Output: "Medium batch: 15 items"
// Output: "From: 1 to 15"
```

### **Example 17: Sliding Window**
```javascript
const window = ReactiveUtils.collection();
const windowSize = 5;

function addToWindow(value) {
  window.add(value);

  // Keep only last N items
  while (window.length > windowSize) {
    window.shift();
  }

  console.log(`Window: [${window.items.join(', ')}]`);
  console.log(`Range: ${window.first} to ${window.last}`);
}

addToWindow(10); // [10] - Range: 10 to 10
addToWindow(20); // [10, 20] - Range: 10 to 20
addToWindow(30); // [10, 20, 30] - Range: 10 to 30
addToWindow(40); // [10, 20, 30, 40] - Range: 10 to 40
addToWindow(50); // [10, 20, 30, 40, 50] - Range: 10 to 50
addToWindow(60); // [20, 30, 40, 50, 60] - Range: 20 to 60
```

### **Example 18: Recent Items Widget**
```javascript
const recentFiles = ReactiveUtils.collection();

function openFile(filename) {
  // Remove if already in list
  const index = recentFiles.items.indexOf(filename);
  if (index !== -1) {
    recentFiles.splice(index, 1);
  }

  // Add to end (most recent)
  recentFiles.add(filename);

  // Keep only 10 most recent
  while (recentFiles.length > 10) {
    recentFiles.shift();
  }

  displayRecent();
}

function displayRecent() {
  console.log('\n📂 Recent Files:');

  if (recentFiles.length === 0) {
    console.log('  (none)');
    return;
  }

  console.log(`  Most recent: ${recentFiles.last}`);
  console.log(`  Oldest: ${recentFiles.first}`);
  console.log(`  Total: ${recentFiles.length}`);
}

openFile('document.txt');
openFile('image.png');
openFile('code.js');
openFile('document.txt'); // Moves to most recent
```

### **Example 19: Buffer Status**
```javascript
const buffer = ReactiveUtils.collection();
const bufferCapacity = 100;

ReactiveUtils.effect(() => {
  const used = buffer.length;
  const available = bufferCapacity - used;
  const percentage = (used / bufferCapacity) * 100;

  console.log(`\n📊 Buffer Status:`);
  console.log(`  Used: ${used}/${bufferCapacity} (${percentage.toFixed(1)}%)`);
  console.log(`  Available: ${available}`);

  if (buffer.length > 0) {
    console.log(`  First: ${JSON.stringify(buffer.first)}`);
    console.log(`  Last: ${JSON.stringify(buffer.last)}`);
  }

  if (percentage > 90) {
    console.log('  ⚠️  Buffer nearly full!');
  }
});

// Add data
for (let i = 0; i < 95; i++) {
  buffer.add({ id: i, data: `Item ${i}` });
}
```

### **Example 20: Change Detection**
```javascript
const items = ReactiveUtils.collection([1, 2, 3]);

let previousLength = items.length;
let previousFirst = items.first;
let previousLast = items.last;

ReactiveUtils.effect(() => {
  const lengthChanged = items.length !== previousLength;
  const firstChanged = items.first !== previousFirst;
  const lastChanged = items.last !== previousLast;

  if (lengthChanged || firstChanged || lastChanged) {
    console.log('\n🔄 Collection changed:');

    if (lengthChanged) {
      console.log(`  Length: ${previousLength} → ${items.length}`);
    }
    if (firstChanged) {
      console.log(`  First: ${previousFirst} → ${items.first}`);
    }
    if (lastChanged) {
      console.log(`  Last: ${previousLast} → ${items.last}`);
    }

    previousLength = items.length;
    previousFirst = items.first;
    previousLast = items.last;
  }
});

items.add(4); // Length and last change
items.shift(); // Length and first change
```

---

## **Common Patterns**

### **Pattern 1: Empty Check**
```javascript
if (collection.length === 0) {
  console.log('Empty');
}
```

### **Pattern 2: First/Last Access**
```javascript
const firstItem = collection.first;
const lastItem = collection.last;
```

### **Pattern 3: Range Display**
```javascript
const range = `${collection.first} to ${collection.last}`;
```

### **Pattern 4: Reactive Count**
```javascript
ReactiveUtils.effect(() => {
  console.log(`Count: ${collection.length}`);
});
```

### **Pattern 5: Safe Access**
```javascript
const item = collection.first ?? 'No items';
```

---

## **Query Method Comparison**

| Method | Returns | Empty Collection | Use Case |
|--------|---------|------------------|----------|
| `length` | number | 0 | Count items, validate size |
| `first` | any \| undefined | undefined | Get first/oldest item |
| `last` | any \| undefined | undefined | Get last/newest item |

---

## **Return Values**

### **`length`**
- Always returns a number (≥ 0)
- Never undefined
- 0 for empty collections

### **`first` and `last`**
- Returns item if exists
- Returns undefined if empty
- Use optional chaining for safety: `collection.first?.property`

---

## **Reactive Behavior**

All three getters are **reactive** - they trigger effects when their values change:

```javascript
const items = ReactiveUtils.collection([1, 2, 3]);

ReactiveUtils.effect(() => {
  console.log(`Length: ${items.length}`); // Tracks length
});

ReactiveUtils.effect(() => {
  console.log(`First: ${items.first}`); // Tracks first
});

ReactiveUtils.effect(() => {
  console.log(`Last: ${items.last}`); // Tracks last
});

items.add(4); // All three effects re-run
```

---

## **Performance**

All getters are **O(1)** operations:
- `length`: Direct property access
- `first`: Array index access `[0]`
- `last`: Array index access `[length - 1]`

No iteration or computation required.

---

## **Key Takeaways**

1. **Reactive Getters**: Automatically track dependencies in effects
2. **Read-Only**: Cannot assign to these properties
3. **Always Current**: Return live values from underlying array
4. **Safe**: Return undefined for empty collections (first/last)
5. **Efficient**: O(1) time complexity for all three
6. **Versatile**: Cover most query needs (count, first, last)
7. **Side-Effect Free**: Safe to call multiple times
8. **Type Safe**: Predictable return types

---

## **Summary**

Collection query methods provide reactive, read-only access to collection metadata through three getter properties: `length` (returns item count), `first` (returns first item or undefined), and `last` (returns last item or undefined). All three are reactive getters that automatically track dependencies in effects, enabling automatic UI updates when values change. They're implemented as simple O(1) operations directly accessing the underlying items array, making them highly efficient. The `length` getter always returns a number (0 for empty collections), while `first` and `last` return undefined for empty collections, requiring optional chaining for safe property access. These getters cover the most common query patterns: checking if collections are empty, accessing head/tail elements, displaying ranges, and tracking counts - all essential for building reactive user interfaces and managing collection state effectively.
