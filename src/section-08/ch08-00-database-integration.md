# Database Integration

Every real app needs to store data. So far I have been building Express routes that return hardcoded data or do not persist anything. That changes now. In this section, I connect Express to real databases.

Choosing a database is one of the first big decisions I make when starting a project. There is no single right answer. It depends on what kind of data I am working with, how much I need to scale, and what my team is comfortable with.

The two main families I work with are:

- **SQL databases** like PostgreSQL and MySQL. They use tables, rows, and strict schemas. Great for relational data and complex queries.
- **NoSQL databases** like MongoDB and Redis. They use flexible document or key-value structures. Great for fast iteration and unstructured data.

I used to think I had to pick one and stick with it. In practice, many apps use more than one. PostgreSQL for the main data, Redis for caching, MongoDB for logs. Each tool has its strengths.

This section covers the databases I use most often with Express:

- **MongoDB with Mongoose** for document-based data with a friendly ODM
- **PostgreSQL with Prisma** for relational data with type-safe queries
- **Redis** for caching and fast key-value lookups

I also cover migrations, which is how I manage changes to my database schema over time without losing data. This is something I ignored for too long, and it caused real problems when I needed to update my tables in production.

By the end of this section, you will be able to connect Express to multiple databases, perform CRUD operations, write queries, and manage your schema as your app evolves. Let me start with MongoDB and Mongoose, which is where my own Express journey began.
