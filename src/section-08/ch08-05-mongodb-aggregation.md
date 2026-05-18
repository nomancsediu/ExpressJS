# MongoDB Aggregation

Simple queries are great, but sometimes I need to ask more complex questions. How many users signed up per month? What is the average order value? Which products are top sellers? For these, I use MongoDB's aggregation framework.

## What Is Aggregation?

Aggregation processes documents through a pipeline. Each stage transforms the data, and the result of one stage feeds into the next. Think of it like an assembly line where each station does something different to the data.

```js
const result = await User.aggregate([
  stage1,
  stage2,
  stage3,
  // ... each stage processes what the previous one produced
]);
```

## $match

`$match` filters documents, like a WHERE clause in SQL:

```js
// Find all active users over 25
const result = await User.aggregate([
  { $match: { isActive: true, age: { $gt: 25 } } }
]);
```

I usually put `$match` early in the pipeline. It reduces the number of documents flowing through later stages, which makes the aggregation faster.

## $group

`$group` groups documents by a field and calculates aggregates:

```js
// Count users by role
const result = await User.aggregate([
  {
    $group: {
      _id: '$role',
      count: { $sum: 1 },
      averageAge: { $avg: '$age' },
    }
  }
]);
// Result: [{ _id: 'admin', count: 3, averageAge: 32 }, { _id: 'user', count: 50, averageAge: 27 }]
```

Common accumulators: `$sum`, `$avg`, `$min`, `$max`, `$first`, `$last`, `$push`.

## $sort

`$sort` orders the results, just like ORDER BY:

```js
// Sort by count descending
const result = await User.aggregate([
  { $group: { _id: '$role', count: { $sum: 1 } } },
  { $sort: { count: -1 } }  // -1 = descending, 1 = ascending
]);
```

## $project

`$project` reshapes documents, including or excluding fields and creating computed fields:

```js
const result = await User.aggregate([
  {
    $project: {
      _id: 0,           // Exclude _id
      name: 1,           // Include name
      email: 1,          // Include email
      birthYear: { $subtract: [2024, '$age'] },  // Computed field
    }
  }
]);
```

## $lookup

`$lookup` performs a left outer join with another collection:

```js
// Get posts with their author info
const result = await Post.aggregate([
  {
    $lookup: {
      from: 'users',          // The collection to join
      localField: 'authorId', // Field in the posts collection
      foreignField: '_id',    // Field in the users collection
      as: 'author',           // Name for the joined data
    }
  },
  { $unwind: '$author' },    // Flatten the array (lookup returns an array)
  {
    $project: {
      title: 1,
      content: 1,
      'author.name': 1,
      'author.email': 1,
    }
  }
]);
```

## A Real-World Example

Monthly signups report for the last year:

```js
const monthlySignups = await User.aggregate([
  {
    $match: {
      createdAt: { $gte: new Date('2024-01-01') }
    }
  },
  {
    $group: {
      _id: {
        year: { $year: '$createdAt' },
        month: { $month: '$createdAt' },
      },
      count: { $sum: 1 },
    }
  },
  { $sort: { '_id.year': 1, '_id.month': 1 } },
  {
    $project: {
      _id: 0,
      year: '$_id.year',
      month: '$_id.month',
      count: 1,
    }
  }
]);
```

Aggregation is powerful but can be tricky. I build pipelines one stage at a time, checking the output at each step, rather than writing the whole thing at once.
