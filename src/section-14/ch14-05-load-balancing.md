# Load Balancing

A single Express instance can only handle so many requests. When traffic grows, you need multiple instances and a way to distribute traffic among them. That is load balancing.

## The Concept

Load balancing sits between users and your servers. It receives incoming requests and forwards them to one of several backend instances. This gives you:

- **Higher throughput**: Multiple servers handle more requests
- **Reliability**: If one server crashes, others keep serving
- **Zero-downtime deploys**: Update one instance while others serve traffic

## Round-Robin

The simplest load balancing algorithm is round-robin. Each request goes to the next server in line:

```
Request 1 -> Server A
Request 2 -> Server B
Request 3 -> Server A
Request 4 -> Server B
```

This distributes load evenly, assuming all requests take similar resources.

## Nginx as a Load Balancer

Nginx is my go-to load balancer for Express apps. It is fast, reliable, and easy to configure:

```nginx
# /etc/nginx/conf.d/load-balancer.conf
upstream express_app {
  server 127.0.0.1:3001;
  server 127.0.0.1:3002;
  server 127.0.0.1:3003;
}

server {
  listen 80;

  location / {
    proxy_pass http://express_app;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
  }
}
```

This runs three Express instances on different ports and Nginx distributes traffic among them.

## Weighted Load Balancing

If one server is more powerful, you can assign it more traffic:

```nginx
upstream express_app {
  server 127.0.0.1:3001 weight=3;
  server 127.0.0.1:3002 weight=1;
}
```

Server A gets 75% of traffic, Server B gets 25%.

## Health Checks

Nginx can detect unhealthy backends and stop sending them traffic:

```nginx
upstream express_app {
  server 127.0.0.1:3001 max_fails=3 fail_timeout=30s;
  server 127.0.0.1:3002 max_fails=3 fail_timeout=30s;
}
```

If a server fails 3 times within 30 seconds, Nginx stops routing to it for 30 seconds.

## Horizontal Scaling

Load balancing enables horizontal scaling, which means adding more servers instead of upgrading one server (vertical scaling). Horizontal scaling is more cost-effective and has no upper limit.

My typical setup:

1. Run 2-4 Express instances on one machine (using all CPU cores)
2. Put Nginx in front as the load balancer
3. As traffic grows, add more machines
4. Use a shared database and Redis for session storage

The important thing is that your app must be stateless. No sticky sessions, no in-memory session storage. Every instance must be able to handle any request. This is why I store sessions in Redis rather than memory.
