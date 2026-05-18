# JSON Web Tokens (JWT)

JWT is the most common way I handle authentication in my Express APIs. Let me explain what it is and how it works.

## What JWT Is

A JSON Web Token is a string that contains encoded data. It has three parts separated by dots:

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VySWQiOjEyMywiaWF0IjoxNjk0ODM2ODAwfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

Those three parts are:

1. **Header**: The algorithm and token type
2. **Payload**: The actual data (user ID, role, etc.)
3. **Signature**: A cryptographic signature that proves the token has not been tampered with

```js
// Decoded header
{ "alg": "HS256", "typ": "JWT" }

// Decoded payload
{ "userId": 123, "iat": 1694836800 }
```

## How It Works

The flow is simple:

1. User sends their credentials (email and password)
2. Server verifies the credentials
3. Server creates a JWT and sends it back
4. Client stores the token (usually in localStorage or a cookie)
5. Client sends the token with every request in the Authorization header
6. Server verifies the token on each request

```
Client                         Server
  |                               |
  |--- POST /login (credentials) -->|
  |<-- JWT token ------------------|
  |                               |
  |--- GET /profile (with JWT) --->|
  |<-- user data ------------------|
```

The key insight is that the server does not need to store the token. The token itself contains the user data and a signature that proves it was issued by the server. This is called stateless authentication.

## Access vs Refresh Tokens

I never use a single long-lived token. Instead, I use two:

**Access token**: Short-lived (15 minutes). Used for API requests. If stolen, it expires quickly.

**Refresh token**: Long-lived (7 days). Used only to get a new access token. Stored more securely.

```js
// Access token: short expiration
const accessToken = jwt.sign(
  { userId: user.id },
  process.env.JWT_SECRET,
  { expiresIn: '15m' }
);

// Refresh token: long expiration
const refreshToken = jwt.sign(
  { userId: user.id },
  process.env.JWT_REFRESH_SECRET,
  { expiresIn: '7d' }
);
```

Why two tokens? If an attacker steals an access token, they can only use it for 15 minutes. If they steal a refresh token, I can revoke it by removing it from my database. With a single long-lived token, I have no way to revoke access without changing the secret, which logs out every user.

## What Goes in the Payload

I keep the payload small. JWTs are sent with every request, so large tokens mean more bandwidth:

```js
// Good: minimal payload
{ userId: 123, role: 'user' }

// Bad: too much data
{ userId: 123, name: 'Alice', email: 'alice@example.com', avatar: '...', bio: '...' }
```

I store only the user ID and maybe the role. Everything else, I look up from the database when needed.

## What JWT Is Not

JWT is not encrypted. Anyone can decode the payload with base64. The signature only proves the token was not modified, not that its contents are secret. I never put sensitive data like passwords in a JWT.
