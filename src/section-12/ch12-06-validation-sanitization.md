# Validation and Sanitization

Never trust user input. Ever. Every piece of data from a request could be malformed, malicious, or just wrong. Validation checks that input matches what you expect. Sanitization cleans input to make it safe.

## express-validator

The most popular validation library for Express:

```bash
npm install express-validator
```

## Basic Validation

```js
const { body, validationResult } = require('express-validator');

app.post('/register',
  body('email').isEmail().withMessage('Enter a valid email'),
  body('password').isLength({ min: 8 }).withMessage('Password must be at least 8 characters'),
  body('name').notEmpty().withMessage('Name is required'),
  (req, res) => {
    const errors = validationResult(req);

    if (!errors.isEmpty()) {
      return res.status(400).json({ errors: errors.array() });
    }

    // Input is valid, proceed
    res.send('Registration successful');
  }
);
```

`validationResult()` collects all validation errors. You check once and return them all together. This gives the user a complete picture of what needs fixing.

## Validation Chain

You can chain multiple validators:

```js
body('username')
  .trim()                           // Remove whitespace
  .notEmpty()                       // Not empty after trimming
  .isLength({ min: 3, max: 20 })   // Length between 3 and 20
  .isAlphanumeric()                 // Only letters and numbers
  .withMessage('Username must be 3-20 alphanumeric characters');
```

Each method in the chain runs in order. If one fails, the rest still run, and all errors are collected.

## Sanitization

Sanitization transforms input to make it safe:

```js
body('email').normalizeEmail(),     // Lowercase, remove dots in Gmail, etc.
body('name').trim().escape(),       // Trim whitespace, escape HTML entities
body('age').toInt(),                // Convert to integer
body('website').trim().toLowerCase()
```

Sanitizers modify the value before validation runs. The `trim()` call removes whitespace so `'  hello  '` becomes `'hello'`. The `escape()` call turns `<script>` into `&lt;script&gt;`.

## Validating Route Parameters

Use `param()` for URL parameters and `query()` for query strings:

```js
const { param, query } = require('express-validator');

app.get('/users/:id',
  param('id').isInt({ min: 1 }).withMessage('Invalid user ID'),
  query('page').optional().isInt({ min: 1 }).default(1),
  (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.status(400).json({ errors: errors.array() });
    }
    // ...
  }
);
```

The `optional()` method skips validation if the field is missing. The `default()` method sets a fallback value.

## Custom Validators

For validation logic that built-in methods cannot handle:

```js
body('password').custom((value, { req }) => {
  if (value !== req.body.confirmPassword) {
    throw new Error('Passwords do not match');
  }
  return true;
});

body('username').custom(async (value) => {
  const user = await db.users.findOne({ username: value });
  if (user) {
    throw new Error('Username already taken');
  }
  return true;
});
```

Custom validators can be async. If the function throws an error or returns a rejected promise, validation fails.

## Reusable Validation Schemas

For routes that share validation logic:

```js
const userValidation = [
  body('email').isEmail().normalizeEmail(),
  body('password').isLength({ min: 8 }),
  body('name').trim().notEmpty()
];

app.post('/register', userValidation, (req, res) => { /* ... */ });
app.post('/update', userValidation, (req, res) => { /* ... */ });
```

I used to validate input manually with if-statements everywhere. It was messy and inconsistent. `express-validator` brought structure and saved me from writing the same checks over and over.
