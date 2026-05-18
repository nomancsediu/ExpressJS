# User Module

The user module is the foundation. Everything else depends on it. Products, carts, orders, they all connect back to a user. So let us build this one carefully.

## User Model

I keep the model simple but complete. Name, email, password, role. The role lets me separate regular users from admins.

```js
// src/modules/user/user.model.js
const mongoose = require('mongoose');
const bcrypt = require('bcryptjs');

const userSchema = new mongoose.Schema({
  name: { type: String, required: true, trim: true },
  email: { type: String, required: true, unique: true, lowercase: true },
  password: { type: String, required: true, minlength: 6 },
  role: { type: String, enum: ['user', 'admin'], default: 'user' },
}, { timestamps: true });

userSchema.pre('save', async function (next) {
  if (!this.isModified('password')) return next();
  this.password = await bcrypt.hash(this.password, 12);
  next();
});

userSchema.methods.comparePassword = async function (candidate) {
  return bcrypt.compare(candidate, this.password);
};

module.exports = mongoose.model('User', userSchema);
```

The pre-save hook hashes the password automatically. I never store plain text passwords.

## Auth Routes

I need four routes: register, login, logout, and refresh token.

```js
// src/modules/user/user.routes.js
const router = require('express').Router();
const { register, login, logout, refreshToken, getProfile, updateProfile } = require('./user.controller');
const { protect } = require('./auth.middleware');

router.post('/register', register);
router.post('/login', login);
router.post('/logout', protect, logout);
router.post('/refresh-token', refreshToken);
router.get('/profile', protect, getProfile);
router.patch('/profile', protect, updateProfile);

module.exports = router;
```

## Auth Controller

The register and login logic uses JWT. I generate both an access token (short lived) and a refresh token (long lived).

```js
// src/modules/user/user.controller.js
const jwt = require('jsonwebtoken');
const User = require('./user.model');
const ApiError = require('../../utils/ApiError');
const asyncHandler = require('../../middleware/asyncHandler');

const signToken = (id) =>
  jwt.sign({ id }, process.env.JWT_SECRET, { expiresIn: '15m' });

const signRefresh = (id) =>
  jwt.sign({ id }, process.env.JWT_REFRESH_SECRET, { expiresIn: '7d' });

exports.register = asyncHandler(async (req, res) => {
  const { name, email, password } = req.body;
  const user = await User.create({ name, email, password });

  const accessToken = signToken(user._id);
  const refreshToken = signRefresh(user._id);

  res.cookie('refreshToken', refreshToken, {
    httpOnly: true,
    secure: process.env.NODE_ENV === 'production',
    maxAge: 7 * 24 * 60 * 60 * 1000,
  });

  res.status(201).json({
    success: true,
    data: { user: { id: user._id, name: user.name, email: user.email }, accessToken },
  });
});

exports.login = asyncHandler(async (req, res) => {
  const { email, password } = req.body;
  const user = await User.findOne({ email });

  if (!user || !(await user.comparePassword(password))) {
    throw new ApiError(401, 'Invalid email or password');
  }

  const accessToken = signToken(user._id);
  const refreshToken = signRefresh(user._id);

  res.cookie('refreshToken', refreshToken, {
    httpOnly: true,
    secure: process.env.NODE_ENV === 'production',
    maxAge: 7 * 24 * 60 * 60 * 1000,
  });

  res.json({
    success: true,
    data: { user: { id: user._id, name: user.name, role: user.role }, accessToken },
  });
});
```

## Auth Middleware

This middleware protects routes by verifying the access token.

```js
// src/modules/user/auth.middleware.js
const jwt = require('jsonwebtoken');
const User = require('./user.model');
const ApiError = require('../../utils/ApiError');
const asyncHandler = require('../../middleware/asyncHandler');

exports.protect = asyncHandler(async (req, res, next) => {
  let token;
  if (req.headers.authorization?.startsWith('Bearer')) {
    token = req.headers.authorization.split(' ')[1];
  }

  if (!token) throw new ApiError(401, 'Not authenticated');

  const decoded = jwt.verify(token, process.env.JWT_SECRET);
  req.user = await User.findById(decoded.id);

  if (!req.user) throw new ApiError(401, 'User no longer exists');
  next();
});

exports.restrictTo = (...roles) => (req, res, next) => {
  if (!roles.includes(req.user.role)) {
    throw new ApiError(403, 'Not authorized');
  }
  next();
};
```

The `protect` middleware checks the token and attaches the user to the request. The `restrictTo` middleware lets me limit routes to specific roles. Now every protected route just needs to include `protect` and optionally `restrictTo('admin')`.
