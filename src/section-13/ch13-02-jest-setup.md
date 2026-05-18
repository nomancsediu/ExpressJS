# Setting Up Jest

Jest is the testing framework I use for all my Express projects. It is popular, well-documented, and works great out of the box. Let me walk through how I set it up.

## Installing Jest

```bash
npm install --save-dev jest
```

For TypeScript projects, I also add the types:

```bash
npm install --save-dev @types/jest ts-jest
```

## Configuration

I create a `jest.config.js` file at the project root:

```javascript
module.exports = {
  testEnvironment: 'node',
  roots: ['<rootDir>/tests'],
  testMatch: ['**/*.test.js'],
  collectCoverageFrom: [
    'src/**/*.js',
    '!src/index.js',
  ],
  coverageDirectory: 'coverage',
};
```

For TypeScript, I swap to `ts-jest`:

```javascript
module.exports = {
  preset: 'ts-jest',
  testEnvironment: 'node',
  roots: ['<rootDir>/tests'],
  testMatch: ['**/*.test.ts'],
};
```

## NPM Scripts

I add these to my `package.json`:

```json
{
  "scripts": {
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage"
  }
}
```

The `--watch` flag is my favorite. It reruns tests automatically when files change, which is super helpful during development.

## Basic Test Structure

Jest uses `describe` and `it` (or `test`) to organize tests:

```javascript
describe('Math utilities', () => {
  it('adds two numbers correctly', () => {
    expect(2 + 3).toBe(5);
  });

  it('handles negative numbers', () => {
    expect(-1 + 1).toBe(0);
  });
});
```

`describe` groups related tests. `it` defines individual test cases.

## Common Matchers

Jest provides many matchers. These are the ones I use most:

```javascript
// Equality
expect(result).toBe(42);           // strict equality
expect(data).toEqual({ a: 1 });   // deep equality

// Truthiness
expect(value).toBeTruthy();
expect(value).toBeFalsy();
expect(value).toBeNull();
expect(value).toBeUndefined();

// Numbers
expect(n).toBeGreaterThan(5);
expect(n).toBeCloseTo(0.3);

// Strings and arrays
expect(name).toContain('Alice');
expect(list).toHaveLength(3);

// Errors
expect(() => parse('')).toThrow();
```

Once you know these basics, you can write tests for pretty much anything. The key is to start simple and add more matchers as you need them.
