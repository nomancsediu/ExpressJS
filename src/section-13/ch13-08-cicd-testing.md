# CI/CD Testing

Running tests locally is great, but what happens when someone forgets to run them before pushing? CI/CD automates testing so it happens on every push and pull request. This caught more bugs for my team than I care to admit.

## GitHub Actions Test Workflow

I set up a GitHub Actions workflow that runs tests automatically:

```yaml
# .github/workflows/test.yml
name: Tests

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest

    strategy:
      matrix:
        node-version: [18.x, 20.x]

    steps:
      - uses: actions/checkout@v4

      - name: Use Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}

      - name: Install dependencies
        run: npm ci

      - name: Run linter
        run: npm run lint

      - name: Run tests
        run: npm test

      - name: Run coverage
        run: npm run test:coverage
```

This workflow runs on every push and PR. It tests across multiple Node versions, which once caught a bug that only appeared on Node 18.

## Husky for Pre-Commit Hooks

Husky runs scripts before commits or pushes. I use it to prevent broken code from even being committed:

```bash
npm install --save-dev husky
npx husky init
```

Add a pre-commit hook:

```bash
echo "npx lint-staged" > .husky/pre-commit
```

## lint-staged

lint-staged runs linters only on staged files, which is much faster than linting the whole project:

```json
// package.json
{
  "lint-staged": {
    "*.js": ["eslint --fix", "jest --bail --findRelatedTests"],
    "*.json": ["prettier --write"]
  }
}
```

This means every commit automatically gets linted and related tests run. If anything fails, the commit is blocked.

## Adding a Pre-Push Hook

I also add a pre-push hook that runs the full test suite:

```bash
echo "npm test" > .husky/pre-push
```

This way, I cannot push code that fails tests. It has saved me from pushing broken code many times.

## Putting It All Together

My workflow looks like this:

1. **Pre-commit**: lint-staged runs ESLint and related tests on changed files
2. **Pre-push**: full test suite runs locally
3. **On PR**: GitHub Actions runs lint, tests, and coverage checks
4. **On merge to main**: GitHub Actions runs again as a final gate

This layered approach catches problems as early as possible. The pre-commit hook is fast (only changed files). The pre-push hook is more thorough. And CI is the final safety net.

The setup takes maybe 30 minutes, but it pays for itself the first time it catches a bug before it reaches production. I would not build a project without it now.
