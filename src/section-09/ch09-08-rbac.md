# Role-Based Access Control

Not all users should have the same permissions. A regular user should not be able to delete other users or access admin dashboards. Role-based access control (RBAC) is how I handle this.

## What RBAC Is

In RBAC, I assign roles to users, and each role has specific permissions. When a user makes a request, I check their role and whether that role allows the action.

```js
// User roles in my app
const ROLES = {
  USER: 'user',
  MODERATOR: 'moderator',
  ADMIN: 'admin',
};
```

A user can create their own posts. A moderator can edit anyone's posts. An admin can delete anything and manage users.

## Defining Roles and Permissions

I keep it simple with a permissions map:

```js
const permissions = {
  user: ['read:posts', 'write:own_posts', 'delete:own_posts'],
  moderator: ['read:posts', 'write:posts', 'delete:posts', 'moderate:comments'],
  admin: ['read:posts', 'write:posts', 'delete:posts', 'manage:users', 'manage:roles'],
};
```

Each permission has a format: `action:resource`. This makes it clear what the permission allows.

## Permission Middleware

I create a reusable middleware that checks permissions:

```js
const AppError = require('../utils/AppError');

const authorize = (...requiredPermissions) => {
  return (req, res, next) => {
    if (!req.user) {
      return next(new AppError('Not authenticated', 401));
    }

    const userPermissions = permissions[req.user.role] || [];

    const hasPermission = requiredPermissions.every(
      (perm) => userPermissions.includes(perm)
    );

    if (!hasPermission) {
      return next(new AppError('Not authorized', 403));
    }

    next();
  };
};

module.exports = authorize;
```

## Role-Based Middleware Shortcut

For simple cases where I only need to check the role:

```js
const authorizeRole = (...roles) => {
  return (req, res, next) => {
    if (!roles.includes(req.user.role)) {
      return next(new AppError('Not authorized', 403));
    }
    next();
  };
};
```

## Using the Middleware

```js
const protect = require('../middleware/protect');
const authorize = require('../middleware/authorize');
const authorizeRole = require('../middleware/authorizeRole');

// Anyone can read posts
router.get('/posts', getPosts);

// Only authenticated users can create posts
router.post('/posts', protect, createPost);

// Only moderators and admins can delete any post
router.delete('/posts/:id', protect, authorize('delete:posts'), deletePost);

// Only admins can manage users
router.get('/admin/users', protect, authorizeRole('admin'), getUsers);
router.patch('/admin/users/:id/role', protect, authorize('manage:roles'), updateRole);
```

## Ownership Checks

Some actions depend on who owns the resource. A user can edit their own posts but not someone else's:

```js
const authorizeOwnerOrAdmin = (model) => {
  return async (req, res, next) => {
    const resource = await model.findById(req.params.id);

    if (!resource) {
      return next(new AppError('Resource not found', 404));
    }

    const isOwner = resource.authorId.toString() === req.user.id;
    const isAdmin = req.user.role === 'admin';

    if (!isOwner && !isAdmin) {
      return next(new AppError('Not authorized', 403));
    }

    req.resource = resource;
    next();
  };
};

// Usage
router.patch('/posts/:id', protect, authorizeOwnerOrAdmin(Post), updatePost);
```

## Storing Roles in the Database

I add a role field to my user schema with a default:

```js
const userSchema = new mongoose.Schema({
  name: String,
  email: { type: String, unique: true },
  passwordHash: String,
  role: {
    type: String,
    enum: ['user', 'moderator', 'admin'],
    default: 'user',
  },
});
```

RBAC starts simple and grows with my app. I begin with three roles and add more as needed. The permission-based approach scales better than checking roles directly because I can add new roles without changing my middleware code.
