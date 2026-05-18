# Error Types

Before I can handle errors, I need to understand what kinds exist. Not all errors are the same, and knowing the difference helps me decide how to respond.

## Syntax Errors

These happen when I write code that JavaScript cannot parse. A missing bracket, a typo in a keyword, a stray comma.

```js
// Missing closing parenthesis
app.get('/users', (req, res => {
  res.json({ users: [] });
});
```

Syntax errors get caught immediately. Node.js will not even start your app. You see the error in the terminal and fix it. These are the easiest to deal with.

## Runtime Errors

The code is valid JavaScript, but it fails when it runs. Maybe you try to read a property on undefined, or call a function that does not exist.

```js
app.get('/user', (req, res) => {
  const name = req.body.user.name; // TypeError: Cannot read property 'name' of undefined
  res.json({ name });
});
```

These are sneaky because the code looks fine until that specific code path runs.

## Logical Errors

The hardest to find. The code runs without crashing, but it produces the wrong result. Maybe you sort ascending when you meant descending, or you calculate a discount the wrong way.

```js
// Off by one? Wrong formula? Wrong condition?
const discount = price * 0.1; // Should this be 10% or 1%?
```

No error gets thrown. No stack trace. Just wrong data quietly flowing through your app.

## Operational vs Programming Errors

This is the most useful distinction I have learned.

**Operational errors** are expected problems. A user sends invalid input. A database connection fails. An external API returns a 500. These are not bugs. They are situations you should anticipate and handle.

```js
// Operational: the user sent bad data
if (!email.includes('@')) {
  throw new Error('Invalid email format');
}
```

**Programming errors** are bugs in your code. Reading a property on undefined. Passing the wrong number of arguments. Using a variable that was never declared. These should be fixed, not handled at runtime.

The key takeaway: operational errors should be caught and handled gracefully. Programming errors should be fixed in your code. Your error handling strategy should account for both, but the response is different for each.
