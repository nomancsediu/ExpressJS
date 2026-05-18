# Why Testing Makes Code Reliable

I used to skip testing entirely. I would write some code, manually check it in the browser, and call it done. That worked fine for tiny projects. But once my Express apps grew beyond a few routes, things started breaking in places I did not expect. A change in one file would silently break something else. I spent more time chasing bugs than writing features.

That is when I realized testing is not just a nice-to-have. It is how you build confidence in your code.

## What Testing Actually Gives You

Tests are like a safety net. When you make changes, your tests catch regressions before they reach users. Instead of hoping nothing broke, you get immediate feedback.

Here is what I noticed after adding tests to my Express projects:

- **Fewer bugs in production** because I catch them locally first
- **Faster development** because I spend less time manually re-checking things
- **Better code design** because testable code tends to be well-structured code
- **Confidence to refactor** since tests tell me if I broke something

## A Quick Example

Imagine this simple utility:

```javascript
function sanitizeInput(str) {
  return str.replace(/<[^>]*>/g, '').trim();
}
```

Without a test, I might only check it once in the browser. With a test:

```javascript
test('removes HTML tags and trims whitespace', () => {
  expect(sanitizeInput('  <script>alert("xss")</script>  '))
    .toBe('alert("xss")');
});
```

Now every time I change this function, the test runs automatically. If I accidentally break the sanitization, I know right away.

## The Real Value

Testing is not about perfection. It is about reducing risk. You cannot test every possible scenario, but even basic tests cover the most important paths. When I deploy an Express app with tests, I sleep better knowing the critical routes and logic are verified.

In this section, I will walk through everything I learned about testing Express applications, from basic unit tests all the way to CI/CD pipelines that run tests automatically. Let us get started.
