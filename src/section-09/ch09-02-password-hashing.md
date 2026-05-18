# Password Hashing

If I take away one thing from this entire book, it is this: never store passwords in plain text. Not in a database. Not in a config file. Not anywhere. If someone gets access to my database, plain text passwords mean every user's account is instantly compromised. Hashing prevents that.

## What Hashing Is

A hash function takes an input and produces a fixed-size output that cannot be reversed. The same input always produces the same output, but you cannot go from the output back to the input.

```js
const bcrypt = require('bcrypt');

// Hashing a password
const password = 'my-secret-password';
const hash = await bcrypt.hash(password, 12);

console.log(hash);
// $2b$12$N9qo8uLOickgx2ZMRZoMyeIjZAgcfl7p92ldGxad68LJZdL17lhWy
```

Even though "my-secret-password" always hashes to the same value with the same salt, the output looks nothing like the input. And there is no function that takes that hash and gives me back "my-secret-password".

## Salting

A salt is random data added to the password before hashing. Why? Because without it, the same password always produces the same hash. An attacker with a list of common passwords and their hashes (a rainbow table) could quickly find matches.

```js
// bcrypt generates a unique salt automatically
const hash1 = await bcrypt.hash('password123', 12);
const hash2 = await bcrypt.hash('password123', 12);

console.log(hash1 === hash2); // false! Different salts, different hashes
```

bcrypt includes the salt right in the hash string, so I do not need to store it separately. When I verify a password, bcrypt extracts the salt from the stored hash and uses it.

## Cost Factor

The second argument to `bcrypt.hash` is the cost factor (also called rounds). It determines how much processing time the hash takes. A cost of 12 means 2^12 (4096) iterations.

```js
// Lower cost = faster but less secure
const hash = await bcrypt.hash(password, 4);   // Fast, less secure

// Higher cost = slower but more secure
const hash = await bcrypt.hash(password, 14);  // Slow, very secure

// My default: 12
const hash = await bcrypt.hash(password, 12);  // Good balance
```

Higher cost means an attacker has to spend more computing power to guess passwords. I use 12 for most projects. If security is extra important, I go higher. The tradeoff is that login takes longer, which affects user experience.

## Verifying Passwords

I never compare passwords directly. I use bcrypt's compare function:

```js
const bcrypt = require('bcrypt');

// During login
const user = await User.findOne({ email: req.body.email });

if (!user) {
  return res.status(401).json({ message: 'Invalid credentials' });
}

const isMatch = await bcrypt.compare(req.body.password, user.passwordHash);

if (!isMatch) {
  return res.status(401).json({ message: 'Invalid credentials' });
}

// Password is correct, log the user in
```

Notice I return the same error message whether the email is wrong or the password is wrong. This prevents an attacker from discovering which emails exist in my system.

## Registration Example

```js
router.post('/register', async (req, res, next) => {
  try {
    const { name, email, password } = req.body;

    const existingUser = await User.findOne({ email });
    if (existingUser) {
      return res.status(400).json({ message: 'Email already registered' });
    }

    const passwordHash = await bcrypt.hash(password, 12);

    const user = await User.create({ name, email, passwordHash });

    res.status(201).json({ id: user.id, name: user.name, email: user.email });
  } catch (err) {
    next(err);
  }
});
```

bcrypt is not the fastest library, and that is the point. Speed is the enemy of password security. A slow hash means brute force attacks take impractically long.
