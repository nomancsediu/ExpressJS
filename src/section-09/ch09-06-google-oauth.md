# Google OAuth with Passport.js

Implementing OAuth from scratch is tedious. There are redirects, code exchanges, token management, and profile fetching. Passport.js makes this much simpler by handling the flow for me.

## What Passport.js Is

Passport is authentication middleware for Express. It has a strategy-based architecture. Each strategy handles a specific authentication method. For Google OAuth, I use the `passport-google-oauth20` strategy.

```bash
npm install passport passport-google-oauth20
```

## Configuration

I configure the Google strategy with my client ID, client secret, and callback URL:

```js
const passport = require('passport');
const GoogleStrategy = require('passport-google-oauth20').Strategy;
const User = require('../models/User');

passport.use(new GoogleStrategy({
  clientID: process.env.GOOGLE_CLIENT_ID,
  clientSecret: process.env.GOOGLE_CLIENT_SECRET,
  callbackURL: '/auth/google/callback',
}, async (accessToken, refreshToken, profile, done) => {
  try {
    // Check if user already exists
    let user = await User.findOne({ googleId: profile.id });

    if (user) {
      return done(null, user);
    }

    // Check if user with same email exists
    user = await User.findOne({ email: profile.emails[0].value });

    if (user) {
      // Link Google account to existing user
      user.googleId = profile.id;
      await user.save();
      return done(null, user);
    }

    // Create new user
    user = await User.create({
      googleId: profile.id,
      name: profile.displayName,
      email: profile.emails[0].value,
      avatar: profile.photos[0]?.value,
    });

    done(null, user);
  } catch (err) {
    done(err, null);
  }
}));
```

The `done` callback tells Passport whether authentication succeeded or failed.

## Session Serialization

If I use sessions with Passport, I need to serialize and deserialize the user:

```js
passport.serializeUser((user, done) => {
  done(null, user.id);
});

passport.deserializeUser(async (id, done) => {
  try {
    const user = await User.findById(id);
    done(null, user);
  } catch (err) {
    done(err, null);
  }
});
```

Serialization stores the user ID in the session. Deserialization fetches the full user on each request.

## Routes

The routes are where Passport really shines. Two lines handle the entire OAuth flow:

```js
const express = require('express');
const passport = require('passport');
const router = express.Router();

// Start Google OAuth flow
router.get('/google',
  passport.authenticate('google', { scope: ['profile', 'email'] })
);

// Google OAuth callback
router.get('/google/callback',
  passport.authenticate('google', { failureRedirect: '/login' }),
  (req, res) => {
    // Successful authentication
    res.redirect('/dashboard');
  }
);

// Logout
router.get('/logout', (req, res) => {
  req.logout((err) => {
    if (err) return next(err);
    res.redirect('/');
  });
});

module.exports = router;
```

## App Setup

I initialize Passport in my Express app:

```js
const passport = require('passport');
const session = require('express-session');

require('./config/passport'); // Load strategies

app.use(session({
  secret: process.env.SESSION_SECRET,
  resave: false,
  saveUninitialized: false,
}));

app.use(passport.initialize());
app.use(passport.session());
```

## Getting the Credentials

I register my app in the Google Cloud Console:

1. Go to console.cloud.google.com
2. Create a new project
3. Enable the Google+ API
4. Create OAuth 2.0 credentials
5. Set the authorized redirect URI to `http://localhost:3000/auth/google/callback`
6. Copy the client ID and client secret into my `.env` file

```env
GOOGLE_CLIENT_ID=your-client-id.apps.googleusercontent.com
GOOGLE_CLIENT_SECRET=your-client-secret
```

## Protecting Routes

```js
const isAuthenticated = (req, res, next) => {
  if (req.isAuthenticated()) {
    return next();
  }
  res.redirect('/login');
};

router.get('/dashboard', isAuthenticated, (req, res) => {
  res.json({ user: req.user });
});
```

Passport's `req.isAuthenticated()` checks if the user is logged in. The `req.user` property contains the deserialized user object.

With Passport, adding Google login takes minutes instead of hours. The same pattern works for GitHub, Facebook, Twitter, and dozens of other providers. Just swap the strategy.
