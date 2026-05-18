# CommonJS vs ESM

One of the more confusing things I encountered when learning Node.js was the two different module systems. Some tutorials used `require`, others used `import`. Some used `module.exports`, others used `export`. Let me clear this up.

## CommonJS: The Original

CommonJS is the module system Node.js was built with. It uses `require()` to import and `module.exports` to export. If you have written any Node.js code, you have used this.

**Exporting:**

```javascript
// math.js
const add = (a, b) => a + b;
const subtract = (a, b) => a - b;

// Export single value
module.exports = add;

// Export multiple values
module.exports = { add, subtract };

// Or add properties one at a time
exports.multiply = (a, b) => a * b;
exports.divide = (a, b) => a / b;
```

**Importing:**

```javascript
// app.js
const add = require('./math');           // Single export
const { add, subtract } = require('./math'); // Named exports
const math = require('./math');          // Import entire module

console.log(math.add(2, 3));    // 5
console.log(add(10, 5));        // 15
console.log(subtract(10, 3));   // 7
```

CommonJS is synchronous. When you call `require`, Node.js reads the file, executes it, and returns the exports. It also caches modules, so requiring the same file twice gives you the same object.

## ESM: ECMAScript Modules

ESM is the official JavaScript module system, standardized in ES6. It uses `import` and `export`. If you have written frontend JavaScript with React or Vue, you have used ESM.

**Exporting:**

```javascript
// math.mjs
export const add = (a, b) => a + b;
export const subtract = (a, b) => a - b;

// Default export
export default function multiply(a, b) {
  return a * b;
}
```

**Importing:**

```javascript
// app.mjs
import multiply, { add, subtract } from './math.mjs';

console.log(add(2, 3));        // 5
console.log(subtract(10, 3));  // 7
console.log(multiply(4, 5));   // 20
```

Notice the `.mjs` extension. By default, Node.js treats `.mjs` files as ESM and `.js` files as CommonJS.

## Enabling ESM in .js Files

You can use ESM with regular `.js` files by adding `"type": "module"` to your `package.json`:

```json
{
  "name": "my-app",
  "version": "1.0.0",
  "type": "module"
}
```

Now all `.js` files in your project use ESM by default. If you want to use CommonJS in a specific file, rename it to `.cjs`.

## Key Differences

Here are the differences that trip people up:

**1. File paths need extensions in ESM:**

```javascript
// CommonJS - extension optional
const math = require('./math');

// ESM - extension required
import math from './math.js';
```

**2. ESM is statically analyzed:**

```javascript
// CommonJS - dynamic, this works
if (condition) {
  const lib = require('./lib');
}

// ESM - imports must be at top level, this does NOT work
if (condition) {
  import lib from './lib.js'; // SyntaxError
}

// But dynamic import works
if (condition) {
  const lib = await import('./lib.js');
}
```

**3. ESM has `this` at the top level as `undefined`:**

```javascript
// CommonJS
console.log(this === module.exports); // true

// ESM
console.log(this); // undefined
```

**4. ESM does not have `__dirname` or `__filename`:**

```javascript
// CommonJS
console.log(__dirname);
console.log(__filename);

// ESM - use import.meta instead
import { fileURLToPath } from 'url';
import { dirname } from 'path';

const __filename = fileURLToPath(import.meta.url);
const __dirname = dirname(__filename);
```

## Which Should You Use?

For new Express projects, I would recommend starting with CommonJS because most Express documentation and tutorials still use it. The Express ecosystem is predominantly CommonJS.

However, ESM is the future. More libraries are moving to it, and it enables features like tree-shaking. If you are starting a brand new project and your dependencies support ESM, go ahead and use it.

The important thing is to pick one and be consistent within a project. Mixing them is possible but adds unnecessary complexity.
