# Behind Proxies

When Express runs behind Nginx, a load balancer, or a CDN, the request information you see is not quite right. The IP address shows the proxy's IP, not the client's. The protocol shows HTTP even if the client used HTTPS. Let me fix this.

## The Problem

Here is what happens when a request goes through a proxy:

```
Client (203.0.113.5, HTTPS) -> Nginx (127.0.0.1, HTTP) -> Express
```

Inside Express, `req.ip` returns `127.0.0.1` and `req.protocol` returns `http`. That is the Nginx connection, not the client connection. For logging, rate limiting, and security, I need the real client information.

## Trust Proxy

Express provides the `trust proxy` setting to solve this. When enabled, Express reads the `X-Forwarded-*` headers that proxies add.

```js
// Trust the first proxy (Nginx)
app.set('trust proxy', 1);
```

Now `req.ip` returns the real client IP and `req.protocol` returns the real protocol.

### Trust Proxy Options

The value you pass to `trust proxy` matters:

```js
// Trust a specific number of proxies
app.set('trust proxy', 1);  // Trust one proxy (common for Nginx)

// Trust specific IPs
app.set('trust proxy', '127.0.0.1, 192.168.1.1');

// Trust any proxy (dangerous in production)
app.set('trust proxy', true);

// Trust based on loopback address
app.set('trust proxy', 'loopback');
```

In most setups, `1` is the right choice. You have one proxy (Nginx) in front of Express.

## X-Forwarded-For

The `X-Forwarded-For` header contains the chain of IP addresses:

```
X-Forwarded-For: 203.0.113.5, 70.41.3.18
```

The first IP is the original client. Subsequent IPs are proxies the request passed through. When `trust proxy` is enabled, Express parses this header and sets `req.ip` to the rightmost untrusted IP.

## X-Forwarded-Proto

This header tells Express what protocol the client used:

```
X-Forwarded-Proto: https
```

When `trust proxy` is enabled, `req.protocol` reads from this header instead of the direct connection.

## Configuring Nginx

Nginx needs to send these headers. I add them in the Nginx config:

```nginx
location / {
    proxy_pass http://127.0.0.1:3000;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
}
```

- `X-Real-IP`: The direct client IP
- `X-Forwarded-For`: Appends the client IP to the existing chain
- `X-Forwarded-Proto`: The original protocol (`http` or `https`)

## Security Warning

Do not set `trust proxy` to `true` in production. A malicious client can spoof the `X-Forwarded-For` header and hide their real IP. Always specify the exact number of trusted proxies or their IP addresses.

If your app is behind Cloudflare or AWS ALB, the proxy count might be 2 instead of 1:

```js
// Behind Cloudflare (1) + Nginx (1) = 2 proxies
app.set('trust proxy', 2);
```

Getting `trust proxy` right means your logs show real IPs, your rate limiting works correctly, and your security checks are not bypassed. It is a small setting with big consequences.
