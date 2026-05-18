# SSL and HTTPS

HTTP sends everything in plain text. Anyone between the user and your server can read passwords, tokens, and personal data. HTTPS encrypts all of that. In 2024 and beyond, HTTPS is not optional. Browsers warn users about HTTP sites, and some features only work over HTTPS.

## How SSL/TLS Works

SSL (Secure Sockets Layer) and its successor TLS (Transport Layer Security) encrypt the connection between the client and server. The process looks like this:

1. Client connects to the server and requests a secure connection
2. Server presents its SSL certificate
3. Client verifies the certificate with a Certificate Authority
4. Both sides agree on an encryption key
5. All data is encrypted for the rest of the session

You need an SSL certificate from a trusted Certificate Authority. Let's Encrypt gives you one for free.

## Getting a Certificate with Certbot

Let's Encrypt provides free SSL certificates. Certbot automates the whole process.

### Install Certbot

```bash
sudo apt install certbot python3-certbot-nginx
```

### Get a Certificate

```bash
sudo certbot --nginx -d api.yourdomain.com
```

Certbot reads your Nginx config, verifies you own the domain, gets a certificate, and updates your Nginx config to use it. It asks if you want to redirect HTTP to HTTPS. Say yes.

After Certbot runs, your Nginx config looks like this:

```nginx
server {
    listen 80;
    server_name api.yourdomain.com;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl http2;
    server_name api.yourdomain.com;

    ssl_certificate /etc/letsencrypt/live/api.yourdomain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/api.yourdomain.com/privkey.pem;

    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### Auto-Renewal

Let's Encrypt certificates expire after 90 days. Certbot installs a cron job that renews them automatically:

```bash
# Test the renewal process
sudo certbot renew --dry-run
```

If the dry run works, auto-renewal is set up. Certbot renews certificates 30 days before expiry.

## HSTS

HTTP Strict Transport Security tells browsers to always use HTTPS for your domain. Add this header in Nginx:

```nginx
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
```

Once a browser sees this header, it will not attempt HTTP connections to your domain for one year. Be careful: if you need to go back to HTTP for some reason, HSTS makes that difficult.

## Security Settings for Nginx

Tighten the SSL configuration for better security:

```nginx
ssl_protocols TLSv1.2 TLSv1.3;
ssl_ciphers HIGH:!aNULL:!MD5;
ssl_prefer_server_ciphers on;
ssl_session_cache shared:SSL:10m;
ssl_session_timeout 10m;
```

This disables old protocols and weak ciphers. TLS 1.2 and 1.3 are the only versions you need.

## Enforcing HTTPS in Express

Even with Nginx handling SSL, I add a safety check in Express:

```js
app.use((req, res, next) => {
  if (req.headers['x-forwarded-proto'] === 'http' && process.env.NODE_ENV === 'production') {
    return res.redirect(301, `https://${req.headers.host}${req.url}`);
  }
  next();
});
```

This catches any requests that somehow bypass Nginx's redirect. Double protection never hurts.

SSL used to be expensive and complicated. Let's Encrypt and Certbot made it free and automatic. There is no excuse to skip HTTPS anymore.
