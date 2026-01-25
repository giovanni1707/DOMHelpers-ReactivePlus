# `builder.bind(defs)` - Add DOM Bindings

**Quick Start (30 seconds)**
```javascript
const app = ReactiveUtils.reactive({ count: 0, name: 'John' })
  .bind({
    '#counter': 'count',  // Bind count to #counter element
    '#name': 'name'       // Bind name to #name element
  })
  .build();

app.count = 5; // #counter automatically updates to "5"
app.name = 'Jane'; // #name automatically updates to "Jane"
```

---

## **What is `builder.bind(defs)`?**

`builder.bind(defs)` is a **builder method** that creates automatic DOM bindings, synchronizing reactive state with DOM elements.

**Key characteristics:**
- **Auto-Update**: DOM updates when state changes
- **Declarative**: Simple selector-to-property mapping
- **Multiple Bindings**: Bind many elements at once
- **Chainable**: Returns builder for method chaining
- **Auto-Cleanup**: Bindings cleaned up on destroy

---

## **Syntax**

```javascript
builder.bind(definitions)
```

### **Parameters**
- **`definitions`** (Object): Object mapping selectors to bindings

**Binding types:**
- String: Property name
- Function: Computed binding
- Object: Multiple property bindings

### **Returns**
- **Type**: Builder instance (for chaining)

---

## **How it works**

```javascript
builder.bind(defs) {
  cleanups.push(bindings(defs));
  return this;
}
```

---

## **Examples**

### **Example 1: Simple Property Binding**
```javascript
const app = ReactiveUtils.reactive({ message: 'Hello' })
  .bind({
    '#output': 'message'
  })
  .build();

app.message = 'Hello World!'; // #output updates
```

### **Example 2: Multiple Bindings**
```javascript
const app = ReactiveUtils.reactive({
  title: 'My App',
  count: 0,
  status: 'active'
})
.bind({
  'h1': 'title',
  '#counter': 'count',
  '.status': 'status'
})
.build();
```

### **Example 3: Computed Binding**
```javascript
const user = ReactiveUtils.reactive({
  firstName: 'John',
  lastName: 'Doe'
})
.bind({
  '#fullName': function() {
    return `${this.firstName} ${this.lastName}`;
  }
})
.build();

user.firstName = 'Jane'; // #fullName updates to "Jane Doe"
```

### **Example 4: Property-Specific Bindings**
```javascript
const app = ReactiveUtils.reactive({
  isActive: true,
  message: 'Hello'
})
.bind({
  '#btn': {
    textContent: 'message',
    disabled: function() { return !this.isActive; },
    className: function() { return this.isActive ? 'active' : 'inactive'; }
  }
})
.build();
```

### **Example 5: Multiple Elements**
```javascript
const counter = ReactiveUtils.reactive({ count: 0 })
  .bind({
    '.counter-display': 'count' // Updates all elements with class
  })
  .build();
```

### **Example 6: Conditional Visibility**
```javascript
const app = ReactiveUtils.reactive({
  showMessage: false,
  message: 'Hello'
})
.bind({
  '#message': {
    textContent: 'message',
    style: function() {
      return `display: ${this.showMessage ? 'block' : 'none'}`;
    }
  }
})
.build();

app.showMessage = true; // Shows message
```

### **Example 7: Form Inputs**
```javascript
const form = ReactiveUtils.reactive({
  username: '',
  email: ''
})
.bind({
  '#username': { value: 'username' },
  '#email': { value: 'email' }
})
.build();

// Note: This is one-way binding (state -> DOM)
// For two-way, use event listeners
```

### **Example 8: Styling**
```javascript
const theme = ReactiveUtils.reactive({
  primaryColor: '#007bff',
  fontSize: 16
})
.bind({
  'body': {
    style: function() {
      return `
        --primary-color: ${this.primaryColor};
        font-size: ${this.fontSize}px;
      `;
    }
  }
})
.build();

theme.primaryColor = '#ff0000';
```

### **Example 9: List Rendering**
```javascript
const todos = ReactiveUtils.reactive({
  items: ['Task 1', 'Task 2', 'Task 3']
})
.bind({
  '#todo-list': {
    innerHTML: function() {
      return this.items.map(item =>
        `<li>${item}</li>`
      ).join('');
    }
  }
})
.build();

todos.items.push('Task 4'); // List updates
```

### **Example 10: Attributes**
```javascript
const link = ReactiveUtils.reactive({
  url: 'https://example.com',
  title: 'Example Site'
})
.bind({
  '#link': {
    href: 'url',
    title: 'title',
    textContent: 'title'
  }
})
.build();

link.url = 'https://newsite.com';
```

---

## **Binding Syntax**

### **Simple (String)**
```javascript
.bind({
  '#element': 'propertyName'
})
```

### **Computed (Function)**
```javascript
.bind({
  '#element': function() {
    return this.prop1 + this.prop2;
  }
})
```

### **Multiple Properties (Object)**
```javascript
.bind({
  '#element': {
    textContent: 'message',
    className: 'status',
    disabled: function() { return !this.isActive; }
  }
})
```

---

## **Common Patterns**

### **Pattern 1: Text Content**
```javascript
.bind({
  '#output': 'message'
})
```

### **Pattern 2: Attributes**
```javascript
.bind({
  '#link': { href: 'url' }
})
```

### **Pattern 3: Styling**
```javascript
.bind({
  '#box': { style: () => `color: ${state.color}` }
})
```

### **Pattern 4: Classes**
```javascript
.bind({
  '#btn': { className: () => state.active ? 'active' : '' }
})
```

### **Pattern 5: Visibility**
```javascript
.bind({
  '#modal': {
    style: () => `display: ${state.show ? 'block' : 'none'}`
  }
})
```

---

## **Best Practices**

1. **Use for output only**
   ```javascript
   // Good: State -> DOM
   .bind({ '#output': 'value' })

   // For input, add event listeners separately
   ```

2. **Keep binding functions simple**
   ```javascript
   .bind({
     '#name': function() {
       return `${this.first} ${this.last}`;
     }
   })
   ```

3. **Use computed for complex logic**
   ```javascript
   .computed({
     displayName: function() { return `${this.first} ${this.last}`; }
   })
   .bind({
     '#name': 'displayName'
   })
   ```

4. **Batch updates**
   ```javascript
   .bind({
     '#element': {
       textContent: 'text',
       className: 'cssClass',
       disabled: 'isDisabled'
     }
   })
   ```

5. **Validate selectors**
   ```javascript
   // Ensure elements exist before binding
   if (querySelector('#myElement')) {
     builder.bind({ '#myElement': 'value' });
   }
   ```

---

## **Key Takeaways**

1. **Auto-Update**: DOM updates when state changes
2. **Declarative**: Simple selector-to-property mapping
3. **One-Way**: State -> DOM (not DOM -> state)
4. **Multiple**: Bind many elements at once
5. **Flexible**: String, function, or object bindings
6. **Chainable**: Returns builder for chaining
7. **Auto-Cleanup**: Bindings cleaned up on destroy
8. **Efficient**: Only updates when needed
9. **Side-Effect**: Creates effects internally
10. **Output-Focused**: Best for displaying data

---

## **Summary**

`builder.bind(defs)` is a builder method that creates automatic DOM bindings, synchronizing reactive state with DOM elements. Bindings can be simple property mappings, computed functions, or objects mapping multiple properties to element attributes. When state changes, bound DOM elements automatically update. Bindings are one-way (state to DOM) and are ideal for displaying reactive data. The method accepts an object where keys are CSS selectors and values define what to bind. Use string values for simple property bindings, functions for computed bindings, and objects for binding multiple properties to different element attributes. Bindings are automatically cleaned up when the component is destroyed. For two-way data binding with form inputs, combine bindings with event listeners.
