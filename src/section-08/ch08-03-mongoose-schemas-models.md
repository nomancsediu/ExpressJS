# Mongoose Schemas and Models

Schemas and models are where Mongoose really shines. A schema defines the shape of my documents, and a model gives me the methods to create, read, update, and delete them.

## Schema Types

A schema defines what fields a document can have, what types they are, and what rules they follow:

```js
const mongoose = require('mongoose');

const userSchema = new mongoose.Schema({
  name: {
    type: String,
    required: [true, 'Name is required'],
    trim: true,
    maxlength: 50,
  },
  email: {
    type: String,
    required: [true, 'Email is required'],
    unique: true,
    lowercase: true,
    match: [/^\S+@\S+\.\S+$/, 'Please provide a valid email'],
  },
  age: {
    type: Number,
    min: [0, 'Age cannot be negative'],
    max: 150,
  },
  role: {
    type: String,
    enum: ['user', 'admin', 'moderator'],
    default: 'user',
  },
  isActive: {
    type: Boolean,
    default: true,
  },
  createdAt: {
    type: Date,
    default: Date.now,
  },
});
```

Each field has a type and optional validators. I can require fields, set defaults, enforce uniqueness, trim whitespace, and match patterns. This catches bad data before it ever reaches the database.

## Validators

Mongoose has built-in validators for common types:

- **String**: `required`, `enum`, `match`, `minlength`, `maxlength`, `trim`, `lowercase`, `uppercase`
- **Number**: `required`, `min`, `max`
- **Date**: `required`, `min`, `max`

For custom validation, I write validator functions:

```js
const postSchema = new mongoose.Schema({
  title: {
    type: String,
    required: true,
    validate: {
      validator: function(value) {
        return value.length >= 3;
      },
      message: 'Title must be at least 3 characters',
    },
  },
});
```

## Virtuals

Virtuals are computed properties that are not stored in the database. They are derived from other fields:

```js
userSchema.virtual('displayName').get(function() {
  return `${this.name} (${this.email})`;
});

// Use it like a regular property
const user = await User.findById(id);
console.log(user.displayName); // "Alice (alice@example.com)"
```

Virtuals do not show up in `toJSON()` by default. I enable them:

```js
userSchema.set('toJSON', { virtuals: true });
userSchema.set('toObject', { virtuals: true });
```

## Methods

I add instance methods and static methods to my schemas:

```js
// Instance method: available on a document
userSchema.methods.toJSON = function() {
  const obj = this.toObject();
  delete obj.password; // Never send password in JSON
  return obj;
};

// Static method: available on the model
userSchema.statics.findByEmail = function(email) {
  return this.findOne({ email });
};
```

## Creating the Model

Once the schema is defined, I compile it into a model:

```js
const User = mongoose.model('User', userSchema);

module.exports = User;
```

The model is what I use to interact with the database. `User.find()`, `User.create()`, `User.findById()` are all model methods. The schema defines the rules, and the model enforces them.
