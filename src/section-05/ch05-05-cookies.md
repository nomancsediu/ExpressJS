# Cookies

Cookies are small pieces of data that the server sends to the browser, and the browser sends back on every subsequent request. They are one of the oldest ways to maintain state in HTTP, and they are still everywhere.

## Reading Cookies

Install `cookie-parser` and add it as middleware:

```bash
npm install cookie-parser
```

```js
const cookieParser = require('cookie-parser');
app.use(cookieParser());

app.get('/check', (req, res) => {
  console.log(req.cookies);        // { theme: 'dark', lang: 'en' }
  console.log(req.cookies.theme);  // 'dark'
  res.send('Checked cookies');
});
```

## Setting Cookies

Use `res.cookie()` to send a cookie to the browser:

```js
app.get('/set-theme', (req, res) => {
  res.cookie('theme', 'dark');
  res.send('Theme cookie set');
});
```

## Cookie Options

Cookies have several important options that control their behavior:

```js
res.cookie('token', 'abc123', {
  maxAge: 86400000,     // expires in 1 day (in milliseconds)
  httpOnly: true,       // not accessible from JavaScript
  secure: true,         // only sent over HTTPS
  sameSite: 'strict',   // prevents CSRF
  domain: '.myapp.com', // available on subdomains
  path: '/admin',       // only sent for /admin routes
  encode: encodeURIComponent, // encoding function
});
```

The options I always use in production are `httpOnly`, `secure`, and `sameSite`. Here is why:

- `httpOnly: true` prevents JavaScript from reading the cookie. This stops XSS attacks from stealing session tokens.
- `secure: true` ensures the cookie is only sent over HTTPS. No one can intercept it.
- `sameSite: 'strict'` means the cookie is only sent for same-site requests. This blocks most CSRF attacks.

## Signed Cookies

Signed cookies cannot be tampered with. You provide a secret string, and cookie-parser adds a signature:

```js
app.use(cookieParser('my-secret-key'));

app.get('/set', (req, res) => {
  res.cookie('userId', '42', { signed: true });
  res.send('Signed cookie set');
});

app.get('/get', (req, res) => {
  console.log(req.signedCookies.userId); // '42' if valid, false if tampered
  res.json(req.signedCookies);
});
```

If someone modifies the cookie value, the signature no longer matches and `req.signedCookies` returns `false` for that cookie.

## Clearing Cookies

```js
app.get('/logout', (req, res) => {
  res.clearCookie('token');
  res.clearCookie('userId');
  res.send('Logged out');
});
```

The cookie options must match the options used when setting it (like `path` and `domain`), otherwise the browser will not delete it.

## Cookies vs Local Storage

I used to wonder when to use cookies vs local storage. Here is my simple rule:
- Cookies are sent automatically with every request. Great for auth tokens and session IDs.
- Local storage is only accessible from JavaScript on the client. Great for preferences and cached data.

For anything security-related, I use `httpOnly` cookies. For UI preferences, local storage is fine.
