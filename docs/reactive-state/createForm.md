# `form()` - Reactive Form State Management

## Quick Start (30 seconds)

```javascript
// Create reactive form with validation
const loginForm = form({
  email: '',
  password: ''
});

// Auto-update DOM and track changes
effect(() => {
  Elements.update({
    submitBtn: {
      disabled: !loginForm.isValid || loginForm.isSubmitting
    },
    saveIndicator: {
      textContent: loginForm.isDirty ? 'Unsaved changes' : 'Saved',
      style: { color: loginForm.isDirty ? 'orange' : 'green' }
    }
  });
});

// Set values using built-in methods
loginForm.$setValue('email', 'alice@example.com');
console.log(loginForm.touched.email); // true
console.log(loginForm.isDirty); // true (computed)
```

**That's it.** Create forms with built-in validation, error tracking, dirty state, and submit handling. No manual field management needed.

---

## What is `form()`?

`form()` creates **reactive form state with built-in field tracking**. It automatically manages form values, errors, touched fields, and validation state.

Think of it as **a smart form assistant** that tracks every field, remembers what's been touched, validates inputs, and tells you when the form is ready to submit.

**In practical terms:** Instead of manually tracking `values`, `errors`, `touched`, and `isValid` separately, you get a pre-configured form state with all the common patterns built-in.

---

## Syntax

```javascript
// Create form
const myForm = form(initialValues);

// Properties
myForm.values       // Form field values
myForm.errors       // Field error messages
myForm.touched      // Fields that have been modified
myForm.isSubmitting // Submit state

// Computed properties
myForm.isValid      // Auto-computed: no errors
myForm.isDirty      // Auto-computed: has touched fields

// Methods
myForm.$setValue(field, value)  // Set value + mark touched
myForm.$setError(field, error)  // Set or clear error
myForm.$reset(newValues)        // Reset entire form

// Example
const contactForm = form({
  name: '',
  email: '',
  message: ''
});

contactForm.$setValue('email', 'user@example.com');
contactForm.$setError('email', 'Invalid email');
console.log(contactForm.isValid); // false
```

**Parameters:**
- `initialValues` - Object with initial field values

**Returns:**
- Reactive form state with values, errors, touched, validation

---

## Why Does This Exist?

### The Problem Without form()

Managing form state manually requires tracking multiple pieces:

```javascript
// ❌ Vanilla JavaScript - manual form state management
let formValues = {
  username: '',
  email: '',
  password: '',
  confirmPassword: ''
};

let formErrors = {};
let touchedFields = {};
let isSubmitting = false;

// Manual validation
function validateField(field, value) {
  if (field === 'email') {
    if (!value.includes('@')) {
      formErrors.email = 'Invalid email';
    } else {
      delete formErrors.email;
    }
  }

  if (field === 'password') {
    if (value.length < 8) {
      formErrors.password = 'Password must be at least 8 characters';
    } else {
      delete formErrors.password;
    }
  }
}

// Manual isValid calculation
function isFormValid() {
  return Object.keys(formErrors).length === 0;
}

// Manual isDirty calculation
function isFormDirty() {
  return Object.keys(touchedFields).length > 0;
}

// Manual update display
function updateDisplay() {
  document.getElementById('email-error').textContent = formErrors.email || '';
  document.getElementById('password-error').textContent = formErrors.password || '';
  document.getElementById('submit-btn').disabled = !isFormValid() || isSubmitting;

  const saveIndicator = document.getElementById('save-indicator');
  saveIndicator.textContent = isFormDirty() ? 'Unsaved changes' : 'Saved';
}

// Manual input handlers
document.getElementById('email-input').addEventListener('input', (e) => {
  formValues.email = e.target.value;
  touchedFields.email = true;
  validateField('email', e.target.value);
  updateDisplay(); // Must call manually!
});

document.getElementById('password-input').addEventListener('input', (e) => {
  formValues.password = e.target.value;
  touchedFields.password = true;
  validateField('password', e.target.value);
  updateDisplay(); // Must call manually!
});

// Initial render
updateDisplay();
```

**Problems:**
❌ **Manual state tracking** - Separate variables for values, errors, touched
❌ **Manual validation** - Must write isValid logic yourself
❌ **Manual dirty tracking** - Must calculate isDirty yourself
❌ **Manual updates** - Must call `updateDisplay()` everywhere
❌ **Scattered logic** - Validation, state, UI updates all mixed

### The Solution with `form()`

```javascript
// ✅ DOM Helpers + Reactive State with form() - automatic everything
const signupForm = form({
  username: '',
  email: '',
  password: '',
  confirmPassword: ''
});

// Validation logic
function validateForm() {
  // Email validation
  if (signupForm.values.email && !signupForm.values.email.includes('@')) {
    signupForm.$setError('email', 'Invalid email');
  } else {
    signupForm.$setError('email', null);
  }

  // Password validation
  if (signupForm.values.password && signupForm.values.password.length < 8) {
    signupForm.$setError('password', 'Password must be at least 8 characters');
  } else {
    signupForm.$setError('password', null);
  }

  // Confirm password validation
  if (signupForm.values.confirmPassword !== signupForm.values.password) {
    signupForm.$setError('confirmPassword', 'Passwords do not match');
  } else {
    signupForm.$setError('confirmPassword', null);
  }
}

// Auto-update DOM using bulk updates
effect(() => {
  Elements.update({
    emailError: {
      textContent: signupForm.errors.email || '',
      style: { display: signupForm.errors.email ? 'block' : 'none' }
    },
    passwordError: {
      textContent: signupForm.errors.password || '',
      style: { display: signupForm.errors.password ? 'block' : 'none' }
    },
    confirmPasswordError: {
      textContent: signupForm.errors.confirmPassword || '',
      style: { display: signupForm.errors.confirmPassword ? 'block' : 'none' }
    },
    submitBtn: {
      disabled: !signupForm.isValid || signupForm.isSubmitting
    },
    saveIndicator: {
      textContent: signupForm.isDirty ? 'Unsaved changes' : 'Saved',
      style: { color: signupForm.isDirty ? 'orange' : 'green' }
    }
  });
});

// Input handlers using bulk event binding
Elements.update({
  emailInput: {
    addEventListener: ['input', (e) => {
      signupForm.$setValue('email', e.target.value);
      validateForm();
    }]
  },
  passwordInput: {
    addEventListener: ['input', (e) => {
      signupForm.$setValue('password', e.target.value);
      validateForm();
    }]
  },
  confirmPasswordInput: {
    addEventListener: ['input', (e) => {
      signupForm.$setValue('confirmPassword', e.target.value);
      validateForm();
    }]
  },
  submitBtn: {
    addEventListener: ['click', async () => {
      if (signupForm.isValid) {
        signupForm.isSubmitting = true;
        try {
          await submitForm(signupForm.values);
          signupForm.$reset();
        } catch (error) {
          signupForm.$setError('_form', error.message);
        } finally {
          signupForm.isSubmitting = false;
        }
      }
    }]
  }
});
```

**Benefits:**
✅ **Built-in state tracking** - values, errors, touched included
✅ **Computed validation** - isValid automatically calculated
✅ **Computed dirty state** - isDirty automatically calculated
✅ **Automatic updates** - Effects handle DOM sync
✅ **Clean API** - $setValue, $setError, $reset methods

---

## Mental Model: Smart Form Assistant

Think of `form()` like **a smart assistant managing your form**:

**Without form() (Manual Management):**
```
┌─────────────────────────────┐
│  You Manage Everything      │
│                             │
│  Track values manually      │
│  Track errors manually      │
│  Track touched manually     │
│  Calculate isValid          │
│  Calculate isDirty          │
│  Update UI manually         │
│                             │
│  Lots of manual work!       │
└─────────────────────────────┘
```

**With form() (Smart Assistant):**
```
┌─────────────────────────────┐
│  Assistant Manages Form     │
│                             │
│  ✓ Values tracked           │
│  ✓ Errors tracked           │
│  ✓ Touched tracked          │
│  ✓ isValid computed         │
│  ✓ isDirty computed         │
│  ✓ UI auto-updates          │
│                             │
│  Just use the methods!      │
└─────────────────────────────┘
```

The form object **knows everything about your form state** and keeps it organized.

---

## Built-in Properties

### State Properties
- `values` - Object containing all field values
- `errors` - Object containing field error messages
- `touched` - Object tracking which fields have been modified
- `isSubmitting` - Boolean indicating submit in progress

### Computed Properties (Auto-calculated)
- `isValid` - `true` if no errors exist
- `isDirty` - `true` if any fields have been touched

---

## Built-in Methods

### `$setValue(field, value)`
Set a field value and mark it as touched.

```javascript
form.$setValue('email', 'user@example.com');
// form.values.email = 'user@example.com'
// form.touched.email = true
```

### `$setError(field, error)`
Set or clear a field error.

```javascript
form.$setError('email', 'Invalid email'); // Set error
form.$setError('email', null);            // Clear error
```

### `$reset(newValues)`
Reset form to initial or new values.

```javascript
form.$reset(); // Reset to initial values
form.$reset({ email: '', password: '' }); // Reset to new values
```

---

## Basic Usage

### Example 1: Login Form

```javascript
const loginForm = form({
  email: '',
  password: '',
  rememberMe: false
});

// Validation
effect(() => {
  // Validate email
  if (loginForm.values.email && !loginForm.values.email.includes('@')) {
    loginForm.$setError('email', 'Invalid email address');
  } else {
    loginForm.$setError('email', null);
  }

  // Validate password
  if (loginForm.values.password && loginForm.values.password.length < 6) {
    loginForm.$setError('password', 'Password must be at least 6 characters');
  } else {
    loginForm.$setError('password', null);
  }
});

// UI updates using bulk updates
effect(() => {
  Elements.update({
    emailError: { textContent: loginForm.errors.email || '' },
    passwordError: { textContent: loginForm.errors.password || '' },
    loginBtn: {
      disabled: !loginForm.isValid || loginForm.isSubmitting,
      textContent: loginForm.isSubmitting ? 'Logging in...' : 'Login'
    }
  });
});

// Input handlers using bulk event binding
Elements.update({
  emailInput: {
    addEventListener: ['input', (e) => {
      loginForm.$setValue('email', e.target.value);
    }]
  },
  passwordInput: {
    addEventListener: ['input', (e) => {
      loginForm.$setValue('password', e.target.value);
    }]
  },
  rememberMeCheckbox: {
    addEventListener: ['change', (e) => {
      loginForm.$setValue('rememberMe', e.target.checked);
    }]
  },
  loginBtn: {
    addEventListener: ['click', async () => {
      loginForm.isSubmitting = true;
      try {
        await login(loginForm.values);
        loginForm.$reset();
      } catch (error) {
        loginForm.$setError('_form', error.message);
      } finally {
        loginForm.isSubmitting = false;
      }
    }]
  }
});
```

---

### Example 2: Contact Form with Validation

```javascript
const contactForm = form({
  name: '',
  email: '',
  subject: '',
  message: ''
});

// Comprehensive validation
effect(() => {
  // Name validation
  if (contactForm.touched.name && contactForm.values.name.length < 2) {
    contactForm.$setError('name', 'Name must be at least 2 characters');
  } else {
    contactForm.$setError('name', null);
  }

  // Email validation
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  if (contactForm.touched.email && !emailRegex.test(contactForm.values.email)) {
    contactForm.$setError('email', 'Please enter a valid email');
  } else {
    contactForm.$setError('email', null);
  }

  // Message validation
  if (contactForm.touched.message && contactForm.values.message.length < 10) {
    contactForm.$setError('message', 'Message must be at least 10 characters');
  } else {
    contactForm.$setError('message', null);
  }
});

// UI updates using bulk updates
effect(() => {
  Elements.update({
    nameError: {
      textContent: contactForm.errors.name || '',
      style: { display: contactForm.errors.name ? 'block' : 'none' }
    },
    emailError: {
      textContent: contactForm.errors.email || '',
      style: { display: contactForm.errors.email ? 'block' : 'none' }
    },
    messageError: {
      textContent: contactForm.errors.message || '',
      style: { display: contactForm.errors.message ? 'block' : 'none' }
    },
    charCount: {
      textContent: `${contactForm.values.message.length} characters`
    },
    submitBtn: { disabled: !contactForm.isValid }
  });
});
```

---

### Example 3: Registration Form with Password Matching

```javascript
const registerForm = form({
  username: '',
  email: '',
  password: '',
  confirmPassword: '',
  agreeToTerms: false
});

// Validation with cross-field checks
effect(() => {
  // Username validation
  if (registerForm.touched.username) {
    if (registerForm.values.username.length < 3) {
      registerForm.$setError('username', 'Username must be at least 3 characters');
    } else if (!/^[a-zA-Z0-9_]+$/.test(registerForm.values.username)) {
      registerForm.$setError('username', 'Username can only contain letters, numbers, and underscores');
    } else {
      registerForm.$setError('username', null);
    }
  }

  // Email validation
  if (registerForm.touched.email) {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    if (!emailRegex.test(registerForm.values.email)) {
      registerForm.$setError('email', 'Please enter a valid email');
    } else {
      registerForm.$setError('email', null);
    }
  }

  // Password strength validation
  if (registerForm.touched.password) {
    if (registerForm.values.password.length < 8) {
      registerForm.$setError('password', 'Password must be at least 8 characters');
    } else if (!/[A-Z]/.test(registerForm.values.password)) {
      registerForm.$setError('password', 'Password must contain at least one uppercase letter');
    } else if (!/[0-9]/.test(registerForm.values.password)) {
      registerForm.$setError('password', 'Password must contain at least one number');
    } else {
      registerForm.$setError('password', null);
    }
  }

  // Confirm password validation
  if (registerForm.touched.confirmPassword) {
    if (registerForm.values.confirmPassword !== registerForm.values.password) {
      registerForm.$setError('confirmPassword', 'Passwords do not match');
    } else {
      registerForm.$setError('confirmPassword', null);
    }
  }

  // Terms validation
  if (registerForm.touched.agreeToTerms && !registerForm.values.agreeToTerms) {
    registerForm.$setError('agreeToTerms', 'You must agree to the terms');
  } else {
    registerForm.$setError('agreeToTerms', null);
  }
});

// Password strength indicator
effect(() => {
  const password = registerForm.values.password;
  let strength = 0;

  if (password.length >= 8) strength++;
  if (/[A-Z]/.test(password)) strength++;
  if (/[a-z]/.test(password)) strength++;
  if (/[0-9]/.test(password)) strength++;
  if (/[^A-Za-z0-9]/.test(password)) strength++;

  const strengthText = ['Weak', 'Fair', 'Good', 'Strong', 'Very Strong'];
  const strengthColor = ['red', 'orange', 'yellow', 'lightgreen', 'green'];

  Elements.update({
    passwordStrength: {
      textContent: password ? strengthText[Math.min(strength - 1, 4)] : '',
      style: {
        color: password ? strengthColor[Math.min(strength - 1, 4)] : '',
        display: password ? 'block' : 'none'
      }
    }
  });
});
```

---

### Example 4: Profile Edit Form with Dirty State

```javascript
const profileForm = form({
  firstName: 'Alice',
  lastName: 'Johnson',
  bio: 'Software developer',
  website: 'https://example.com'
});

// Unsaved changes warning
effect(() => {
  Elements.update({
    unsavedWarning: {
      textContent: profileForm.isDirty ? '⚠️ You have unsaved changes' : '',
      style: {
        display: profileForm.isDirty ? 'block' : 'none',
        color: 'orange'
      }
    },
    saveBtn: {
      disabled: !profileForm.isDirty || !profileForm.isValid,
      textContent: profileForm.isSubmitting ? 'Saving...' : 'Save Changes'
    },
    discardBtn: {
      style: { display: profileForm.isDirty ? 'inline-block' : 'none' }
    }
  });
});

// Warn before leaving page
effect(() => {
  window.onbeforeunload = profileForm.isDirty
    ? () => 'You have unsaved changes. Are you sure you want to leave?'
    : null;
});

// Input handlers using bulk event binding
Elements.update({
  saveBtn: {
    addEventListener: ['click', async () => {
      profileForm.isSubmitting = true;
      try {
        await saveProfile(profileForm.values);
        profileForm.$reset(profileForm.values); // Reset with current values
      } finally {
        profileForm.isSubmitting = false;
      }
    }]
  },
  discardBtn: {
    addEventListener: ['click', () => {
      if (confirm('Discard unsaved changes?')) {
        profileForm.$reset();
      }
    }]
  }
});
```

---

### Example 5: Multi-Step Form

```javascript
const wizardForm = form({
  // Step 1
  firstName: '',
  lastName: '',
  email: '',

  // Step 2
  address: '',
  city: '',
  zipCode: '',

  // Step 3
  cardNumber: '',
  expiryDate: '',
  cvv: ''
});

const currentStep = ref(1);

// Step-specific validation
effect(() => {
  if (currentStep.value === 1) {
    // Validate step 1
    if (wizardForm.touched.firstName && !wizardForm.values.firstName) {
      wizardForm.$setError('firstName', 'First name is required');
    } else {
      wizardForm.$setError('firstName', null);
    }

    if (wizardForm.touched.email && !wizardForm.values.email.includes('@')) {
      wizardForm.$setError('email', 'Invalid email');
    } else {
      wizardForm.$setError('email', null);
    }
  }

  if (currentStep.value === 2) {
    // Validate step 2
    if (wizardForm.touched.zipCode && !/^\d{5}$/.test(wizardForm.values.zipCode)) {
      wizardForm.$setError('zipCode', 'Invalid zip code');
    } else {
      wizardForm.$setError('zipCode', null);
    }
  }
});

// Step navigation
effect(() => {
  Elements.update({
    step1: { style: { display: currentStep.value === 1 ? 'block' : 'none' } },
    step2: { style: { display: currentStep.value === 2 ? 'block' : 'none' } },
    step3: { style: { display: currentStep.value === 3 ? 'block' : 'none' } },

    prevBtn: {
      style: { display: currentStep.value > 1 ? 'inline-block' : 'none' }
    },
    nextBtn: {
      style: { display: currentStep.value < 3 ? 'inline-block' : 'none' },
      disabled: !wizardForm.isValid
    },
    submitBtn: {
      style: { display: currentStep.value === 3 ? 'inline-block' : 'none' },
      disabled: !wizardForm.isValid
    },

    progressBar: {
      style: { width: `${(currentStep.value / 3) * 100}%` }
    }
  });
});

// Navigation handlers using bulk event binding
Elements.update({
  nextBtn: {
    addEventListener: ['click', () => {
      currentStep.value++;
    }]
  },
  prevBtn: {
    addEventListener: ['click', () => {
      currentStep.value--;
    }]
  }
});
```

---

### Example 6: Search Form with Debounced Validation

```javascript
const searchForm = form({
  query: '',
  category: 'all',
  minPrice: 0,
  maxPrice: 1000
});

let searchTimeout;

// Debounced search
effect(() => {
  clearTimeout(searchTimeout);

  if (searchForm.values.query.length > 0 && searchForm.values.query.length < 3) {
    searchForm.$setError('query', 'Search must be at least 3 characters');
  } else {
    searchForm.$setError('query', null);

    if (searchForm.values.query.length >= 3) {
      searchTimeout = setTimeout(() => {
        performSearch(searchForm.values);
      }, 500);
    }
  }
});

// Price range validation
effect(() => {
  if (searchForm.values.minPrice > searchForm.values.maxPrice) {
    searchForm.$setError('minPrice', 'Min price cannot exceed max price');
  } else {
    searchForm.$setError('minPrice', null);
  }
});
```

---

### Example 7: Dynamic Field Form

```javascript
const surveyForm = form({
  name: '',
  email: '',
  responses: {}
});

const questions = ref([
  { id: 1, text: 'How satisfied are you?', type: 'rating' },
  { id: 2, text: 'Any comments?', type: 'text' }
]);

// Initialize response fields
effect(() => {
  questions.value.forEach(q => {
    if (!(q.id in surveyForm.values.responses)) {
      surveyForm.values.responses[q.id] = '';
    }
  });
});

// Validation
effect(() => {
  let allAnswered = true;
  questions.value.forEach(q => {
    if (!surveyForm.values.responses[q.id]) {
      allAnswered = false;
    }
  });

  if (surveyForm.touched.responses && !allAnswered) {
    surveyForm.$setError('responses', 'Please answer all questions');
  } else {
    surveyForm.$setError('responses', null);
  }
});
```

---

### Example 8: File Upload Form

```javascript
const uploadForm = form({
  title: '',
  description: '',
  file: null,
  category: 'general'
});

// File validation
effect(() => {
  if (uploadForm.touched.file) {
    if (!uploadForm.values.file) {
      uploadForm.$setError('file', 'Please select a file');
    } else if (uploadForm.values.file.size > 5 * 1024 * 1024) {
      uploadForm.$setError('file', 'File must be smaller than 5MB');
    } else if (!['image/jpeg', 'image/png'].includes(uploadForm.values.file.type)) {
      uploadForm.$setError('file', 'Only JPEG and PNG files are allowed');
    } else {
      uploadForm.$setError('file', null);
    }
  }
});

// UI updates using bulk updates
effect(() => {
  Elements.update({
    fileError: {
      textContent: uploadForm.errors.file || '',
      style: { display: uploadForm.errors.file ? 'block' : 'none' }
    },
    fileInfo: {
      textContent: uploadForm.values.file
        ? `${uploadForm.values.file.name} (${(uploadForm.values.file.size / 1024).toFixed(1)}KB)`
        : 'No file selected'
    },
    uploadBtn: {
      disabled: !uploadForm.isValid || uploadForm.isSubmitting
    }
  });
});

// File input using bulk event binding
Elements.update({
  fileInput: {
    addEventListener: ['change', (e) => {
      uploadForm.$setValue('file', e.target.files[0]);
    }]
  }
});
```

---

### Example 9: Settings Form with Sections

```javascript
const settingsForm = form({
  // Account settings
  displayName: '',
  email: '',

  // Privacy settings
  profileVisibility: 'public',
  showEmail: false,

  // Notification settings
  emailNotifications: true,
  pushNotifications: false,

  // Preferences
  language: 'en',
  theme: 'light'
});

// Track which sections have changes
const sectionsDirty = computed(settingsForm, {
  accountDirty: function() {
    return this.touched.displayName || this.touched.email;
  },
  privacyDirty: function() {
    return this.touched.profileVisibility || this.touched.showEmail;
  },
  notificationsDirty: function() {
    return this.touched.emailNotifications || this.touched.pushNotifications;
  },
  preferencesDirty: function() {
    return this.touched.language || this.touched.theme;
  }
});

// Show section indicators
effect(() => {
  Elements.update({
    accountIndicator: {
      textContent: sectionsDirty.accountDirty ? '•' : '',
      style: { color: 'orange' }
    },
    privacyIndicator: {
      textContent: sectionsDirty.privacyDirty ? '•' : '',
      style: { color: 'orange' }
    },
    notificationsIndicator: {
      textContent: sectionsDirty.notificationsDirty ? '•' : '',
      style: { color: 'orange' }
    },
    preferencesIndicator: {
      textContent: sectionsDirty.preferencesDirty ? '•' : '',
      style: { color: 'orange' }
    }
  });
});
```

---

### Example 10: Checkout Form with Real-Time Validation

```javascript
const checkoutForm = form({
  email: '',
  cardNumber: '',
  expiryMonth: '',
  expiryYear: '',
  cvv: '',
  billingZip: ''
});

// Real-time card validation
effect(() => {
  if (checkoutForm.touched.cardNumber) {
    const cleaned = checkoutForm.values.cardNumber.replace(/\s/g, '');

    if (!/^\d+$/.test(cleaned)) {
      checkoutForm.$setError('cardNumber', 'Card number must contain only digits');
    } else if (cleaned.length !== 16) {
      checkoutForm.$setError('cardNumber', 'Card number must be 16 digits');
    } else if (!luhnCheck(cleaned)) {
      checkoutForm.$setError('cardNumber', 'Invalid card number');
    } else {
      checkoutForm.$setError('cardNumber', null);
    }
  }

  // Expiry validation
  if (checkoutForm.touched.expiryMonth || checkoutForm.touched.expiryYear) {
    const month = parseInt(checkoutForm.values.expiryMonth);
    const year = parseInt(checkoutForm.values.expiryYear);
    const now = new Date();
    const currentYear = now.getFullYear();
    const currentMonth = now.getMonth() + 1;

    if (year < currentYear || (year === currentYear && month < currentMonth)) {
      checkoutForm.$setError('expiry', 'Card has expired');
    } else {
      checkoutForm.$setError('expiry', null);
    }
  }

  // CVV validation
  if (checkoutForm.touched.cvv) {
    if (!/^\d{3,4}$/.test(checkoutForm.values.cvv)) {
      checkoutForm.$setError('cvv', 'CVV must be 3 or 4 digits');
    } else {
      checkoutForm.$setError('cvv', null);
    }
  }
});

// Luhn algorithm for card validation
function luhnCheck(cardNumber) {
  let sum = 0;
  let isEven = false;

  for (let i = cardNumber.length - 1; i >= 0; i--) {
    let digit = parseInt(cardNumber[i]);

    if (isEven) {
      digit *= 2;
      if (digit > 9) digit -= 9;
    }

    sum += digit;
    isEven = !isEven;
  }

  return sum % 10 === 0;
}
```

---

## Common Patterns

### Pattern 1: Field-Level Blur Validation

```javascript
Elements.update({
  emailInput: {
    addEventListener: [
      ['input', (e) => {
        form.$setValue('email', e.target.value);
      }],
      ['blur', () => {
        // Only validate on blur
        if (form.values.email && !form.values.email.includes('@')) {
          form.$setError('email', 'Invalid email');
        }
      }]
    ]
  }
});
```

### Pattern 2: Async Validation

```javascript
let validationTimeout;

effect(() => {
  clearTimeout(validationTimeout);

  if (form.values.username) {
    validationTimeout = setTimeout(async () => {
      const exists = await checkUsernameExists(form.values.username);
      if (exists) {
        form.$setError('username', 'Username already taken');
      } else {
        form.$setError('username', null);
      }
    }, 500);
  }
});
```

### Pattern 3: Form-Level Error

```javascript
// Submit handler
Elements.update({
  submitBtn: {
    addEventListener: ['click', async () => {
      form.isSubmitting = true;
      try {
        await submitData(form.values);
      } catch (error) {
        form.$setError('_form', error.message);
      } finally {
        form.isSubmitting = false;
      }
    }]
  }
});

// Display form-level error
effect(() => {
  Elements.update({
    formError: {
      textContent: form.errors._form || '',
      style: { display: form.errors._form ? 'block' : 'none' }
    }
  });
});
```

---

## Key Takeaways

✅ **Built-in state** - values, errors, touched, isSubmitting included
✅ **Computed validation** - isValid and isDirty auto-calculated
✅ **Clean methods** - $setValue, $setError, $reset for common operations
✅ **Reactive** - Works seamlessly with effects and bindings
✅ **Form patterns** - Handles all common form scenarios

---

## What's Next?

- **`state()`** - For general reactive state
- **`asyncState()`** - For async operations with loading/error states
- **`computed()`** - Add custom computed properties
- **`validation libraries`** - Integrate third-party validation

---

## Summary

`form()` creates **reactive form state with built-in validation tracking**. It manages values, errors, touched fields, and computed validation states automatically.

**The magic formula:**
```
form(initialValues) =
  state({ values, errors, touched, isSubmitting }) +
  computed({ isValid, isDirty }) +
  methods ($setValue, $setError, $reset)
─────────────────────────────────────────────────
Complete form state management
```

Think of it as **a smart form assistant** — it tracks every field, remembers what's touched, validates inputs, and tells you when the form is ready to submit. No manual state juggling needed.
