# PostgreSQL with Prisma

MongoDB is great, but sometimes I need a relational database. PostgreSQL is my SQL database of choice, and Prisma is the tool I use to interact with it. Let me explain why.

## Why PostgreSQL?

PostgreSQL is a powerful, open-source relational database. It enforces data integrity with schemas, supports complex joins, and handles transactions across multiple tables. When my data has clear relationships (users have posts, posts have comments, comments belong to users), PostgreSQL shines.

## Why Prisma?

Prisma is a next-generation ORM for Node.js and TypeScript. I switched to it after struggling with raw SQL queries and older ORMs. Here is what I love about it:

- **Type-safe queries**: I get autocomplete and type checking on my database queries
- **Declarative schema**: I define my data model in a simple `.prisma` file
- **Auto-generated client**: Prisma generates a type-safe client from my schema
- **Migrations built-in**: I can create and apply database migrations without extra tools

## Setup

First, I install Prisma:

```bash
npm install prisma --save-dev
npm install @prisma/client
npx prisma init
```

This creates a `prisma/schema.prisma` file and a `.env` file.

## Schema Definition

I define my data model in the Prisma schema:

```prisma
// prisma/schema.prisma

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id        Int      @id @default(autoincrement())
  name      String
  email     String   @unique
  createdAt DateTime @default(now())
  posts     Post[]
}

model Post {
  id        Int      @id @default(autoincrement())
  title     String
  content   String?
  published Boolean  @default(false)
  authorId  Int
  author    User     @relation(fields: [authorId], references: [id])
  createdAt DateTime @default(now())
}
```

The `@relation` attribute defines the foreign key. Prisma understands that a User has many Posts and a Post belongs to a User.

## Client Setup

After running `npx prisma generate`, I create a singleton client:

```js
const { PrismaClient } = require('@prisma/client');
const prisma = new PrismaClient();

module.exports = prisma;
```

## CRUD Operations

```js
// Create
const user = await prisma.user.create({
  data: {
    name: 'Alice',
    email: 'alice@example.com',
  },
});

// Create with related record
const post = await prisma.post.create({
  data: {
    title: 'My First Post',
    content: 'Hello world',
    author: {
      connect: { id: 1 },
    },
  },
});

// Read
const users = await prisma.user.findMany();

// Read with relations
const usersWithPosts = await prisma.user.findMany({
  include: { posts: true },
});

// Update
await prisma.user.update({
  where: { id: 1 },
  data: { name: 'Alice Updated' },
});

// Delete
await prisma.user.delete({
  where: { id: 1 },
});
```

Prisma queries feel natural. I can include relations, filter, sort, and paginate, all with a consistent API. The type safety catches errors before my code even runs.
