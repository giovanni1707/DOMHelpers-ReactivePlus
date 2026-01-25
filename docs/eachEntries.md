# `eachEntries(obj, callback, selector)` - Iterate Over Object Entries

**Quick Start (30 seconds)**
```javascript
const user = {
  name: 'Alice',
  age: 30,
  email: 'alice@example.com'
};

// Iterate over entries
eachEntries(user, (key, value, index) => {
  console.log(`${key}: ${value}`);
});
// Output:
// "name: Alice"
// "age: 30"
// "email: alice@example.com"

// Generate HTML
const html = eachEntries(user, (key, value) => {
  return `<div><strong>${key}:</strong> ${value}</div>`;
});
console.log(html);
// "<div><strong>name:</strong> Alice</div>
//  <div><strong>age:</strong> 30</div>
//  <div><strong>email:</strong> alice@example.com</div>"

// Render directly to DOM
eachEntries(user, (key, value) => {
  return `<div><strong>${key}:</strong> ${value}</div>`;
}, '#output');
// Renders HTML to element with id="output"
```

---

## **What is `eachEntries(obj, callback, selector)`?**

`eachEntries()` is a **utility function** that iterates over an object's entries using `Object.entries()` and calls a callback for each key-value pair, with optional HTML generation and direct DOM rendering.

**Key characteristics:**
- **Iterates Entries**: Loops over object key-value pairs
- **Callback Per Entry**: Calls function for each entry
- **HTML Generation**: Can accumulate HTML strings
- **DOM Rendering**: Can render directly to selector
- **Index Provided**: Passes iteration index
- **Returns HTML**: Returns accumulated HTML if callback returns strings
- **Side-Effect Option**: Can render to DOM without returning

---

## **Syntax**

```javascript
eachEntries(obj, callback, selector)
```

### **Parameters**

**`obj`** (object, required)
- Object to iterate over
- Must be an object (not null, not primitive)

**`callback`** (function, required)
- Function called for each entry
- Signature: `(key, value, index) => any`
- If returns string, accumulated as HTML
- If returns undefined, no HTML accumulated

**`selector`** (string, optional)
- CSS selector for DOM element
- If provided, renders HTML to this element
- Example: `'#output'`, `'.container'`, `'div.results'`

### **Returns**
- **Type**: `string` or `undefined`
- Returns accumulated HTML if callback returns strings
- Returns `undefined` if callback doesn't return anything

---

## **How it works**

```javascript
function eachEntries(obj, callback, selector) {
  // 1. Validate object
  if (obj === null || typeof obj !== 'object') {
    console.warn('eachEntries: First argument must be an object');
    return '';
  }

  // 2. Iterate over entries
  let html = '';
  let isReturningHTML = false;

  Object.entries(obj).forEach(([key, value], index) => {
    const result = callback(key, value, index);

    // 3. Accumulate HTML if callback returns string
    if (result !== undefined) {
      html += result;
      isReturningHTML = true;
    }
  });

  const output = isReturningHTML ? html : undefined;

  // 4. Render to DOM if selector provided
  if (selector && typeof selector === 'string') {
    const element = querySelector(selector);
    if (element) {
      element.innerHTML = output || '';
    }
  }

  // 5. Return accumulated HTML
  return output;
}
```

**What happens:**
1. Validates object argument
2. Iterates using `Object.entries()`
3. Calls callback for each entry with (key, value, index)
4. Accumulates HTML if callback returns strings
5. Renders to DOM if selector provided
6. Returns accumulated HTML or undefined

---

## **Examples**

### **Example 1: Simple Iteration**
```javascript
const config = {
  theme: 'dark',
  fontSize: 16,
  language: 'en'
};

eachEntries(config, (key, value) => {
  console.log(`${key} = ${value}`);
});
// Output:
// "theme = dark"
// "fontSize = 16"
// "language = en"
```

### **Example 2: Generate HTML**
```javascript
const product = {
  name: 'Laptop',
  price: 999,
  stock: 5
};

const html = eachEntries(product, (key, value) => {
  return `<li>${key}: ${value}</li>`;
});

console.log(html);
// "<li>name: Laptop</li><li>price: 999</li><li>stock: 5</li>"

// Wrap in ul
querySelector('#product').innerHTML = `<ul>${html}</ul>`;
```

### **Example 3: Render to DOM**
```javascript
const user = {
  username: 'alice',
  email: 'alice@example.com',
  role: 'admin'
};

eachEntries(user, (key, value) => {
  return `
    <div class="field">
      <label>${key}:</label>
      <span>${value}</span>
    </div>
  `;
}, '#userDetails');

// Rendered directly to <div id="userDetails"></div>
```

### **Example 4: With Index**
```javascript
const tasks = {
  task1: 'Write documentation',
  task2: 'Review code',
  task3: 'Deploy app'
};

eachEntries(tasks, (key, value, index) => {
  return `<p>${index + 1}. ${value}</p>`;
}, '#tasks');

// Rendered:
// <p>1. Write documentation</p>
// <p>2. Review code</p>
// <p>3. Deploy app</p>
```

### **Example 5: Settings Panel**
```javascript
const settings = {
  notifications: true,
  darkMode: false,
  autoSave: true,
  language: 'en'
};

eachEntries(settings, (key, value) => {
  const type = typeof value === 'boolean' ? 'checkbox' : 'text';
  const checked = value === true ? 'checked' : '';

  return `
    <div class="setting">
      <label>${key}:</label>
      <input type="${type}" name="${key}" ${checked} value="${value}">
    </div>
  `;
}, '#settings');
```

### **Example 6: Data Table**
```javascript
const stats = {
  users: 1234,
  posts: 5678,
  comments: 9012,
  likes: 34567
};

const rows = eachEntries(stats, (key, value, index) => {
  const rowClass = index % 2 === 0 ? 'even' : 'odd';
  return `
    <tr class="${rowClass}">
      <td>${key}</td>
      <td>${value.toLocaleString()}</td>
    </tr>
  `;
});

querySelector('#statsTable').innerHTML = `
  <table>
    <thead>
      <tr><th>Metric</th><th>Count</th></tr>
    </thead>
    <tbody>${rows}</tbody>
  </table>
`;
```

### **Example 7: Side Effects Only**
```javascript
const errors = {
  email: 'Invalid email format',
  password: 'Password too short',
  username: 'Username already taken'
};

// Just log, don't return HTML
eachEntries(errors, (key, value) => {
  console.error(`${key}: ${value}`);
  // No return - no HTML generated
});

// Returns undefined (no HTML)
```

### **Example 8: Conditional Rendering**
```javascript
const permissions = {
  read: true,
  write: true,
  delete: false,
  admin: false
};

eachEntries(permissions, (key, value) => {
  if (value) {
    return `<span class="permission granted">✓ ${key}</span>`;
  } else {
    return `<span class="permission denied">✗ ${key}</span>`;
  }
}, '#permissions');
```

### **Example 9: Form Builder**
```javascript
const formFields = {
  name: 'text',
  email: 'email',
  age: 'number',
  bio: 'textarea'
};

eachEntries(formFields, (key, type) => {
  if (type === 'textarea') {
    return `
      <div class="field">
        <label for="${key}">${key}:</label>
        <textarea id="${key}" name="${key}"></textarea>
      </div>
    `;
  } else {
    return `
      <div class="field">
        <label for="${key}">${key}:</label>
        <input type="${type}" id="${key}" name="${key}">
      </div>
    `;
  }
}, '#form');
```

### **Example 10: Object to List**
```javascript
const features = {
  'Dark Mode': 'Toggle dark/light theme',
  'Auto Save': 'Automatically save your work',
  'Cloud Sync': 'Sync across devices',
  'Offline Mode': 'Work without internet'
};

eachEntries(features, (name, description) => {
  return `
    <div class="feature">
      <h3>${name}</h3>
      <p>${description}</p>
    </div>
  `;
}, '.features-list');
```

---

## **Common Patterns**

### **Pattern 1: Simple Iteration**
```javascript
eachEntries(obj, (key, value) => {
  console.log(key, value);
});
```

### **Pattern 2: Generate HTML**
```javascript
const html = eachEntries(obj, (key, value) => {
  return `<div>${key}: ${value}</div>`;
});
```

### **Pattern 3: Render to DOM**
```javascript
eachEntries(obj, (key, value) => {
  return `<div>${key}: ${value}</div>`;
}, '#output');
```

### **Pattern 4: With Index**
```javascript
eachEntries(obj, (key, value, index) => {
  return `<div>${index + 1}. ${key}: ${value}</div>`;
});
```

### **Pattern 5: Conditional Content**
```javascript
eachEntries(obj, (key, value) => {
  if (shouldShow(value)) {
    return `<div>${key}</div>`;
  }
  // Return nothing for filtered items
});
```

---

## **When to Use**

| Scenario | Use eachEntries() |
|----------|-------------------|
| Iterate over object entries | ✓ Yes |
| Generate HTML from object | ✓ Yes |
| Render settings/config | ✓ Yes |
| Build forms from schema | ✓ Yes |
| Display object properties | ✓ Yes |
| Need transformation | ⚠ Use mapEntries() |
| Iterate array | ✗ Use forEach/map |
| Transform to array | ✗ Use mapEntries() |

---

## **vs. Object.entries().forEach()**

| Feature | `eachEntries()` | `Object.entries().forEach()` |
|---------|----------------|------------------------------|
| HTML accumulation | ✓ Built-in | ✗ Manual |
| DOM rendering | ✓ Built-in | ✗ Manual |
| Index parameter | ✓ Yes | ✓ Yes (manual) |
| Callback signature | `(key, value, index)` | `([key, value], index)` |
| Return value | HTML string | undefined |

```javascript
// eachEntries - convenient for HTML
const html = eachEntries(obj, (key, value) => {
  return `<div>${key}: ${value}</div>`;
}, '#output');

// Object.entries - manual HTML
let html = '';
Object.entries(obj).forEach(([key, value]) => {
  html += `<div>${key}: ${value}</div>`;
});
querySelector('#output').innerHTML = html;
```

---

## **Callback Signature**

```javascript
function callback(key, value, index) {
  // key: property name (string)
  // value: property value (any type)
  // index: iteration index (number, 0-based)

  // Return string to accumulate HTML
  return `<div>${key}: ${value}</div>`;

  // Or return nothing for side effects only
  // (no HTML accumulated)
}
```

---

## **Error Handling**

### **Invalid Object**
```javascript
eachEntries(null, callback);
// Warning: "eachEntries: First argument must be an object"
// Returns: ''

eachEntries('string', callback);
// Warning: "eachEntries: First argument must be an object"
// Returns: ''

eachEntries(123, callback);
// Warning: "eachEntries: First argument must be an object"
// Returns: ''
```

### **Invalid Selector**
```javascript
eachEntries(obj, callback, '#missing');
// Warning: "eachEntries: Element not found for selector '#missing'"
// HTML not rendered, but still returned

eachEntries(obj, callback, 'invalid>>selector');
// Warning: "eachEntries: Invalid selector..."
// HTML not rendered, but still returned
```

---

## **Best Practices**

1. **Use for HTML generation**
   ```javascript
   const html = eachEntries(data, (key, value) => {
     return `<div>${key}: ${value}</div>`;
   });
   ```

2. **Render directly when possible**
   ```javascript
   eachEntries(data, (key, value) => {
     return `<div>${key}: ${value}</div>`;
   }, '#output');
   ```

3. **Escape HTML in values**
   ```javascript
   eachEntries(user, (key, value) => {
     const safe = escapeHTML(value);
     return `<div>${key}: ${safe}</div>`;
   });
   ```

4. **Handle different types**
   ```javascript
   eachEntries(obj, (key, value) => {
     if (typeof value === 'object') {
       return `<div>${key}: ${JSON.stringify(value)}</div>`;
     }
     return `<div>${key}: ${value}</div>`;
   });
   ```

5. **Use index for styling**
   ```javascript
   eachEntries(items, (key, value, index) => {
     const className = index % 2 === 0 ? 'even' : 'odd';
     return `<div class="${className}">${key}: ${value}</div>`;
   });
   ```

---

## **Access Locations**

This function is available globally:

```javascript
// Global function
eachEntries(obj, callback);

// May also be available in ReactiveUtils (check your version)
ReactiveUtils.eachEntries?.(obj, callback);
```

---

## **Key Takeaways**

1. **Iterates Entries**: Loops over object key-value pairs
2. **Callback Per Entry**: Calls function with (key, value, index)
3. **HTML Generation**: Accumulates returned strings
4. **DOM Rendering**: Can render directly to selector
5. **Index Included**: Provides iteration index (0-based)
6. **Returns HTML**: Returns accumulated HTML string
7. **Side Effects**: Can use without returning HTML
8. **Object.entries**: Uses native Object.entries() internally
9. **Error Safe**: Validates object and selector
10. **Convenient**: Simplifies object-to-HTML patterns

---

## **Summary**

`eachEntries(obj, callback, selector)` is a utility function that iterates over an object's entries using `Object.entries()` and calls a callback for each key-value pair, with optional HTML generation and direct DOM rendering. When called, it validates the object argument, loops through each entry calling the callback with (key, value, index), accumulates any returned strings as HTML, and optionally renders the accumulated HTML to a DOM element specified by the selector. The function returns the accumulated HTML string if the callback returns strings, or undefined if the callback doesn't return anything (for side-effects only). Use `eachEntries()` for displaying object properties, building forms from schemas, rendering settings panels, generating lists from objects, or any scenario where you need to iterate over an object and generate HTML. The function provides a convenient alternative to manually using `Object.entries().forEach()` with HTML accumulation, especially when you need to render the result directly to the DOM.
