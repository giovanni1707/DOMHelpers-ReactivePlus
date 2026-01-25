# `mapEntries(obj, callback, joinHTML, selector)` - Map Over Object Entries

**Quick Start (30 seconds)**
```javascript
const user = {
  name: 'Alice',
  age: 30,
  email: 'alice@example.com'
};

// Map to array of strings
const fields = mapEntries(user, (key, value) => {
  return `${key}: ${value}`;
});
console.log(fields);
// ['name: Alice', 'age: 30', 'email: alice@example.com']

// Map and join as HTML
const html = mapEntries(user, (key, value) => {
  return `<div>${key}: ${value}</div>`;
}, true);
console.log(html);
// "<div>name: Alice</div><div>age: 30</div><div>email: alice@example.com</div>"

// Map and render directly to DOM
mapEntries(user, (key, value) => {
  return `<div><strong>${key}:</strong> ${value}</div>`;
}, '#output');
// Renders HTML to element with id="output"
```

---

## **What is `mapEntries(obj, callback, joinHTML, selector)`?**

`mapEntries()` is a **utility function** that maps over an object's entries using `Object.entries()` and transforms each key-value pair, returning an array of transformed values or joined HTML string, with optional direct DOM rendering.

**Key characteristics:**
- **Maps Entries**: Transforms object key-value pairs
- **Returns Array**: Returns array of transformed values
- **Optional Join**: Can join array as HTML string
- **DOM Rendering**: Can render directly to selector
- **Index Provided**: Passes iteration index
- **Transformation**: Each callback result becomes array element
- **Flexible**: Array or HTML string output

---

## **Syntax**

```javascript
mapEntries(obj, callback, joinHTML, selector)
```

### **Parameters**

**`obj`** (object, required)
- Object to map over
- Must be an object (not null, not primitive)

**`callback`** (function, required)
- Function called for each entry
- Signature: `(key, value, index) => any`
- Return value becomes array element

**`joinHTML`** (boolean or string, optional)
- If `true`: joins array as HTML string
- If `false` or omitted: returns array
- If string: treated as selector, joins as HTML and renders

**`selector`** (string, optional)
- CSS selector for DOM element (when joinHTML is boolean)
- Renders joined HTML to this element
- Example: `'#output'`, `'.container'`

### **Returns**
- **Type**: `Array` or `string`
- Returns array if `joinHTML` is false/omitted
- Returns joined HTML string if `joinHTML` is true
- Returns joined HTML string if `joinHTML` is a selector string

---

## **How it works**

```javascript
function mapEntries(obj, callback, joinHTMLOrSelector, selector) {
  // 1. Validate object
  if (obj === null || typeof obj !== 'object') {
    console.warn('mapEntries: First argument must be an object');
    return joinHTMLOrSelector ? '' : [];
  }

  // 2. Map over entries
  const result = Object.entries(obj).map(([key, value], index) => {
    return callback(key, value, index);
  });

  // 3. Determine if should join and render
  let joinHTML = false;
  let targetSelector = null;

  if (typeof joinHTMLOrSelector === 'boolean') {
    joinHTML = joinHTMLOrSelector;
    targetSelector = selector;
  } else if (typeof joinHTMLOrSelector === 'string') {
    joinHTML = true;
    targetSelector = joinHTMLOrSelector;
  }

  // 4. Join if requested
  const output = joinHTML ? result.join('') : result;

  // 5. Render to DOM if selector provided
  if (targetSelector) {
    const element = document.querySelector(targetSelector);
    if (element) {
      element.innerHTML = joinHTML ? output : output.join('');
    }
  }

  // 6. Return array or HTML string
  return output;
}
```

**What happens:**
1. Validates object argument
2. Maps using `Object.entries()` and `Array.map()`
3. Calls callback for each entry with (key, value, index)
4. Returns array of transformed values
5. Optionally joins array as HTML string
6. Optionally renders to DOM element
7. Returns array or HTML string

---

## **Examples**

### **Example 1: Map to Array**
```javascript
const config = {
  theme: 'dark',
  fontSize: 16,
  language: 'en'
};

const pairs = mapEntries(config, (key, value) => {
  return `${key}=${value}`;
});

console.log(pairs);
// ['theme=dark', 'fontSize=16', 'language=en']
```

### **Example 2: Map and Join**
```javascript
const product = {
  name: 'Laptop',
  price: 999,
  stock: 5
};

const html = mapEntries(product, (key, value) => {
  return `<li>${key}: ${value}</li>`;
}, true);

console.log(html);
// "<li>name: Laptop</li><li>price: 999</li><li>stock: 5</li>"

document.getElementById('product').innerHTML = `<ul>${html}</ul>`;
```

### **Example 3: Map and Render**
```javascript
const user = {
  username: 'alice',
  email: 'alice@example.com',
  role: 'admin'
};

mapEntries(user, (key, value) => {
  return `
    <div class="field">
      <label>${key}:</label>
      <span>${value}</span>
    </div>
  `;
}, '#userDetails');

// Joined and rendered to <div id="userDetails"></div>
```

### **Example 4: Transform to Objects**
```javascript
const settings = {
  notifications: true,
  darkMode: false,
  language: 'en'
};

const settingsArray = mapEntries(settings, (key, value) => {
  return {
    name: key,
    value: value,
    type: typeof value
  };
});

console.log(settingsArray);
// [
//   { name: 'notifications', value: true, type: 'boolean' },
//   { name: 'darkMode', value: false, type: 'boolean' },
//   { name: 'language', value: 'en', type: 'string' }
// ]
```

### **Example 5: Map with Index**
```javascript
const tasks = {
  task1: 'Write docs',
  task2: 'Review code',
  task3: 'Deploy app'
};

const html = mapEntries(tasks, (key, value, index) => {
  return `<p>${index + 1}. ${value}</p>`;
}, true);

document.getElementById('tasks').innerHTML = html;
// <p>1. Write docs</p>
// <p>2. Review code</p>
// <p>3. Deploy app</p>
```

### **Example 6: Filter and Map**
```javascript
const permissions = {
  read: true,
  write: true,
  delete: false,
  admin: false
};

const granted = mapEntries(permissions, (key, value) => {
  if (value) {
    return `<span class="granted">✓ ${key}</span>`;
  }
  return null;
}).filter(item => item !== null);

console.log(granted);
// ['<span class="granted">✓ read</span>', '<span class="granted">✓ write</span>']
```

### **Example 7: Build Options**
```javascript
const countries = {
  us: 'United States',
  uk: 'United Kingdom',
  ca: 'Canada',
  au: 'Australia'
};

const options = mapEntries(countries, (code, name) => {
  return `<option value="${code}">${name}</option>`;
}, true);

document.getElementById('country').innerHTML = options;
```

### **Example 8: Stats Dashboard**
```javascript
const stats = {
  users: 1234,
  posts: 5678,
  comments: 9012,
  likes: 34567
};

const cards = mapEntries(stats, (label, count) => {
  return `
    <div class="stat-card">
      <h3>${label}</h3>
      <p class="count">${count.toLocaleString()}</p>
    </div>
  `;
}, '.stats-grid');
```

### **Example 9: Form Builder**
```javascript
const schema = {
  name: { type: 'text', required: true },
  email: { type: 'email', required: true },
  age: { type: 'number', required: false },
  bio: { type: 'textarea', required: false }
};

const fields = mapEntries(schema, (name, field) => {
  const required = field.required ? 'required' : '';

  if (field.type === 'textarea') {
    return `
      <div class="field">
        <label for="${name}">${name}:</label>
        <textarea id="${name}" name="${name}" ${required}></textarea>
      </div>
    `;
  } else {
    return `
      <div class="field">
        <label for="${name}">${name}:</label>
        <input type="${field.type}" id="${name}" name="${name}" ${required}>
      </div>
    `;
  }
}, '#form');
```

### **Example 10: Data Transformation**
```javascript
const apiResponse = {
  user_name: 'alice',
  user_email: 'alice@example.com',
  user_role: 'admin'
};

// Transform to clean object
const cleanData = Object.fromEntries(
  mapEntries(apiResponse, (key, value) => {
    const cleanKey = key.replace('user_', '');
    return [cleanKey, value];
  })
);

console.log(cleanData);
// { name: 'alice', email: 'alice@example.com', role: 'admin' }
```

---

## **Common Patterns**

### **Pattern 1: Map to Array**
```javascript
const array = mapEntries(obj, (key, value) => {
  return value;
});
```

### **Pattern 2: Map and Join**
```javascript
const html = mapEntries(obj, (key, value) => {
  return `<div>${key}: ${value}</div>`;
}, true);
```

### **Pattern 3: Map and Render**
```javascript
mapEntries(obj, (key, value) => {
  return `<div>${key}: ${value}</div>`;
}, '#output');
```

### **Pattern 4: Transform with Index**
```javascript
const result = mapEntries(obj, (key, value, index) => {
  return { id: index, key, value };
});
```

### **Pattern 5: Conditional Mapping**
```javascript
const filtered = mapEntries(obj, (key, value) => {
  return shouldInclude(value) ? transform(value) : null;
}).filter(Boolean);
```

---

## **When to Use**

| Scenario | Use mapEntries() |
|----------|------------------|
| Transform object to array | ✓ Yes |
| Generate HTML from object | ✓ Yes |
| Build option lists | ✓ Yes |
| Convert object format | ✓ Yes |
| Map with index | ✓ Yes |
| Just iterate (no transform) | ✗ Use eachEntries() |
| Map array | ✗ Use array.map() |
| Need original object | ✗ Wrong tool |

---

## **vs. eachEntries()**

| Feature | `mapEntries()` | `eachEntries()` |
|---------|---------------|-----------------|
| Purpose | Transform entries | Iterate entries |
| Returns | Array or HTML | HTML or undefined |
| Use case | Transformation | Side effects / iteration |
| Array output | ✓ Yes | ✗ No |

```javascript
// mapEntries - returns array
const items = mapEntries(obj, (k, v) => v);
// ['value1', 'value2', 'value3']

// eachEntries - returns HTML or undefined
const html = eachEntries(obj, (k, v) => `<div>${v}</div>`);
// "<div>value1</div><div>value2</div><div>value3</div>"
```

---

## **vs. Object.entries().map()**

| Feature | `mapEntries()` | `Object.entries().map()` |
|---------|---------------|--------------------------|
| HTML joining | ✓ Built-in | ✗ Manual |
| DOM rendering | ✓ Built-in | ✗ Manual |
| Callback signature | `(key, value, index)` | `([key, value], index)` |
| Convenience | ✓ High | ⚠ Medium |

```javascript
// mapEntries - convenient
const html = mapEntries(obj, (key, value) => {
  return `<div>${key}: ${value}</div>`;
}, '#output');

// Object.entries - manual
const html = Object.entries(obj)
  .map(([key, value]) => `<div>${key}: ${value}</div>`)
  .join('');
document.querySelector('#output').innerHTML = html;
```

---

## **joinHTML Parameter Options**

### **Option 1: No Joining (Default)**
```javascript
const array = mapEntries(obj, (k, v) => v);
// Returns: ['value1', 'value2', 'value3']
```

### **Option 2: Join as HTML (true)**
```javascript
const html = mapEntries(obj, (k, v) => `<div>${v}</div>`, true);
// Returns: "<div>value1</div><div>value2</div><div>value3</div>"
```

### **Option 3: Join and Render (selector string)**
```javascript
mapEntries(obj, (k, v) => `<div>${v}</div>`, '#output');
// Returns: HTML string
// Renders: to #output element
```

### **Option 4: Join and Render with Selector Parameter**
```javascript
mapEntries(obj, (k, v) => `<div>${v}</div>`, true, '#output');
// Returns: HTML string
// Renders: to #output element
```

---

## **Callback Signature**

```javascript
function callback(key, value, index) {
  // key: property name (string)
  // value: property value (any type)
  // index: iteration index (number, 0-based)

  // Return transformed value
  return transformedValue;
}
```

---

## **Error Handling**

### **Invalid Object**
```javascript
mapEntries(null, callback);
// Warning: "mapEntries: First argument must be an object"
// Returns: [] (or '' if joinHTML is true)

mapEntries('string', callback);
// Warning: "mapEntries: First argument must be an object"
// Returns: [] (or '' if joinHTML is true)
```

### **Invalid Selector**
```javascript
mapEntries(obj, callback, '#missing');
// Warning: "mapEntries: Element not found for selector '#missing'"
// HTML not rendered, but still returned

mapEntries(obj, callback, 'invalid>>selector');
// Warning: "mapEntries: Invalid selector..."
// HTML not rendered, but still returned
```

---

## **Best Practices**

1. **Return consistent types**
   ```javascript
   // Good: all strings
   mapEntries(obj, (k, v) => String(v));

   // Bad: mixed types (unless intentional)
   mapEntries(obj, (k, v) => v);
   ```

2. **Use for transformation**
   ```javascript
   const transformed = mapEntries(obj, (k, v) => {
     return { label: k, value: v };
   });
   ```

3. **Join for HTML output**
   ```javascript
   const html = mapEntries(obj, (k, v) => {
     return `<div>${k}: ${v}</div>`;
   }, true);
   ```

4. **Escape HTML values**
   ```javascript
   mapEntries(user, (k, v) => {
     const safe = escapeHTML(v);
     return `<div>${k}: ${safe}</div>`;
   }, '#output');
   ```

5. **Filter unwanted entries**
   ```javascript
   const filtered = mapEntries(obj, (k, v) => {
     return shouldInclude(k) ? v : null;
   }).filter(Boolean);
   ```

---

## **Access Locations**

This function is available globally:

```javascript
// Global function
mapEntries(obj, callback);

// May also be available in ReactiveUtils (check your version)
ReactiveUtils.mapEntries?.(obj, callback);
```

---

## **Key Takeaways**

1. **Maps Entries**: Transforms object key-value pairs
2. **Returns Array**: Default return type is array
3. **Optional Join**: Can join as HTML string
4. **DOM Rendering**: Can render directly to selector
5. **Index Included**: Provides iteration index
6. **Transformation**: Each callback result is array element
7. **Flexible Output**: Array or HTML string
8. **Object.entries**: Uses native Object.entries() internally
9. **Convenient**: Simplifies object transformation patterns
10. **Chainable**: Result is array (can chain array methods)

---

## **Summary**

`mapEntries(obj, callback, joinHTML, selector)` is a utility function that maps over an object's entries using `Object.entries()` and transforms each key-value pair, returning an array of transformed values or optionally a joined HTML string, with optional direct DOM rendering. When called, it validates the object argument, maps through each entry calling the callback with (key, value, index), collects transformed values in an array, and returns the array by default. If the `joinHTML` parameter is `true` or a selector string, the array is joined as an HTML string. If a selector is provided (either as the third parameter string or fourth parameter), the HTML is rendered to the matching DOM element. Use `mapEntries()` for transforming objects to arrays, generating HTML from object data, building option lists, converting object formats, or any scenario where you need to map over an object's entries and potentially render the result. The function provides a more convenient alternative to manually using `Object.entries().map()` with HTML joining and DOM rendering, especially when working with template-based HTML generation.
