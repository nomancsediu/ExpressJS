# Testing with Supertest

Supertest is my go-to library for testing Express routes. It lets you make HTTP requests to your app without actually starting a server. That makes tests fast and self-contained.

## Installing Supertest

```bash
npm install --save-dev supertest
```

For TypeScript, also add the types:

```bash
npm install --save-dev @types/supertest
```

## Making Requests

The key thing is to export your Express app without calling `app.listen()`. That way Supertest can use it directly:

```javascript
// src/app.js
const express = require('express');
const app = express();
app.use(express.json());

app.get('/api/health', (req, res) => {
  res.json({ status: 'ok' });
});

module.exports = app;
// Do NOT call app.listen() here
```

```javascript
// src/server.js
const app = require('./app');
app.listen(3000);
```

Now the tests:

```javascript
const request = require('supertest');
const app = require('../../src/app');

describe('Health endpoint', () => {
  it('returns ok status', async () => {
    const res = await request(app).get('/api/health');

    expect(res.status).toBe(200);
    expect(res.body.status).toBe('ok');
  });
});
```

## Testing POST Requests

```javascript
describe('POST /api/users', () => {
  it('creates a user with valid data', async () => {
    const res = await request(app)
      .post('/api/users')
      .send({ name: 'Alice', email: 'alice@test.com' })
      .set('Accept', 'application/json');

    expect(res.status).toBe(201);
    expect(res.body).toHaveProperty('id');
    expect(res.body.name).toBe('Alice');
  });

  it('returns 400 for missing fields', async () => {
    const res = await request(app)
      .post('/api/users')
      .send({ name: '' })
      .set('Accept', 'application/json');

    expect(res.status).toBe(400);
    expect(res.body).toHaveProperty('errors');
  });
});
```

## Testing Authenticated Routes

For routes that require authentication, I set the Authorization header:

```javascript
describe('GET /api/profile', () => {
  it('returns profile when authenticated', async () => {
    // First, log in to get a token
    const loginRes = await request(app)
      .post('/api/auth/login')
      .send({ email: 'alice@test.com', password: 'pass123' });

    const token = loginRes.body.token;

    // Then access the protected route
    const res = await request(app)
      .get('/api/profile')
      .set('Authorization', `Bearer ${token}`);

    expect(res.status).toBe(200);
    expect(res.body.email).toBe('alice@test.com');
  });

  it('returns 401 without token', async () => {
    const res = await request(app).get('/api/profile');
    expect(res.status).toBe(401);
  });
});
```

Supertest handles the HTTP layer cleanly, and I do not need to worry about ports or server startup. It just works with my Express app instance.
