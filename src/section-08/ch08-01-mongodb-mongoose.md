# MongoDB and Mongoose

MongoDB was the first database I used with Express, and it is still my go-to for many projects. Let me explain what it is and why Mongoose makes it even better.

## What MongoDB Is

MongoDB is a NoSQL database that stores data as JSON-like documents instead of tables and rows. Each document lives in a collection, and each collection lives in a database.

```json
// A MongoDB document looks like this
{
  "_id": "65f1a2b3c4d5e6f7a8b9c0d1",
  "name": "Alice",
  "email": "alice@example.com",
  "age": 28,
  "hobbies": ["coding", "hiking"]
}
```

There is no fixed schema. One document can have five fields, another can have ten. This flexibility is great when my data structure changes often, which happens a lot in early-stage projects.

## What Mongoose Is

Mongoose is an ODM, which stands for Object Document Mapper. It sits between my Express app and MongoDB, giving me a structured way to define and interact with my data.

Why do I need it? Because raw MongoDB can be too flexible. Without any rules, I could accidentally store a string where a number should be, or forget a required field entirely. Mongoose adds schemas, validation, and helpful methods on top of MongoDB.

## The ODM Concept

An ODM maps objects in my code to documents in the database. I define what a User looks like in JavaScript, and Mongoose handles the translation to and from MongoDB.

```
My JavaScript Object  -->  Mongoose  -->  MongoDB Document
{ name, email, age }  -->  validates  -->  { _id, name, email, age }
```

Without Mongoose, I write MongoDB queries directly:

```js
// Raw MongoDB
db.collection('users').insertOne({ name: 'Alice', email: 'alice@example.com' });
```

With Mongoose, I work with models and methods:

```js
// With Mongoose
const user = new User({ name: 'Alice', email: 'alice@example.com' });
await user.save();
```

The Mongoose version gives me automatic validation, type casting, and hooks. I can define that `email` must be unique and `age` must be a number, and Mongoose enforces those rules before anything reaches the database.

## When I Use MongoDB

I reach for MongoDB when:

- My data structure is still evolving and I need flexibility
- I am working with mostly document-like data (blog posts, user profiles)
- I want to prototype quickly without defining strict schemas upfront
- I need horizontal scaling for large amounts of data

When I need strict relationships, complex joins, or ACID transactions across multiple records, I go with PostgreSQL instead. I will cover that later in this section.
