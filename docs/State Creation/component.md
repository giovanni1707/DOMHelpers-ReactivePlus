# `component()` - Complete Component Pattern

## Quick Start (30 seconds)

```javascript
// Create self-contained component with everything
const counterComponent = component({
  state: {
    count: 0,
    step: 1
  },
  computed: {
    double: function() { return this.count * 2; }
  },
  bindings: {
    '#count': () => counterComponent.state.count,
    '#double': () => counterComponent.state.double
  },
  actions: {
    increment: function() { this.count += this.step; },
    decrement: function() { this.count -= this.step; }
  },
  mounted() {
    console.log('Component mounted!');
  }
});

// Use it using bulk event binding
Elements.update({
  incrementBtn: {
    addEventListener: ['click', () => counterComponent.increment()]
  }
});

// Later: cleanup
counterComponent.$destroy();
```

**That's it.** Create complete, self-contained components with state, computed, watchers, effects, bindings, actions, and lifecycle hooks all in one config object.

---

## What is `component()`?

`component()` creates **complete, self-contained components** with state, computed properties, watchers, effects, bindings, actions, and lifecycle hooks. It's the all-in-one pattern for building reusable UI components.

Think of it as **a component framework** built into your reactive library — everything you need to build isolated, reusable components.

**In practical terms:** Instead of manually wiring up state, computed, effects, and bindings separately, you define everything in one config object and get a complete component ready to use.

---

## Syntax

```javascript
// Create component
const myComponent = component(config);

// Config structure
const config = {
  state: {},          // Initial state
  computed: {},       // Computed properties
  watch: {},          // Watchers
  effects: {},        // Effects
  bindings: {},       // DOM bindings
  actions: {},        // Action methods
  mounted() {}        // Lifecycle hook (called on creation)
};

// Example
const todoComponent = component({
  state: {
    todos: [],
    filter: 'all'
  },
  computed: {
    filteredTodos: function() {
      if (this.filter === 'active') return this.todos.filter(t => !t.done);
      if (this.filter === 'completed') return this.todos.filter(t => t.done);
      return this.todos;
    }
  },
  watch: {
    todos: (newTodos) => {
      console.log(`Now ${newTodos.length} todos`);
    }
  },
  effects: {
    renderList: () => {
      Elements.update({
        todoList: {
          innerHTML: todoComponent.filteredTodos
            .map(t => `<li>${t.text}</li>`)
            .join('')
        }
      });
    }
  },
  bindings: {
    '#todo-count': () => todoComponent.todos.length
  },
  actions: {
    addTodo: function(text) {
      this.todos.push({ text, done: false });
    }
  },
  mounted() {
    console.log('Todo component ready!');
  }
});

// Use component
todoComponent.addTodo('Buy milk');

// Cleanup when done
todoComponent.$destroy();
```

**Parameters:**
- `config.state` - Initial state object
- `config.computed` (optional) - Computed properties
- `config.watch` (optional) - Property watchers
- `config.effects` (optional) - Reactive effects
- `config.bindings` (optional) - DOM bindings
- `config.actions` (optional) - Action methods
- `config.mounted` (optional) - Lifecycle hook

**Returns:**
- Component state with all features and `$destroy()` method

---

## Why Does This Exist?

### The Problem Without component()

Building reusable components manually requires lots of setup:

```javascript
// ❌ Vanilla JavaScript - manual component setup
function createCounter(elementId) {
  let count = 0;
  let step = 1;

  const element = document.getElementById(elementId);

  function getDouble() {
    return count * 2;
  }

  function render() {
    element.querySelector('.count').textContent = count;
    element.querySelector('.double').textContent = getDouble();
  }

  function increment() {
    count += step;
    render();
  }

  function decrement() {
    count -= step;
    render();
  }

  function destroy() {
    // Manual cleanup
    element.innerHTML = '';
  }

  // Initial render
  render();

  return {
    increment,
    decrement,
    destroy
  };
}

const counter1 = createCounter('counter-1');
const counter2 = createCounter('counter-2');
```

**Problems:**
❌ **Manual everything** - Render, state, cleanup all manual
❌ **No reactivity** - Must call render() everywhere
❌ **Scattered logic** - State, getters, actions not organized
❌ **No standard pattern** - Each component different
❌ **Hard to maintain** - Lots of boilerplate

### The Solution with `component()`

```javascript
// ✅ DOM Helpers + Reactive State with component() - complete pattern
function createCounter(prefix) {
  return component({
    state: {
      count: 0,
      step: 1
    },
    computed: {
      double: function() {
        return this.count * 2;
      }
    },
    bindings: {
      [`#${prefix}-count`]: () => counter.count,
      [`#${prefix}-double`]: () => counter.double
    },
    watch: {
      count: (newCount) => {
        console.log(`${prefix}: count changed to ${newCount}`);
      }
    },
    actions: {
      increment: function() {
        this.count += this.step;
      },
      decrement: function() {
        this.count -= this.step;
      },
      reset: function() {
        this.count = 0;
      }
    },
    mounted() {
      console.log(`Counter ${prefix} mounted!`);
    }
  });
}

const counter1 = createCounter('counter-1');
const counter2 = createCounter('counter-2');

// Use them using bulk event binding
Elements.update({
  'counter-1-inc-btn': {
    addEventListener: ['click', () => counter1.increment()]
  },
  'counter-2-inc-btn': {
    addEventListener: ['click', () => counter2.increment()]
  }
});

// Cleanup
counter1.$destroy();
counter2.$destroy();
```

**Benefits:**
✅ **Complete pattern** - State, computed, watch, effects, bindings, actions all included
✅ **Automatic reactivity** - Everything updates automatically
✅ **Organized structure** - Clear, consistent component pattern
✅ **Lifecycle hooks** - mounted() for initialization
✅ **Automatic cleanup** - $destroy() removes all effects and watchers

---

## Mental Model: Component Blueprint

Think of `component()` like **a complete blueprint for building components**:

**Without component() (Manual Construction):**
```
┌─────────────────────────────┐
│  Build Everything Manually  │
│                             │
│  ☐ Create state             │
│  ☐ Add computed             │
│  ☐ Set up watchers          │
│  ☐ Create effects           │
│  ☐ Add bindings             │
│  ☐ Write actions            │
│  ☐ Setup lifecycle          │
│  ☐ Remember cleanup         │
│                             │
│  Lots of manual work!       │
└─────────────────────────────┘
```

**With component() (Blueprint Pattern):**
```
┌─────────────────────────────┐
│  Complete Blueprint         │
│                             │
│  ✓ State                    │
│  ✓ Computed                 │
│  ✓ Watch                    │
│  ✓ Effects                  │
│  ✓ Bindings                 │
│  ✓ Actions                  │
│  ✓ Mounted hook             │
│  ✓ Auto cleanup             │
│                             │
│  Everything included!       │
└─────────────────────────────┘
```

You provide the **blueprint** (config), and get a **complete component** ready to use.

---

## Component Configuration

### state
Initial component state (like data in Vue/React state).

```javascript
state: {
  count: 0,
  name: '',
  items: []
}
```

### computed
Computed properties (like Vue computed/React useMemo).

```javascript
computed: {
  double: function() { return this.count * 2; },
  total: function() { return this.items.length; }
}
```

### watch
Property watchers (like Vue watch/React useEffect with deps).

```javascript
watch: {
  count: (newVal, oldVal) => {
    console.log(`Count: ${oldVal} → ${newVal}`);
  }
}
```

### effects
Reactive effects (like React useEffect).

```javascript
effects: {
  logCount: () => {
    console.log('Current count:', component.count);
  }
}
```

### bindings
DOM bindings (auto DOM sync).

```javascript
bindings: {
  '#display': () => component.count,
  '#double': () => component.double
}
```

### actions
Methods that modify state (like Vue methods/React handlers).

```javascript
actions: {
  increment: function() {
    this.count++;
  },
  reset: function() {
    this.count = 0;
  }
}
```

### mounted
Lifecycle hook called when component is created.

```javascript
mounted() {
  console.log('Component ready!');
  this.loadData();
}
```

---

## Basic Usage

### Example 1: Todo List Component

```javascript
const todoList = component({
  state: {
    todos: [],
    newTodoText: ''
  },
  computed: {
    activeCount: function() {
      return this.todos.filter(t => !t.done).length;
    },
    completedCount: function() {
      return this.todos.filter(t => t.done).length;
    }
  },
  effects: {
    renderList: () => {
      Elements.update({
        todoList: {
          innerHTML: todoList.todos
            .map((t, i) => `
              <li class="${t.done ? 'done' : ''}">
                <input type="checkbox" ${t.done ? 'checked' : ''}
                       data-index="${i}">
                <span>${t.text}</span>
                <button data-index="${i}">Delete</button>
              </li>
            `)
            .join('')
        }
      });
    }
  },
  bindings: {
    '#active-count': () => todoList.activeCount,
    '#completed-count': () => todoList.completedCount,
    '#total-count': () => todoList.todos.length
  },
  actions: {
    addTodo: function(text) {
      if (text.trim()) {
        this.todos.push({
          id: Date.now(),
          text: text.trim(),
          done: false
        });
      }
    },
    toggleTodo: function(index) {
      this.todos[index].done = !this.todos[index].done;
    },
    removeTodo: function(index) {
      this.todos.splice(index, 1);
    },
    clearCompleted: function() {
      this.todos = this.todos.filter(t => !t.done);
    }
  },
  mounted() {
    // Load from localStorage
    const saved = localStorage.getItem('todos');
    if (saved) {
      this.todos = JSON.parse(saved);
    }
  }
});

// Save to localStorage on changes
watch(todoList, 'todos', (todos) => {
  localStorage.setItem('todos', JSON.stringify(todos));
});

// Event handlers using bulk event binding
Elements.update({
  addTodoBtn: {
    addEventListener: ['click', () => {
      todoList.addTodo(Elements.todoInput.value);
      Elements.todoInput.value = '';
    }]
  },
  todoList: {
    addEventListener: ['click', (e) => {
      const index = parseInt(e.target.dataset.index);

      if (e.target.type === 'checkbox') {
        todoList.toggleTodo(index);
      }

      if (e.target.tagName === 'BUTTON') {
        todoList.removeTodo(index);
      }
    }]
  }
});
```

---

### Example 2: Timer Component

```javascript
const timer = component({
  state: {
    seconds: 0,
    isRunning: false,
    intervalId: null
  },
  computed: {
    formattedTime: function() {
      const mins = Math.floor(this.seconds / 60);
      const secs = this.seconds % 60;
      return `${mins.toString().padStart(2, '0')}:${secs.toString().padStart(2, '0')}`;
    }
  },
  bindings: {
    '#timer-display': () => timer.formattedTime,
    '#start-btn': () => timer.isRunning ? 'Pause' : 'Start',
    '#status': () => timer.isRunning ? '⏸ Running' : '⏹ Stopped'
  },
  actions: {
    start: function() {
      if (!this.isRunning) {
        this.isRunning = true;
        this.intervalId = setInterval(() => {
          this.seconds++;
        }, 1000);
      }
    },
    pause: function() {
      this.isRunning = false;
      if (this.intervalId) {
        clearInterval(this.intervalId);
        this.intervalId = null;
      }
    },
    reset: function() {
      this.pause();
      this.seconds = 0;
    },
    toggle: function() {
      if (this.isRunning) {
        this.pause();
      } else {
        this.start();
      }
    }
  },
  mounted() {
    console.log('Timer component ready!');
  }
});

// Event handlers using bulk event binding
Elements.update({
  startBtn: {
    addEventListener: ['click', () => timer.toggle()]
  },
  resetBtn: {
    addEventListener: ['click', () => timer.reset()]
  }
});

// Cleanup on unmount
window.addEventListener('beforeunload', () => {
  timer.$destroy();
});
```

---

### Example 3: Profile Card Component

```javascript
function createProfileCard(userData, containerId) {
  return component({
    state: {
      user: userData,
      isEditing: false,
      tempName: '',
      tempBio: ''
    },
    computed: {
      initials: function() {
        return this.user.name
          .split(' ')
          .map(n => n[0])
          .join('');
      }
    },
    effects: {
      renderCard: () => {
        const prefix = `#${containerId}`;

        Elements.update({
          [`${prefix}-name`]: {
            textContent: card.user.name,
            style: { display: card.isEditing ? 'none' : 'block' }
          },
          [`${prefix}-bio`]: {
            textContent: card.user.bio,
            style: { display: card.isEditing ? 'none' : 'block' }
          },
          [`${prefix}-initials`]: {
            textContent: card.initials
          },
          [`${prefix}-edit-form`]: {
            style: { display: card.isEditing ? 'block' : 'none' }
          },
          [`${prefix}-name-input`]: {
            value: card.tempName
          },
          [`${prefix}-bio-input`]: {
            value: card.tempBio
          }
        });
      }
    },
    actions: {
      startEdit: function() {
        this.tempName = this.user.name;
        this.tempBio = this.user.bio;
        this.isEditing = true;
      },
      saveEdit: function() {
        this.user.name = this.tempName;
        this.user.bio = this.tempBio;
        this.isEditing = false;
      },
      cancelEdit: function() {
        this.isEditing = false;
      }
    },
    mounted() {
      console.log(`Profile card for ${this.user.name} mounted`);
    }
  });

  const card = arguments[2]; // Get component reference
  return card;
}

// Create multiple instances
const card1 = createProfileCard({ name: 'Alice', bio: 'Developer' }, 'card-1');
const card2 = createProfileCard({ name: 'Bob', bio: 'Designer' }, 'card-2');
```

---

## Component Lifecycle

### Creation
1. State is created
2. Computed properties added
3. Watchers set up
4. Effects created
5. Bindings established
6. Actions added to state
7. `mounted()` hook called

### Updates
- State changes trigger computed recalculation
- Effects re-run when dependencies change
- Bindings auto-update DOM
- Watchers trigger on property changes

### Destruction
- Call `$destroy()` to cleanup
- All effects removed
- All watchers removed
- All bindings removed
- No memory leaks

---

## Multiple Component Instances

```javascript
function createCounter(id) {
  return component({
    state: { count: 0 },
    bindings: {
      [`#${id}-display`]: () => counter.count
    },
    actions: {
      increment: function() { this.count++; }
    }
  });

  const counter = arguments[1];
  return counter;
}

// Create multiple independent instances
const counter1 = createCounter('counter-1');
const counter2 = createCounter('counter-2');
const counter3 = createCounter('counter-3');

// Each has independent state
counter1.increment(); // Only counter1 updates
counter2.increment(); // Only counter2 updates
```

---

## Key Takeaways

✅ **All-in-one** - State, computed, watch, effects, bindings, actions, lifecycle
✅ **Component pattern** - Reusable, self-contained components
✅ **Automatic cleanup** - $destroy() removes everything
✅ **Lifecycle hooks** - mounted() for initialization
✅ **Multiple instances** - Create many components from same config

---

## What's Next?

- **`builder()`** - Chainable component construction
- **`store()`** - Global state management
- **`state()`** - Simple reactive state

---

## Summary

`component()` creates **complete, self-contained components** with everything needed: state, computed, watchers, effects, bindings, actions, and lifecycle hooks.

**The magic formula:**
```
component(config) =
  state +
  computed +
  watch +
  effects +
  bindings +
  actions +
  lifecycle
──────────────────────────────
Complete reusable component
```

Think of it as **a component framework built-in** — define your component configuration once, and get a fully-featured, reactive component ready to use. Perfect for building reusable UI widgets, isolated features, or complete mini-applications.
