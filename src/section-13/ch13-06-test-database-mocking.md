# Test Database Mocking

Running tests against a real database is slow and fragile. I learned this the hard way when my tests started failing because another developer changed the shared dev database. Now I use dedicated strategies for test databases.

## Using a Separate Test Database

The simplest approach is a separate test database. I configure it through environment variables:

```javascript
// tests/helpers/db.js
const mongoose = require('mongoose');

beforeAll(async () => {
  const uri = process.env.MONGODB_TEST_URI || 'mongodb://localhost/express_test';
  await mongoose.connect(uri);
});

afterAll(async () => {
  await mongoose.connection.dropDatabase();
  await mongoose.connection.close();
});

afterEach(async () => {
  const collections = mongoose.connection.collections;
  for (const key in collections) {
    await collections[key].deleteMany({});
  }
});
```

The `afterEach` cleanup is crucial. Without it, data from one test leaks into the next.

## mongodb-memory-server

An even better option is `mongodb-memory-server`. It runs MongoDB entirely in memory, which makes tests lightning fast:

```bash
npm install --save-dev mongodb-memory-server
```

```javascript
const { MongoMemoryServer } = require('mongodb-memory-server');
const mongoose = require('mongoose');

let mongoServer;

beforeAll(async () => {
  mongoServer = await MongoMemoryServer.create();
  const uri = mongoServer.getUri();
  await mongoose.connect(uri);
});

afterAll(async () => {
  await mongoose.disconnect();
  await mongoServer.stop();
});
```

No external MongoDB instance needed. This is perfect for CI environments where installing MongoDB is a hassle.

## Factory Functions

Instead of manually creating test data in every test, I use factory functions. They keep tests clean and easy to read:

```javascript
// tests/helpers/factories.js
function createUser(overrides = {}) {
  return {
    name: 'Test User',
    email: `test${Date.now()}@example.com`,
    password: 'password123',
    ...overrides,
  };
}

function createPost(overrides = {}) {
  return {
    title: 'Test Post',
    content: 'This is test content.',
    ...overrides,
  };
}

module.exports = { createUser, createPost };
```

Using factories in tests:

```javascript
const { createUser } = require('../helpers/factories');

test('saves a user to the database', async () => {
  const userData = createUser({ name: 'Alice' });
  const user = new User(userData);
  await user.save();

  const found = await User.findOne({ name: 'Alice' });
  expect(found.email).toBe(userData.email);
});
```

The spread operator in factories means I can override any field I want while getting sensible defaults for the rest. This pattern saves me a ton of repetitive test setup code and makes each test focused on what it is actually testing.
