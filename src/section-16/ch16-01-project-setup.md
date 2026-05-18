# Project Setup

Let us start from nothing and build up. No templates, no boilerplate generators. Just us and the terminal.

## Creating the Project

First, I create a new folder and initialize it:

```bash
mkdir ecommerce-api
cd ecommerce-api
npm init -y
```

Then I install the core dependencies:

```bash
npm install express mongoose dotenv cors helmet morgan multer joi jsonwebtoken bcryptjs cookie-parser stripe swagger-ui-express yamljs
```

And the dev dependencies:

```bash
npm install -D nodemon
```

## Folder Structure

Here is the structure I am going with:

```
ecommerce-api/
  src/
    app.js
    server.js
    config/
      db.js
    middleware/
      errorHandler.js
      asyncHandler.js
    modules/
      user/
        user.model.js
        user.controller.js
        user.routes.js
        user.validation.js
        auth.middleware.js
      product/
      category/
      cart/
      order/
      review/
    utils/
      ApiError.js
      ApiResponse.js
  uploads/
  .env
  package.json
```

Each module lives in its own folder. This is cleaner than grouping by type (all models together, all routes together). When I need to work on users, everything is right there.

## The Entry Point

I split the Express app and the server into two files. `app.js` sets up the app. `server.js` starts it.

```js
// src/app.js
const express = require('express');
const cors = require('cors');
const helmet = require('helmet');
const morgan = require('morgan');
const cookieParser = require('cookie-parser');
const errorHandler = require('./middleware/errorHandler');

const app = express();

app.use(helmet());
app.use(cors());
app.use(morgan('dev'));
app.use(express.json());
app.use(express.urlencoded({ extended: true }));
app.use(cookieParser());
app.use('/uploads', express.static('uploads'));

// Routes will be added here

app.use(errorHandler);

module.exports = app;
```

```js
// src/server.js
const dotenv = require('dotenv');
dotenv.config();

const app = require('./app');
const { connectDB } = require('./config/db');

const PORT = process.env.PORT || 3000;

connectDB().then(() => {
  app.listen(PORT, () => {
    console.log(`Server running on port ${PORT}`);
  });
});
```

## Error Handling Utility

I create a custom error class and an async handler wrapper. This saves me from writing try-catch in every controller.

```js
// src/utils/ApiError.js
class ApiError extends Error {
  constructor(statusCode, message) {
    super(message);
    this.statusCode = statusCode;
  }
}
module.exports = ApiError;
```

```js
// src/middleware/asyncHandler.js
const ApiError = require('../utils/ApiError');

const asyncHandler = (fn) => (req, res, next) => {
  Promise.resolve(fn(req, res, next)).catch(next);
};

module.exports = asyncHandler;
```

```js
// src/middleware/errorHandler.js
const ApiError = require('../utils/ApiError');

module.exports = (err, req, res, next) => {
  const statusCode = err.statusCode || 500;
  const message = err.message || 'Internal Server Error';

  res.status(statusCode).json({
    success: false,
    message,
    stack: process.env.NODE_ENV === 'development' ? err.stack : undefined,
  });
};
```

This setup gives me a clean foundation. Every response follows the same format. Every error gets caught and handled the same way. Now I can start adding modules.
