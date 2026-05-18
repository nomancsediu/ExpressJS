# Authentication vs Authorization

These two terms get mixed up all the time. I used them interchangeably for months before I realized they are completely different things. Let me clear this up.

## Authentication: Who Are You?

Authentication is about verifying identity. It answers the question "Are you who you claim to be?"

When a user enters their email and password, that is authentication. When they click "Sign in with Google", that is authentication. When they scan their fingerprint, that is authentication.

```js
// Authentication: verifying credentials
const user = await User.findOne({ email: req.body.email });
const isValid = await bcrypt.compare(req.body.password, user.passwordHash);

if (!isValid) {
  return res.status(401).json({ message: 'Invalid credentials' });
}
// At this point, the user is authenticated. We know who they are.
```

Common authentication methods:
- Email and password
- Social login (Google, GitHub, Facebook)
- Multi-factor authentication (SMS code, authenticator app)
- API keys
- Biometrics

## Authorization: What Can You Do?

Authorization is about permissions. It answers the question "Are you allowed to do this?"

After authentication, I know the user is Alice. But is Alice allowed to delete a post? Is Alice an admin? Can Alice access the billing page? Those are authorization questions.

```js
// Authorization: checking permissions
const deletePost = (req, res, next) => {
  const post = await Post.findById(req.params.id);

  // Is this user the author or an admin?
  if (post.authorId !== req.user.id && req.user.role !== 'admin') {
    return res.status(403).json({ message: 'Forbidden' });
  }

  await post.deleteOne();
  res.status(204).send();
};
```

## The Key Difference

The simplest way I remember it:

- **Authentication** = Can you enter the building? (Checking your ID badge)
- **Authorization** = Can you enter the server room? (Checking your access level)

```js
// 401 Unauthorized = Authentication failed (wrong password, missing token)
res.status(401).json({ message: 'Please log in' });

// 403 Forbidden = Authorization failed (you are logged in but not allowed)
res.status(403).json({ message: 'You do not have permission' });
```

The HTTP status codes are confusing here. 401 is called "Unauthorized" but it actually means "Unauthenticated." 403 "Forbidden" is the true authorization failure. I had to look this up multiple times before it stuck.

## How They Work Together

In my Express apps, the flow is always:

1. **Authenticate first**: Middleware checks the token or session
2. **Authorize second**: Middleware checks the user's role or ownership

```js
// Step 1: Authenticate (verify identity)
const authenticate = async (req, res, next) => {
  const token = req.headers.authorization?.split(' ')[1];
  if (!token) return res.status(401).json({ message: 'Missing token' });

  const decoded = jwt.verify(token, process.env.JWT_SECRET);
  req.user = await User.findById(decoded.id);
  next();
};

// Step 2: Authorize (check permissions)
const authorize = (...roles) => {
  return (req, res, next) => {
    if (!roles.includes(req.user.role)) {
      return res.status(403).json({ message: 'Forbidden' });
    }
    next();
  };
};

// Usage
app.delete('/admin/users/:id', authenticate, authorize('admin'), deleteUser);
```

Authentication comes first because authorization makes no sense if I do not know who the user is.
