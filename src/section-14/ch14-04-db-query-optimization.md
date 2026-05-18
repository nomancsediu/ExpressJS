# Database Query Optimization

Database queries are the most common bottleneck I find in Express apps. A single unoptimized query can make an endpoint 10x slower. Here is what I learned about making queries fast.

## Indexing

Indexes are the single biggest performance improvement you can make. Without an index, the database scans every document. With an index, it jumps straight to the right data.

```javascript
// Mongoose schema with index
const userSchema = new mongoose.Schema({
  email: { type: String, unique: true },  // unique creates an index
  name: String,
  createdAt: { type: Date, default: Date.now },
});

// Compound index for queries that filter on multiple fields
userSchema.index({ name: 1, createdAt: -1 });
```

Check which indexes your queries use:

```javascript
// In MongoDB shell
db.users.find({ name: 'Alice' }).explain('executionStats');
```

Look for `totalDocsExamined`. If it equals your total document count, you need an index.

## Projection

Only fetch the fields you need. I used to do `User.find()` and return everything. That wastes memory and bandwidth:

```javascript
// Bad: fetches all fields
const users = await User.find({ active: true });

// Better: only select what you need
const users = await User.find({ active: true }).select('name email');
```

## Using lean()

Mongoose documents carry a lot of overhead for change tracking and methods. If you only need the data, use `.lean()`:

```javascript
// Returns full Mongoose documents (slower, more memory)
const users = await User.find();

// Returns plain JavaScript objects (faster, less memory)
const users = await User.find().lean();
```

`.lean()` can make queries 2-3x faster. I use it for any read-only operation.

## Pagination

Never fetch all records at once. Always paginate:

```javascript
app.get('/api/posts', async (req, res) => {
  const page = parseInt(req.query.page) || 1;
  const limit = Math.min(parseInt(req.query.limit) || 20, 100);

  const [posts, total] = await Promise.all([
    Post.find()
      .skip((page - 1) * limit)
      .limit(limit)
      .lean(),
    Post.countDocuments(),
  ]);

  res.json({
    posts,
    page,
    totalPages: Math.ceil(total / limit),
    total,
  });
});
```

I cap `limit` at 100 to prevent someone from requesting millions of records.

## The N+1 Problem

This is a classic trap. You fetch a list of items, then make a separate query for each one:

```javascript
// N+1: 1 query for posts + N queries for authors
const posts = await Post.find();
for (const post of posts) {
  post.author = await User.findById(post.authorId);
}
```

Fix it with populate or batch queries:

```javascript
// Single query with populate
const posts = await Post.find().populate('author', 'name email');
```

Or manually batch:

```javascript
const posts = await Post.find().lean();
const authorIds = [...new Set(posts.map(p => p.authorId))];
const authors = await User.find({ _id: { $in: authorIds } }).lean();
const authorMap = new Map(authors.map(a => [a._id.toString(), a]));
const result = posts.map(p => ({ ...p, author: authorMap.get(p.authorId.toString()) }));
```

Two queries instead of N+1. The difference is dramatic with large datasets.
