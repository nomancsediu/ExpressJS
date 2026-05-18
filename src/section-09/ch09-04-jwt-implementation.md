# JWT Implementation

Now let me build a real JWT authentication system in Express. This is the setup I use in most of my projects.

## The jsonwebtoken Package

```bash
npm install jsonwebtoken
```

This is the standard JWT library for Node.js. It handles signing, verifying, and decoding tokens.

## Signing Tokens

I create a utility function for generating tokens:

```js
const jwt = require('jsonwebtoken');

const signToken = (id) => {
  return jwt.sign({ id }, process.env.JWT_SECRET, {
    expiresIn: process.env.JWT_EXPIRES_IN || '15m',
  });
};

const signRefreshToken = (id) => {
  return jwt.sign({ id }, process.env.JWT_REFRESH_SECRET, {
    expiresIn: process.env.JWT_REFRESH_EXPIRES_IN || '7d',
  });
};

module.exports = { signToken, signRefreshToken };
```

## Login Route

```js
const bcrypt = require('bcrypt');
const { signToken, signRefreshToken } = require('../utils/jwt');

router.post('/login', async (req, res, next) => {
  try {
    const { email, password } = req.body;

    if (!email || !password) {
      return res.status(400).json({ message: 'Email and password required' });
    }

    const user = await User.findOne({ email }).select('+passwordHash');
    if (!user) {
      return res.status(401).json({ message: 'Invalid credentials' });
    }

    const isMatch = await bcrypt.compare(password, user.passwordHash);
    if (!isMatch) {
      return res.status(401).json({ message: 'Invalid credentials' });
    }

    const accessToken = signToken(user.id);
    const refreshToken = signRefreshToken(user.id);

    // Store refresh token in database
    user.refreshToken = refreshToken;
    await user.save();

    // Send refresh token as httpOnly cookie
    res.cookie('refreshToken', refreshToken, {
      httpOnly: true,
      secure: process.env.NODE_ENV === 'production',
      sameSite: 'strict',
      maxAge: 7 * 24 * 60 * 60 * 1000, // 7 days
    });

    res.json({
      accessToken,
      user: { id: user.id, name: user.name, email: user.email },
    });
  } catch (err) {
    next(err);
  }
});
```

## Verify Middleware

I protect routes with a middleware that checks the access token:

```js
const jwt = require('jsonwebtoken');
const AppError = require('../utils/AppError');

const protect = async (req, res, next) => {
  try {
    let token;
    if (req.headers.authorization?.startsWith('Bearer')) {
      token = req.headers.authorization.split(' ')[1];
    }

    if (!token) {
      return next(new AppError('Not authenticated', 401));
    }

    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    const user = await User.findById(decoded.id);

    if (!user) {
      return next(new AppError('User no longer exists', 401));
    }

    req.user = user;
    next();
  } catch (err) {
    if (err.name === 'JsonWebTokenError') {
      return next(new AppError('Invalid token', 401));
    }
    if (err.name === 'TokenExpiredError') {
      return next(new AppError('Token expired', 401));
    }
    next(err);
  }
};

module.exports = protect;
```

## Refresh Flow

When the access token expires, the client uses the refresh token to get a new one:

```js
router.post('/refresh', async (req, res, next) => {
  try {
    const refreshToken = req.cookies.refreshToken;
    if (!refreshToken) {
      return res.status(401).json({ message: 'No refresh token' });
    }

    const decoded = jwt.verify(refreshToken, process.env.JWT_REFRESH_SECRET);
    const user = await User.findById(decoded.id);

    if (!user || user.refreshToken !== refreshToken) {
      return res.status(401).json({ message: 'Invalid refresh token' });
    }

    const accessToken = signToken(user.id);
    res.json({ accessToken });
  } catch (err) {
    return res.status(401).json({ message: 'Refresh token expired' });
  }
});
```

## Logout

```js
router.post('/logout', protect, async (req, res) => {
  req.user.refreshToken = null;
  await req.user.save();

  res.clearCookie('refreshToken');
  res.json({ message: 'Logged out' });
});
```

## Using the Middleware

```js
const protect = require('../middleware/protect');

// Public route
router.get('/posts', getPosts);

// Protected route
router.post('/posts', protect, createPost);

// Protected route with user context
router.get('/profile', protect, getProfile);
```

This setup gives me secure, stateless authentication with token refresh. The access token is short-lived for security, and the refresh token lets users stay logged in without re-entering their password.
