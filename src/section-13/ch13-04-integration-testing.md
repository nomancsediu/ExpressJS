# Integration Testing

Unit tests are great for isolated logic, but they do not tell you if your routes actually work end to end. That is where integration tests come in. They verify that multiple parts of your app work together correctly.

## Testing Route Interactions

I like to test that my routes accept the right input and return the right output. Here is a basic setup using the Express app directly:

```javascript
const express = require('express');
const request = require('supertest');
const app = require('../../src/app'); // your Express app

describe('GET /api/users', () => {
  it('returns a list of users', async () => {
    const response = await request(app).get('/api/users');

    expect(response.status).toBe(200);
    expect(response.body).toHaveProperty('users');
    expect(Array.isArray(response.body.users)).toBe(true);
  });
});
```

Wait, I am getting ahead of myself. We will cover Supertest in detail later. The key idea here is that integration tests spin up your Express app (or parts of it) and make real requests through it.

## Testing with a Database

Integration tests often need a database. I set up a test database connection that is separate from my development database:

```javascript
// tests/helpers/setup.js
const mongoose = require('mongoose');

beforeAll(async () => {
  await mongoose.connect(process.env.TEST_DB_URI || 'mongodb://localhost/test_db');
});

afterAll(async () => {
  await mongoose.connection.dropDatabase();
  await mongoose.connection.close();
});
```

Then my route tests can create and query real data:

```javascript
describe('POST /api/users', () => {
  beforeEach(async () => {
    // Clean the collection before each test
    await User.deleteMany({});
  });

  it('creates a new user', async () => {
    const response = await request(app)
      .post('/api/users')
      .send({ name: 'Alice', email: 'alice@example.com' });

    expect(response.status).toBe(201);
    expect(response.body.name).toBe('Alice');

    // Verify it is actually in the database
    const user = await User.findOne({ email: 'alice@example.com' });
    expect(user).not.toBeNull();
  });
});
```

## Test Isolation

The most important lesson I learned about integration tests is isolation. Each test must be independent. If one test leaves data behind, the next test might fail unexpectedly.

I handle isolation with these strategies:

- **beforeEach**: Clean the database before each test
- **afterEach**: Reset any state or mocks
- **beforeAll/afterAll**: Set up and tear down the database connection once

```javascript
beforeEach(async () => {
  await User.deleteMany({});
  await Post.deleteMany({});
});

afterEach(() => {
  jest.clearAllMocks();
});
```

Without isolation, tests pass on one run and fail on the next. That flakiness drove me crazy until I started cleaning up properly between tests. Now my integration tests are reliable and repeatable.
