# Nginx Reverse Proxy

Nginx sits in front of Express and handles the things Express is not great at: SSL termination, static files, load balancing, and rate limiting. Think of Nginx as the bouncer at the door and Express as the bartender inside.

## Why Use Nginx?

Running Express directly on port 80 or 443 works, but Nginx gives you several advantages:

- **SSL termination**: Nginx handles HTTPS so Express only deals with HTTP
- **Static files**: Nginx serves images, CSS, and JS much faster than Express
- **Load balancing**: Distribute traffic across multiple Express instances
- **Compression**: Nginx compresses responses efficiently
- **Security**: Hide Express behind Nginx, add extra protection layers

## Basic Configuration

Install Nginx on your server:

```bash
sudo apt update
sudo apt install nginx
```

Create a configuration file for the app:

```nginx
# /etc/nginx/sites-available/ecommerce
server {
    listen 80;
    server_name api.yourdomain.com;

    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }

    # Serve uploaded files directly
    location /uploads/ {
        alias /home/ubuntu/ecommerce-api/uploads/;
        expires 30d;
        add_header Cache-Control "public, immutable";
    }
}
```

Enable the site and restart Nginx:

```bash
sudo ln -s /etc/nginx/sites-available/ecommerce /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

The `proxy_set_header` lines pass the original request information to Express. This is important so Express knows the real IP and protocol of the client.

## Load Balancing

When one Express instance is not enough, run multiple instances and let Nginx distribute traffic:

```nginx
upstream express_app {
    least_conn;
    server 127.0.0.1:3000;
    server 127.0.0.1:3001;
    server 127.0.0.1:3002;
}

server {
    listen 80;
    server_name api.yourdomain.com;

    location / {
        proxy_pass http://express_app;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

The `least_conn` directive sends traffic to the instance with the fewest active connections. Other options include `round_robin` (default) and `ip_hash` (sticky sessions).

Start multiple Express instances with PM2:

```bash
pm2 start src/server.js -i 3 --name ecommerce
```

The `-i 3` flag starts 3 instances in cluster mode.

## Rate Limiting at Nginx Level

Nginx can rate limit before requests even reach Express:

```nginx
limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;

server {
    location /api/ {
        limit_req zone=api burst=20 nodelay;
        proxy_pass http://express_app;
    }
}
```

This allows 10 requests per second with a burst of 20. Nginx handles it efficiently without Express doing any work.

## Trust Proxy in Express

When Express is behind Nginx, I need to tell it to trust the proxy headers:

```js
app.set('trust proxy', 1);
```

This makes `req.ip` return the real client IP instead of the Nginx IP. The `1` means trust the first proxy (Nginx).

Nginx plus Express is a proven combination. Nginx handles the front line, Express handles the application logic. Together they are fast and reliable.
