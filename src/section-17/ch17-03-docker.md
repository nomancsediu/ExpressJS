# Docker

"Works on my machine" is a running joke for a reason. Docker solves this by packaging your app with everything it needs to run. Same container, same behavior, everywhere.

## Dockerfile

I start with a simple Dockerfile for the Express app:

```dockerfile
# Dockerfile
FROM node:20-alpine

WORKDIR /app

COPY package*.json ./
RUN npm ci --only=production

COPY . .

EXPOSE 3000

USER node

CMD ["node", "src/server.js"]
```

This is fine for getting started. But I can do better with a multi-stage build.

## Multi-Stage Build

Multi-stage builds keep the final image small by separating the build stage from the runtime stage.

```dockerfile
# Stage 1: Build
FROM node:20-alpine AS builder

WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY . .

# Stage 2: Production
FROM node:20-alpine

WORKDIR /app

COPY package*.json ./
RUN npm ci --only=production && npm cache clean --force

COPY --from=builder /app/src ./src

EXPOSE 3000

USER node

CMD ["node", "src/server.js"]
```

The builder stage installs all dependencies (including devDependencies). The production stage only copies production dependencies and source code. The final image is much smaller.

## .dockerignore

Just like `.gitignore`, I need a `.dockerignore` to keep unnecessary files out of the image:

```text
# .dockerignore
node_modules
npm-debug.log
.env
.env.*
.git
.gitignore
docker-compose*.yml
Dockerfile
README.md
uploads/*
```

This prevents the local `node_modules` from being copied in. It also keeps `.env` files out of the image. Secrets should never be baked into containers.

## Docker Compose

For local development, I use Docker Compose to run the app and MongoDB together:

```yaml
# docker-compose.yml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=development
      - MONGODB_URI=mongodb://mongo:27017/ecommerce-dev
      - JWT_SECRET=dev-secret-key
      - JWT_REFRESH_SECRET=dev-refresh-secret
    depends_on:
      - mongo
    volumes:
      - ./src:/app/src
      - ./uploads:/app/uploads

  mongo:
    image: mongo:7
    ports:
      - "27017:27017"
    volumes:
      - mongo-data:/data/db

volumes:
  mongo-data:
```

Run it with:

```bash
docker compose up --build
```

The app starts, connects to MongoDB, and hot-reloads when I change files because of the volume mount. When I stop it, the data persists in the `mongo-data` volume.

## Production Docker Compose

For production, I add environment variables from a file and remove the volume mount for source code:

```yaml
# docker-compose.prod.yml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "3000:3000"
    env_file:
      - .env.production
    depends_on:
      - mongo
    restart: unless-stopped

  mongo:
    image: mongo:7
    volumes:
      - mongo-data:/data/db
    restart: unless-stopped

volumes:
  mongo-data:
```

Run with:

```bash
docker compose -f docker-compose.prod.yml up -d
```

Docker gives me consistency. The same container that runs on my machine runs on the server. No surprises.
