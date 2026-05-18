# Designing RESTful URLs

URL design is one of the most important parts of building a REST API. A well-designed URL structure makes your API intuitive and easy to learn. A poorly designed one confuses everyone, including you three months later.

## Use Nouns, Not Verbs

The URL identifies a resource. The HTTP method describes the action:

```
Good:
GET    /users          → list users
POST   /users          → create a user
GET    /users/42       → get user 42
PUT    /users/42       → update user 42
DELETE /users/42       → delete user 42

Bad:
GET    /getUsers
POST   /createUser
GET    /getUser?id=42
POST   /deleteUser
```

The "bad" examples put verbs in the URL and ignore HTTP methods. They work, but they are not RESTful and they waste the semantics of HTTP.

## Plural Nouns for Collections

Use plural nouns for collection resources:

```
/users     → collection of users
/posts     → collection of posts
/comments  → collection of comments
```

Singular nouns feel inconsistent. `/user` could mean one user or the concept of a user. `/users` clearly means the collection.

## Nesting for Relationships

When resources belong to other resources, nest them:

```
/users/42/posts         → posts by user 42
/users/42/posts/7       → post 7 by user 42
/posts/7/comments       → comments on post 7
```

Do not nest too deep, though. Two levels is usually enough:

```
Good:  /users/42/posts
Okay:  /users/42/posts/7/comments
Bad:   /users/42/posts/7/comments/3/replies
```

Deep nesting makes URLs long and hard to parse. For deeper relationships, use query parameters:

```
/replies?commentId=3&postId=7
```

## HTTP Methods Mapping

| Method | Action | Idempotent | Safe |
|---|---|---|---|
| GET | Read | Yes | Yes |
| POST | Create | No | No |
| PUT | Replace entirely | Yes | No |
| PATCH | Partial update | No | No |
| DELETE | Remove | Yes | No |

Idempotent means making the same request multiple times produces the same result. PUT is idempotent because replacing user 42 with the same data twice gives the same user. POST is not because creating a user twice creates two users.

## Actions That Are Not CRUD

Some operations do not map cleanly to CRUD. For these, use a sub-resource with a verb:

```
POST /users/42/activate      → activate user 42
POST /users/42/deactivate    → deactivate user 42
POST /orders/7/cancel        → cancel order 7
POST /emails/3/send          → send email 3
```

Some people prefer `PATCH /users/42` with `{ "active": true }`. Both work. I pick whichever feels clearer for the specific case.

## Consistent Naming

Pick a convention and stick with it:

- Use lowercase: `/users` not `/Users`
- Use hyphens for multi-word: `/blog-posts` not `/blogPosts`
- No trailing slashes: `/users` not `/users/`
- No file extensions: `/users` not `/users.json`

I learned these the hard way by building an inconsistent API and then having to refactor it. Consistency from the start saves pain later.
