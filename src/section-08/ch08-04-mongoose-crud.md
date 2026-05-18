# Mongoose CRUD

CRUD stands for Create, Read, Update, and Delete. These are the four basic operations I perform on any database. Let me walk through each one with Mongoose.

## Create

There are two ways to create a document:

```js
// Method 1: Using create() (most common)
const user = await User.create({
  name: 'Alice',
  email: 'alice@example.com',
  age: 28,
});

// Method 2: Using new + save()
const user = new User({
  name: 'Bob',
  email: 'bob@example.com',
});
await user.save();
```

I use `create()` most of the time because it is shorter. The `new + save()` approach is useful when I need to modify the document before saving.

## Read

Mongoose gives me several ways to query:

```js
// Find all documents
const users = await User.find();

// Find with filter
const admins = await User.find({ role: 'admin' });

// Find one by condition
const user = await User.findOne({ email: 'alice@example.com' });

// Find by ID
const user = await User.findById('65f1a2b3c4d5e6f7a8b9c0d1');

// Select specific fields
const users = await User.find().select('name email');

// Sort results
const users = await User.find().sort({ createdAt: -1 });

// Pagination
const page = 1;
const limit = 10;
const users = await User.find()
  .skip((page - 1) * limit)
  .limit(limit);
```

I can chain methods to build complex queries. Each method returns a query object that I can keep building on.

## Update

```js
// Find by ID and update
const updated = await User.findByIdAndUpdate(
  id,
  { age: 29 },
  { new: true, runValidators: true }
);

// Update one document matching a filter
await User.updateOne({ email: 'alice@example.com' }, { age: 29 });

// Update multiple documents
await User.updateMany({ role: 'user' }, { isActive: true });
```

The `{ new: true }` option returns the updated document instead of the original. The `{ runValidators: true }` option runs schema validators on the update, which is off by default. I always include both.

## Delete

```js
// Find by ID and delete
await User.findByIdAndDelete(id);

// Delete one matching a filter
await User.deleteOne({ email: 'old@example.com' });

// Delete multiple
await User.deleteMany({ isActive: false });
```

## Lean Queries

By default, Mongoose returns full model instances with all methods attached. This is useful but uses more memory. When I only need the data and do not plan to modify it, I use `.lean()`:

```js
// Without lean: returns a Mongoose document (heavier)
const users = await User.find();

// With lean: returns plain JavaScript objects (faster, less memory)
const users = await User.find().lean();
```

Lean queries can be 5 to 10 times faster for large result sets. I use them for read-only operations like API responses.

## CRUD in Express Routes

Here is how I wire CRUD into Express routes:

```js
const express = require('express');
const router = express.Router();
const AppError = require('../utils/AppError');

// CREATE
router.post('/', async (req, res, next) => {
  try {
    const user = await User.create(req.body);
    res.status(201).json(user);
  } catch (err) {
    next(err);
  }
});

// READ all
router.get('/', async (req, res) => {
  const users = await User.find().lean();
  res.json(users);
});

// READ one
router.get('/:id', async (req, res, next) => {
  const user = await User.findById(req.params.id).lean();
  if (!user) return next(new AppError('User not found', 404));
  res.json(user);
});

// UPDATE
router.patch('/:id', async (req, res, next) => {
  const user = await User.findByIdAndUpdate(
    req.params.id, req.body, { new: true, runValidators: true }
  );
  if (!user) return next(new AppError('User not found', 404));
  res.json(user);
});

// DELETE
router.delete('/:id', async (req, res, next) => {
  const user = await User.findByIdAndDelete(req.params.id);
  if (!user) return next(new AppError('User not found', 404));
  res.status(204).send();
});

module.exports = router;
```

This pattern repeats in every project. Once you know CRUD with Mongoose, you can build any data API.
