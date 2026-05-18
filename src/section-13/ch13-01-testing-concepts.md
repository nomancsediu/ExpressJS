# Testing Concepts

When I first started reading about testing, the terminology overwhelmed me. Unit tests, integration tests, end-to-end tests, TDD, AAA pattern. It felt like a whole new language. Let me break down what each of these means in plain terms.

## Types of Tests

There are three main categories I use in my Express projects:

**Unit tests** test a single function or module in isolation. They are fast and focused.

```javascript
// Testing a utility function in isolation
function add(a, b) {
  return a + b;
}

test('adds two numbers', () => {
  expect(add(2, 3)).toBe(5);
});
```

**Integration tests** check how multiple pieces work together. For example, does your route handler correctly talk to the database?

**End-to-end (E2E) tests** simulate real user behavior from start to finish. They hit your running server, make HTTP requests, and verify the full response.

## The Test Pyramid

I learned about the test pyramid early on, and it really clicked. The idea is simple:

- **Bottom (many)**: Unit tests. Fast, cheap, lots of them.
- **Middle (some)**: Integration tests. Slower, moderate amount.
- **Top (few)**: E2E tests. Slowest, expensive, only the critical paths.

```
      /  E2E  \
     / Integration \
    /   Unit Tests   \
```

Most of your tests should be unit tests. They run in milliseconds and catch the majority of bugs.

## TDD (Test-Driven Development)

TDD means writing the test before writing the code. The cycle is:

1. **Red**: Write a failing test
2. **Green**: Write the minimum code to pass
3. **Refactor**: Clean up while keeping tests green

I do not always follow TDD strictly, but I find it helpful for tricky logic.

## The AAA Pattern

Every test I write follows the Arrange-Act-Assert pattern:

```javascript
test('user validation rejects empty email', () => {
  // Arrange
  const userInput = { name: 'Alice', email: '' };

  // Act
  const result = validateUser(userInput);

  // Assert
  expect(result.valid).toBe(false);
  expect(result.errors).toContain('Email is required');
});
```

- **Arrange**: Set up your test data and conditions
- **Act**: Call the function or run the code you are testing
- **Assert**: Check that the result matches your expectation

This pattern keeps tests readable and consistent. Once I started using AAA, my tests became much easier to understand at a glance.
