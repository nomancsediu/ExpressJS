# API Documentation with Swagger

An undocumented API is a useless API. No matter how well-designed your endpoints are, if people cannot figure out how to use them, they will not. Swagger (now called OpenAPI) lets you generate interactive documentation that lives alongside your code.

## Setting Up swagger-jsdoc and swagger-ui-express

```bash
npm install swagger-jsdoc swagger-ui-express
```

```js
const swaggerJsdoc = require('swagger-jsdoc');
const swaggerUi = require('swagger-ui-express');

const options = {
  definition: {
    openapi: '3.0.0',
    info: {
      title: 'My API',
      version: '1.0.0',
      description: 'A simple Express API',
    },
    servers: [
      { url: 'http://localhost:3000', description: 'Development' },
    ],
  },
  apis: ['./routes/*.js'], // files with JSDoc comments
};

const specs = swaggerJsdoc(options);

app.use('/api-docs', swaggerUi.serve, swaggerUi.setup(specs));
```

Now visit `/api-docs` and you get a full interactive documentation page.

## Documenting Endpoints with JSDoc

Write documentation as comments above your route handlers:

```js
/**
 * @openapi
 * /users:
 *   get:
 *     summary: Get all users
 *     description: Returns a list of all users
 *     tags:
 *       - Users
 *     parameters:
 *       - in: query
 *         name: page
 *         schema:
 *           type: integer
 *           default: 1
 *         description: Page number
 *       - in: query
 *         name: limit
 *         schema:
 *           type: integer
 *           default: 10
 *         description: Items per page
 *     responses:
 *       200:
 *         description: A list of users
 *         content:
 *           application/json:
 *             schema:
 *               type: object
 *               properties:
 *                 data:
 *                   type: array
 *                   items:
 *                     $ref: '#/components/schemas/User'
 */
app.get('/users', (req, res) => {
  res.json({ data: users });
});
```

## Defining Schemas

Reusable schemas go in the components section:

```js
/**
 * @openapi
 * components:
 *   schemas:
 *     User:
 *       type: object
 *       properties:
 *         id:
 *           type: integer
 *           example: 1
 *         name:
 *           type: string
 *           example: Alice
 *         email:
 *           type: string
 *           example: alice@test.com
 *     Error:
 *       type: object
 *       properties:
 *         error:
 *           type: string
 *           example: Not found
 */
```

## Documenting POST Endpoints

```js
/**
 * @openapi
 * /users:
 *   post:
 *     summary: Create a new user
 *     tags:
 *       - Users
 *     requestBody:
 *       required: true
 *       content:
 *         application/json:
 *           schema:
 *             type: object
 *             required:
 *               - name
 *               - email
 *             properties:
 *               name:
 *                 type: string
 *               email:
 *                 type: string
 *     responses:
 *       201:
 *         description: User created
 *       400:
 *         description: Missing required fields
 */
app.post('/users', (req, res) => {
  // handler
});
```

## Tags for Organization

Tags group related endpoints in the UI:

```js
/**
 * @openapi
 * tags:
 *   - name: Users
 *     description: User management
 *   - name: Posts
 *     description: Post management
 */
```

Swagger UI gives you a try-it-out feature where users can send real requests right from the docs. This is incredibly helpful for onboarding new developers. I now add Swagger on day one of every API project.
