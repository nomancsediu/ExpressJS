# CSRF Protection

Cross-Site Request Forgery, or CSRF, tricks users into making requests they did not intend. Imagine you are logged into your bank. You visit a malicious site that has a hidden form submitting a transfer request to the bank. Since your browser sends your cookies automatically, the bank thinks the request is legitimate.

That is a CSRF attack. And it is sneaky because the user never knows it happened.

## How CSRF Works

1. User logs into your app (gets a session cookie)
2. User visits a malicious site
3. The malicious site sends a request to your app
4. The browser includes the session cookie automatically
5. Your app processes the request as if the user sent it

## CSRF Tokens

The solution is to include a secret token in your forms that the malicious site cannot guess:

```bash
npm install csrf-csrf
```

```js
const { doubleCsrf } = require('csrf-csrf');

const { generateToken, doubleCsrfProtection } = doubleCsrf({
  getSecret: () => 'your-secret-key',
  cookieName: '__Host-csrf',
  cookieOptions: {
    sameSite: 'lax',
    path: '/',
    secure: true,
    httpOnly: true
  },
  size: 64,
  ignoredMethods: ['GET', 'HEAD', 'OPTIONS']
});

app.use(doubleCsrfProtection);
```

## Using Tokens in Forms

```js
app.get('/form', (req, res) => {
  const csrfToken = generateToken(req, res);
  res.render('form', { csrfToken });
});
```

In EJS:

```html
<form method="POST" action="/submit">
  <input type="hidden" name="_csrf" value="<%= csrfToken %>" />
  <input type="text" name="message" />
  <button type="submit">Submit</button>
</form>
```

In API responses:

```js
app.get('/api/csrf-token', (req, res) => {
  const csrfToken = generateToken(req, res);
  res.json({ csrfToken });
});
```

Your frontend JavaScript reads the token and includes it in request headers:

```js
fetch('/api/submit', {
  method: 'POST',
  headers: {
    'X-CSRF-Token': csrfToken,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({ message: 'hello' })
});
```

## Double Submit Cookie

The `csrf-csrf` package uses the double submit cookie pattern. The token is stored in both a cookie and the request body or header. The server verifies both match. A malicious site cannot read the cookie due to Same-Origin Policy, so it cannot include the correct token.

## SameSite Cookies

Modern browsers support the `SameSite` attribute, which provides built-in CSRF protection:

```js
app.use(session({
  cookie: {
    sameSite: 'lax',  // or 'strict'
    secure: true,
    httpOnly: true
  }
}));
```

- `strict` blocks all cross-site cookie sends, even from following links
- `lax` allows cookies from top-level GET navigations but blocks POST
- `none` allows all cross-site sends (requires `secure: true`)

With `SameSite: lax`, most CSRF attacks are already blocked. But do not rely on it alone. Not all browsers handle it the same way, and there are edge cases. Use both SameSite cookies and CSRF tokens.

## Error Handling

```js
app.use((err, req, res, next) => {
  if (err.code === 'EBADCSRFTOKEN') {
    return res.status(403).send('CSRF token mismatch');
  }
  next(err);
});
```

I used to skip CSRF protection on APIs because "APIs use tokens, not cookies." But if your API accepts cookies for auth, CSRF is a real threat. Always protect state-changing endpoints.
