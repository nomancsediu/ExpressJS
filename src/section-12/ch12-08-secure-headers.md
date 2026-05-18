# Security Headers

HTTP headers are your first line of defense. They tell browsers how to handle your content safely. Most of these are set automatically by Helmet, but understanding what they do helps you configure them correctly.

## X-Content-Type-Options

Prevents browsers from guessing (sniffing) the content type:

```
X-Content-Type-Options: nosniff
```

Without this header, a browser might interpret a JSON file as HTML and execute embedded scripts. With `nosniff`, the browser strictly follows the `Content-Type` header you set.

## Strict-Transport-Security (HSTS)

Tells browsers to only use HTTPS for future requests:

```
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
```

- `max-age=31536000`: Remember this for 1 year
- `includeSubDomains`: Apply to all subdomains
- `preload`: Eligible for browser HSTS preload lists

Once a browser sees this header, it refuses to connect via HTTP for the specified duration. Even if a user types `http://`, the browser converts it to `https://`.

Be careful with HSTS. Start with a short `max-age` like `300` (5 minutes) to test, then increase it. If something goes wrong with your HTTPS setup, users will be locked out for the entire `max-age` duration.

## X-Frame-Options

Prevents your page from being embedded in iframes on other sites:

```
X-Frame-Options: DENY
```

Options:
- `DENY`: No framing allowed at all
- `SAMEORIGIN`: Only your own site can frame it

This stops clickjacking attacks where a malicious site overlays invisible iframes to trick users into clicking things.

## Referrer-Policy

Controls how much referrer information is sent when users click links:

```
Referrer-Policy: strict-origin-when-cross-origin
```

Common values:
- `no-referrer`: Never send referrer
- `same-origin`: Only send for same-site navigation
- `strict-origin-when-cross-origin`: Send full URL for same-site, only origin for cross-site

## Permissions-Policy

Controls which browser features your page can use:

```
Permissions-Policy: camera=(), microphone=(), geolocation=(self)
```

This disables camera and microphone access entirely and allows geolocation only from the same origin. Even if an injected script tries to access these features, the browser blocks it.

## Cross-Origin Headers

Several headers control cross-origin behavior:

```
Cross-Origin-Opener-Policy: same-origin
Cross-Origin-Resource-Policy: same-origin
Cross-Origin-Embedder-Policy: require-corp
```

These prevent other sites from reading your resources or embedding your content. They are especially important for APIs that serve sensitive data.

## Setting Headers Manually

If you do not use Helmet, you can set headers yourself:

```js
app.use((req, res, next) => {
  res.setHeader('X-Content-Type-Options', 'nosniff');
  res.setHeader('X-Frame-Options', 'DENY');
  res.setHeader('X-XSS-Protection', '0');  // Disabled, CSP is better
  res.setHeader('Referrer-Policy', 'strict-origin-when-cross-origin');
  res.setHeader('Permissions-Policy', 'camera=(), microphone=(), geolocation=(self)');
  next();
});
```

But honestly, just use Helmet. It sets all these headers correctly and keeps them updated as standards evolve. Manual header management is error-prone and easy to forget when new headers are introduced.
