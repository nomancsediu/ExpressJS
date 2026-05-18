# Test Coverage

Test coverage tells you what percentage of your code is exercised by tests. It is not a perfect metric, but it gives me a good sense of where my testing gaps are.

## Running Coverage with Jest

Jest has built-in coverage support powered by Istanbul. No extra installation needed:

```bash
npx jest --coverage
```

Or with the npm script:

```bash
npm run test:coverage
```

Jest outputs a table like this:

```
----------|---------|----------|---------|---------|
File      | % Stmts | % Branch | % Funcs | % Lines |
----------|---------|----------|---------|---------|
All files |   78.57 |    66.66 |   85.71 |   78.57 |
 utils.js |   90.00 |    80.00 |  100.00 |   90.00 |
 routes.js|   70.00 |    60.00 |   75.00 |   70.00 |
----------|---------|----------|---------|---------|
```

This tells me my `routes.js` needs more attention.

## Coverage Configuration

I configure coverage thresholds in `jest.config.js` to enforce minimum standards:

```javascript
module.exports = {
  testEnvironment: 'node',
  collectCoverageFrom: [
    'src/**/*.js',
    '!src/index.js',
    '!src/config/**',
  ],
  coverageThreshold: {
    global: {
      branches: 70,
      functions: 80,
      lines: 80,
      statements: 80,
    },
  },
  coverageDirectory: 'coverage',
};
```

If coverage drops below these thresholds, Jest fails the test run. This prevents coverage from slowly eroding over time.

## Reading Coverage Reports

The terminal output is useful, but the HTML report is even better. Jest generates it in the `coverage/lcov-report` directory. Open `coverage/lcov-report/index.html` in a browser and you get a clickable, file-by-file view showing exactly which lines are covered and which are not.

Red lines mean uncovered code. I use this to find edge cases I forgot to test.

## What Istanbul Does

Istanbul (now called nyc) is the tool Jest uses under the hood. It instruments your code by adding tracking counters to every statement, branch, function, and line. When tests run, these counters record what was executed.

The four metrics it tracks:

- **Statements**: Did each statement run?
- **Branches**: Did each if/else path execute?
- **Functions**: Was each function called?
- **Lines**: Did each line of code run?

## A Healthy Perspective on Coverage

I used to chase 100% coverage. Now I know that is often impractical and sometimes counterproductive. High coverage does not mean high quality. A test that calls a function without checking the result still counts as covered.

I aim for 80% coverage on critical paths and do not stress about edge utility functions. The goal is confidence, not a number.
