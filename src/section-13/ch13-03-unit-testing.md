# Unit Testing

Unit testing is where I spend most of my testing time. These tests are fast, focused, and catch bugs before they spread. In an Express app, I unit test utilities, middleware, and any standalone logic.

## Testing a Utility Function

Let us start with something simple. Here is a validation utility:

```javascript
// src/utils/validate.js
function validateEmail(email) {
  const regex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return regex.test(email);
}

module.exports = { validateEmail };
```

The test:

```javascript
// tests/utils/validate.test.js
const { validateEmail } = require('../../src/utils/validate');

describe('validateEmail', () => {
  it('returns true for valid emails', () => {
    expect(validateEmail('user@example.com')).toBe(true);
  });

  it('returns false for missing @', () => {
    expect(validateEmail('userexample.com')).toBe(false);
  });

  it('returns false for empty string', () => {
    expect(validateEmail('')).toBe(false);
  });
});
```

## Testing Middleware

Middleware functions can be tricky since they depend on `req`, `res`, and `next`. I create mock objects for those:

```javascript
// tests/middleware/auth.test.js
const { requireAuth } = require('../../src/middleware/auth');

describe('requireAuth middleware', () => {
  it('calls next() when user is authenticated', () => {
    const req = { session: { userId: '123' } };
    const res = {};
    const next = jest.fn();

    requireAuth(req, res, next);

    expect(next).toHaveBeenCalled();
  });

  it('returns 401 when no user in session', () => {
    const req = { session: {} };
    const res = { status: jest.fn().mockReturnThis(), json: jest.fn() };
    const next = jest.fn();

    requireAuth(req, res, next);

    expect(res.status).toHaveBeenCalledWith(401);
    expect(next).not.toHaveBeenCalled();
  });
});
```

## Mocking with jest.fn and jest.mock

Mocking lets you replace real dependencies with controlled fakes. This keeps unit tests isolated.

**jest.fn()** creates a mock function you can track:

```javascript
const callback = jest.fn();
callback('hello');
expect(callback).toHaveBeenCalledWith('hello');
expect(callback).toHaveBeenCalledTimes(1);
```

**jest.mock()** replaces an entire module:

```javascript
// Mock the database module
jest.mock('../../src/models/User', () => ({
  findById: jest.fn().mockResolvedValue({ id: '1', name: 'Alice' }),
}));

const User = require('../../src/models/User');

test('finds user by id', async () => {
  const user = await User.findById('1');
  expect(user.name).toBe('Alice');
  expect(User.findById).toHaveBeenCalledWith('1');
});
```

The power of mocking is that your tests do not need a real database. They run instantly and produce consistent results. I make sure to reset mocks between tests with `afterEach(() => jest.clearAllMocks())` so one test does not leak into another.
