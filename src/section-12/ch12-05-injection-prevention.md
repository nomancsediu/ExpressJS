# Injection Prevention

Injection attacks happen when untrusted data gets interpreted as code. SQL injection, NoSQL injection, and command injection are all variations of the same problem: mixing data with commands.

## SQL Injection

This is the classic injection attack. Consider this dangerous code:

```js
// DANGEROUS: never do this
app.get('/users', (req, res) => {
  const query = `SELECT * FROM users WHERE name = '${req.query.name}'`;
  db.query(query, (err, results) => {
    res.json(results);
  });
});
```

If someone sends `?name=' OR '1'='1`, the query becomes:

```sql
SELECT * FROM users WHERE name = '' OR '1'='1'
```

This returns every user in the database. An attacker could also delete tables or extract sensitive data.

## Parameterized Queries

The solution is to separate data from the query:

```js
// Safe: parameterized query
app.get('/users', (req, res) => {
  const query = 'SELECT * FROM users WHERE name = ?';
  db.query(query, [req.query.name], (err, results) => {
    res.json(results);
  });
});
```

With Prisma, this is handled automatically:

```js
// Prisma uses parameterized queries under the hood
const users = await db.user.findMany({
  where: { name: req.query.name }
});
```

## NoSQL Injection

MongoDB has its own injection risks. Consider this:

```js
// DANGEROUS: MongoDB injection
app.post('/login', (req, res) => {
  const { username, password } = req.body;
  db.users.findOne({ username, password }, (err, user) => {
    if (user) res.send('Logged in');
  });
});
```

If an attacker sends `{"username": "admin", "password": {"$gt": ""}}`, the query matches any user where the password is greater than an empty string. That is probably every user.

Prevention:

```js
// Validate input types
const { username, password } = req.body;

if (typeof username !== 'string' || typeof password !== 'string') {
  return res.status(400).send('Invalid input');
}

// Use proper query methods
const user = await db.users.findOne({
  username: String(username),
  password: String(password)
});
```

## Command Injection

When you run shell commands with user input:

```js
// DANGEROUS: command injection
app.get('/ping', (req, res) => {
  exec(`ping -c 1 ${req.query.host}`, (err, stdout) => {
    res.send(stdout);
  });
});
```

An attacker sends `?host=google.com; rm -rf /` and your server deletes everything.

Prevention:

```js
const { execFile } = require('child_process');

// Safe: execFile does not use a shell
execFile('ping', ['-c', '1', req.query.host], (err, stdout) => {
  res.send(stdout);
});

// Or validate the input
const host = req.query.host;
if (!/^[a-zA-Z0-9.-]+$/.test(host)) {
  return res.status(400).send('Invalid hostname');
}
```

## General Rules

1. **Never concatenate user input into queries or commands**
2. **Use parameterized queries for databases**
3. **Validate input types and formats**
4. **Use the least powerful tool** (`execFile` instead of `exec`)
5. **Apply the principle of least privilege** (database users should not be able to drop tables)

I once found a SQL injection in a production app during a code review. The developer had copy-pasted a Stack Overflow answer that used string concatenation. Always review database queries carefully. Parameterized queries are not optional.
