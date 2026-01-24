# `form.isSubmitting` - Form Submission State Property

**Quick Start (30 seconds)**
```javascript
const form = Forms.createForm({ email: '', password: '' }, {
  async onSubmit(values) {
    // Simulate API call
    await new Promise(resolve => setTimeout(resolve, 2000));
    return { success: true };
  }
});

console.log(form.isSubmitting); // false (not submitting)

form.submit();
console.log(form.isSubmitting); // true (submitting)

// After 2 seconds...
console.log(form.isSubmitting); // false (submission complete)

// React to submission state
effect(() => {
  submitButton.disabled = form.isSubmitting;
  submitButton.textContent = form.isSubmitting ? 'Submitting...' : 'Submit';
});
```

---

## **What is `form.isSubmitting`?**

`form.isSubmitting` is a **reactive state property** that indicates whether the form is currently being submitted. It's automatically set to `true` when `form.submit()` is called and back to `false` when the submission completes (successfully or with error).

**Key characteristics:**
- **Reactive**: Changes trigger effects and re-renders
- **Boolean**: Always `true` (submitting) or `false` (not submitting)
- **Automatic**: Managed automatically by `form.submit()`
- **Direct Access**: Can read and write the value
- **Initially False**: Starts as `false` when form is created

---

## **Syntax**

```javascript
// Read submission state
const isSubmitting = form.isSubmitting;

// Write submission state (rare, usually automatic)
form.isSubmitting = true;
form.isSubmitting = false;

// Automatic update via submit()
await form.submit(); // Sets to true, then false when done
```

### **Type**
```javascript
form.isSubmitting: boolean
```

---

## **Why does `form.isSubmitting` exist?**

### ❌ **The Problem**
```javascript
// Without form.isSubmitting, you'd manually track submission state
let isSubmitting = false;

async function handleSubmit() {
  isSubmitting = true;
  updateUI(); // Update submit button

  try {
    await fetch('/api/submit', {
      method: 'POST',
      body: JSON.stringify(formData)
    });
  } finally {
    isSubmitting = false;
    updateUI(); // Update submit button again
  }
}

// Tedious and error-prone
```

### ✅ **The Solution**
```javascript
// Automatic submission state management
const form = Forms.createForm({ email: '', password: '' }, {
  async onSubmit(values) {
    await fetch('/api/submit', {
      method: 'POST',
      body: JSON.stringify(values)
    });
  }
});

// State is managed automatically
await form.submit();

// Reactive UI updates
effect(() => {
  submitButton.disabled = form.isSubmitting;
  submitButton.textContent = form.isSubmitting ? 'Submitting...' : 'Submit';
});
```

---

## **Mental Model**

Think of `form.isSubmitting` as a **traffic light** that controls form submission flow:

```
┌─────────────────────────────────────────────┐
│       FORM SUBMISSION LIFECYCLE             │
├─────────────────────────────────────────────┤
│                                             │
│  1. Initial State                           │
│     isSubmitting: false        🟢 Ready     │
│                                             │
│  2. form.submit() called                    │
│     isSubmitting: true         🔴 Busy      │
│                                             │
│  3. API call in progress                    │
│     isSubmitting: true         🔴 Busy      │
│                                             │
│  4. Submission complete                     │
│     isSubmitting: false        🟢 Ready     │
│                                             │
└─────────────────────────────────────────────┘
```

**The submission flow:**
```
submit() → isSubmitting = true → API Call → Complete → isSubmitting = false
                ↓                                              ↓
          Disable Button                                 Enable Button
          Show Spinner                                   Hide Spinner
```

---

## **How does it work?**

When you create a form, the `isSubmitting` property is initialized and made reactive:

```javascript
const formObj = {
  values: { ...initialValues },
  errors: {},
  touched: {},
  isSubmitting: false,  // Initial state
  submitCount: 0
};

const form = createState(formObj); // Made reactive
```

When `form.submit()` is called:
```javascript
async submit() {
  this.isSubmitting = true;  // Set to true

  try {
    await onSubmitCallback(this.values);
    this.submitCount++;
  } catch (error) {
    // Handle error
  } finally {
    this.isSubmitting = false;  // Set back to false
  }

  return this;
}
```

---

## **Examples**

### **Example 1: Disable Submit Button During Submission**
```javascript
const form = Forms.createForm({ email: '', password: '' }, {
  async onSubmit(values) {
    const response = await fetch('/api/login', {
      method: 'POST',
      body: JSON.stringify(values)
    });
    return response.json();
  }
});

const submitBtn = document.querySelector('#submit');

effect(() => {
  submitBtn.disabled = form.isSubmitting;
});

// Button is enabled initially
// When form.submit() is called, button disables automatically
// When submission completes, button re-enables
```

### **Example 2: Show Loading Spinner**
```javascript
const form = Forms.createForm({ email: '' }, {
  async onSubmit(values) {
    await fetch('/api/subscribe', {
      method: 'POST',
      body: JSON.stringify(values)
    });
  }
});

effect(() => {
  const spinner = document.querySelector('#loading-spinner');
  const submitBtn = document.querySelector('#submit');

  if (form.isSubmitting) {
    spinner.style.display = 'inline-block';
    submitBtn.textContent = 'Submitting...';
    submitBtn.disabled = true;
  } else {
    spinner.style.display = 'none';
    submitBtn.textContent = 'Submit';
    submitBtn.disabled = false;
  }
});
```

### **Example 3: Prevent Double Submission**
```javascript
const form = Forms.createForm({ email: '' }, {
  async onSubmit(values) {
    await fetch('/api/submit', {
      method: 'POST',
      body: JSON.stringify(values)
    });
  }
});

async function handleSubmit(e) {
  e.preventDefault();

  // Prevent double submission
  if (form.isSubmitting) {
    console.log('Already submitting, please wait...');
    return;
  }

  await form.submit();
}
```

### **Example 4: Submit Button Text Changes**
```javascript
const form = Forms.createForm({ name: '', email: '' }, {
  async onSubmit(values) {
    await new Promise(resolve => setTimeout(resolve, 2000));
  }
});

effect(() => {
  const submitBtn = document.querySelector('#submit');

  if (form.isSubmitting) {
    submitBtn.innerHTML = `
      <span class="spinner"></span>
      Please wait...
    `;
  } else if (form.submitCount > 0) {
    submitBtn.textContent = 'Submit Again';
  } else {
    submitBtn.textContent = 'Submit';
  }
});
```

### **Example 5: Disable Form Inputs During Submission**
```javascript
const form = Forms.createForm({
  firstName: '',
  lastName: '',
  email: ''
}, {
  async onSubmit(values) {
    await fetch('/api/register', {
      method: 'POST',
      body: JSON.stringify(values)
    });
  }
});

effect(() => {
  const inputs = document.querySelectorAll('input');

  inputs.forEach(input => {
    input.disabled = form.isSubmitting;
  });
});
```

### **Example 6: Show Progress Overlay**
```javascript
const form = Forms.createForm({ data: '' }, {
  async onSubmit(values) {
    await uploadLargeFile(values.data);
  }
});

effect(() => {
  const overlay = document.querySelector('#submit-overlay');

  if (form.isSubmitting) {
    overlay.classList.add('visible');
    overlay.innerHTML = `
      <div class="spinner"></div>
      <p>Uploading... Please don't close this window.</p>
    `;
  } else {
    overlay.classList.remove('visible');
  }
});
```

### **Example 7: Track Submission Attempts**
```javascript
const form = Forms.createForm({ email: '' }, {
  async onSubmit(values) {
    const response = await fetch('/api/submit', {
      method: 'POST',
      body: JSON.stringify(values)
    });

    if (!response.ok) {
      throw new Error('Submission failed');
    }

    return response.json();
  }
});

effect(() => {
  const status = document.querySelector('#status');

  if (form.isSubmitting) {
    status.textContent = `Submitting... (Attempt ${form.submitCount + 1})`;
  } else if (form.submitCount > 0) {
    status.textContent = `Submitted ${form.submitCount} time(s)`;
  } else {
    status.textContent = 'Ready to submit';
  }
});
```

### **Example 8: Conditional Navigation Blocking**
```javascript
const form = Forms.createForm({ content: '' }, {
  async onSubmit(values) {
    await fetch('/api/save', {
      method: 'POST',
      body: JSON.stringify(values)
    });
  }
});

window.addEventListener('beforeunload', (e) => {
  if (form.isSubmitting) {
    e.preventDefault();
    e.returnValue = 'Form is being submitted, please wait...';
    return 'Form is being submitted, please wait...';
  }
});
```

### **Example 9: Multi-Step Form Submission**
```javascript
const form = Forms.createForm({
  step1: '',
  step2: '',
  step3: ''
}, {
  async onSubmit(values) {
    // Submit all steps
    await fetch('/api/multi-step', {
      method: 'POST',
      body: JSON.stringify(values)
    });
  }
});

effect(() => {
  const steps = document.querySelectorAll('.step');
  const submitBtn = document.querySelector('#submit');

  if (form.isSubmitting) {
    // Disable all step navigation
    steps.forEach(step => {
      step.classList.add('disabled');
    });

    submitBtn.innerHTML = `
      <span class="spinner"></span>
      Finalizing...
    `;
  } else {
    steps.forEach(step => {
      step.classList.remove('disabled');
    });

    submitBtn.innerHTML = 'Submit';
  }
});
```

### **Example 10: Error Handling with Submission State**
```javascript
const form = Forms.createForm({ email: '' }, {
  async onSubmit(values) {
    const response = await fetch('/api/subscribe', {
      method: 'POST',
      body: JSON.stringify(values)
    });

    if (!response.ok) {
      throw new Error('Subscription failed');
    }

    return response.json();
  }
});

async function handleSubmit() {
  try {
    await form.submit();
    alert('Subscription successful!');
  } catch (error) {
    // form.isSubmitting is already false here
    alert('Subscription failed: ' + error.message);
  }
}

effect(() => {
  const submitBtn = document.querySelector('#submit');
  const message = document.querySelector('#message');

  if (form.isSubmitting) {
    submitBtn.disabled = true;
    submitBtn.textContent = 'Subscribing...';
    message.textContent = 'Please wait while we process your subscription...';
  } else {
    submitBtn.disabled = false;
    submitBtn.textContent = 'Subscribe';
    message.textContent = '';
  }
});
```

---

## **Common Patterns**

### **Pattern 1: Disable Button During Submission**
```javascript
effect(() => {
  submitButton.disabled = form.isSubmitting;
});
```

### **Pattern 2: Change Button Text**
```javascript
effect(() => {
  submitButton.textContent = form.isSubmitting ? 'Submitting...' : 'Submit';
});
```

### **Pattern 3: Show Loading Spinner**
```javascript
effect(() => {
  spinner.style.display = form.isSubmitting ? 'block' : 'none';
});
```

### **Pattern 4: Prevent Double Submit**
```javascript
async function handleSubmit() {
  if (form.isSubmitting) return;
  await form.submit();
}
```

### **Pattern 5: Disable All Inputs**
```javascript
effect(() => {
  document.querySelectorAll('input, button').forEach(el => {
    el.disabled = form.isSubmitting;
  });
});
```

---

## **Comparison with Related Properties**

| Property | Type | What it indicates |
|----------|------|------------------|
| `form.isSubmitting` | Boolean | Currently submitting |
| `form.submitCount` | Number | How many times submitted |
| `form.isValid` | Boolean | Form is valid |
| `form.isDirty` | Boolean | Form has changes |

```javascript
// Example states
form.isSubmitting;   // false → true → false (during submit)
form.submitCount;    // 0 → 1 → 2 → ... (increases with each submit)
form.isValid;        // true/false (validation state)
form.isDirty;        // true/false (touched state)
```

---

## **Submission State Timeline**

```
Time →

0ms:   User clicks submit
       form.isSubmitting = true
       Button disabled
       Spinner shown

100ms: API request sent
       Still submitting...

2000ms: API responds
        form.isSubmitting = false
        Button re-enabled
        Spinner hidden
        form.submitCount++
```

---

## **Key Takeaways**

1. **Reactive Boolean**: `isSubmitting` automatically updates during submission
2. **Automatic Management**: Set by `form.submit()`, no manual control needed
3. **Prevents Double Submission**: Easy to check before submitting
4. **UI Feedback**: Perfect for disabling buttons, showing spinners
5. **Initially False**: Starts as `false` when form is created
6. **True During Async**: Remains `true` throughout async submission
7. **False on Complete**: Set to `false` whether success or error
8. **Common Use**: Button states, loading indicators, input disabling

---

## **Summary**

`form.isSubmitting` is a reactive state property that indicates whether the form is currently being submitted. It's automatically managed by the `form.submit()` method, setting to `true` when submission starts and `false` when it completes. This property is essential for creating better user experiences by disabling submit buttons to prevent double submissions, showing loading spinners to provide feedback, and disabling form inputs to prevent changes during submission. Use it in reactive effects to automatically update your UI based on submission state without manual state management.
