# Database Migrations

When I first started, I changed my database schema by hand. I would open a database console, run ALTER TABLE commands, and hope nothing broke. Then I deployed to production and realized my local database and the production database were completely out of sync. That is when I learned about migrations.

## What Are Migrations?

A migration is a versioned change to your database schema. Instead of making changes manually, you write a migration file that describes the change. Then you run a command to apply it.

Migrations give me:
- **A history** of every schema change, in order
- **The ability to roll back** a change if something goes wrong
- **Consistency** between development and production databases
- **Team coordination** so everyone's database stays in sync

## Prisma Migrate

With Prisma, migrations are built right in. After I modify my schema, I create a migration:

```bash
# After editing prisma/schema.prisma
npx prisma migrate dev --name add_user_avatar
```

This command does three things:
1. Creates a new migration file in `prisma/migrations/`
2. Applies the migration to your development database
3. Regenerates the Prisma Client

The migration file looks like this:

```sql
-- prisma/migrations/20240315103000_add_user_avatar/migration.sql
ALTER TABLE "User" ADD COLUMN "avatar" TEXT;
```

## Writing Migrations

Most migrations are generated automatically from schema changes. But sometimes I need custom SQL:

```bash
# Create an empty migration for custom SQL
npx prisma migrate dev --create-only --name custom_migration
```

Then I edit the SQL file:

```sql
-- Add a full-text search index
CREATE INDEX CONCURRENTLY IF NOT EXISTS "posts_content_idx"
ON "Post" USING gin(to_tsvector('english', "content"));
```

And apply it:

```bash
npx prisma migrate dev
```

## Seeding

Seeding populates the database with initial data. I create a seed file:

```js
// prisma/seed.js
const { PrismaClient } = require('@prisma/client');
const prisma = new PrismaClient();

async function main() {
  // Create default admin user
  await prisma.user.upsert({
    where: { email: 'admin@example.com' },
    update: {},
    create: {
      name: 'Admin',
      email: 'admin@example.com',
      role: 'admin',
    },
  });

  // Create sample posts
  await prisma.post.createMany({
    data: [
      { title: 'First Post', content: 'Welcome!', authorId: 1 },
      { title: 'Second Post', content: 'Hello world', authorId: 1 },
    ],
    skipDuplicates: true,
  });

  console.log('Seed data created');
}

main()
  .catch(console.error)
  .finally(() => prisma.$disconnect());
```

I configure the seed command in `package.json`:

```json
{
  "prisma": {
    "seed": "node prisma/seed.js"
  }
}
```

Then run it:

```bash
npx prisma db seed
```

## Production Migrations

In production, I never use `migrate dev`. That command can reset data. Instead, I deploy migrations with:

```bash
npx prisma migrate deploy
```

This applies pending migrations without any development features. It is safe for production. I run this as part of my deployment pipeline, before the new code starts serving requests.

```bash
# Typical deployment script
npx prisma migrate deploy
npx prisma generate
pm2 restart app
```

Migrations took me from chaos to control. Now every schema change is tracked, tested, and applied consistently across every environment.
