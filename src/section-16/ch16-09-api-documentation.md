# API Documentation with Swagger

An API without documentation is like a city without street signs. Nobody knows where to go. Swagger (OpenAPI) generates interactive docs that let developers explore and test your endpoints right in the browser.

## Setting Up Swagger

I use `swagger-ui-express` and `swagger-jsdoc` to generate docs from JSDoc comments in the code.

```bash
npm install swagger-ui-express swagger-jsdoc
```

Then I create a config file:

```js
// src/config/swagger.js
const swaggerJsdoc = require('swagger-jsdoc');

const options = {
  definition: {
    openapi: '3.0.0',
    info: {
      title: 'E-Commerce API',
      version: '1.0.0',
      description: 'A full-featured e-commerce API built with Express.js',
    },
    servers: [
      { url: 'http://localhost:3000/api', description: 'Development' },
    ],
    components: {
      securitySchemes: {
        bearerAuth: {
          type: 'http',
          scheme: 'bearer',
          bearerFormat: 'JWT',
        },
      },
      schemas: {
        User: {
          type: 'object',
          properties: {
            id: { type: 'string' },
            name: { type: 'string' },
            email: { type: 'string', format: 'email' },
            role: { type: 'string', enum: ['user', 'admin'] },
          },
        },
        Product: {
          type: 'object',
          properties: {
            id: { type: 'string' },
            name: { type: 'string' },
            description: { type: 'string' },
            price: { type: 'number' },
            stock: { type: 'number' },
            category: { type: 'string' },
            brand: { type: 'string' },
          },
        },
        Error: {
          type: 'object',
          properties: {
            success: { type: 'boolean', example: false },
            message: { type: 'string' },
          },
        },
      },
    },
  },
  apis: ['./src/modules/**/*.js'],
};

module.exports = swaggerJsdoc(options);
```

Then I add the Swagger UI route to `app.js`:

```js
const swaggerUi = require('swagger-ui-express');
const swaggerSpec = require('./config/swagger');

app.use('/api/docs', swaggerUi.serve, swaggerUi.setup(swaggerSpec));
```

Now I can visit `/api/docs` and see the interactive documentation.

## Documenting Endpoints

I write Swagger docs as JSDoc comments above each controller function. Here is an example for the product module:

```js
/**
 * @swagger
 * /products:
 *   get:
 *     summary: Get all products
 *     tags: [Products]
 *     parameters:
 *       - in: query
 *         name: page
 *         schema: { type: integer, default: 1 }
 *       - in: query
 *         name: limit
 *         schema: { type: integer, default: 10 }
 *       - in: query
 *         name: category
 *         schema: { type: string }
 *         description: Filter by category ID
 *       - in: query
 *         name: search
 *         schema: { type: string }
 *         description: Text search in name and description
 *       - in: query
 *         name: minPrice
 *         schema: { type: number }
 *       - in: query
 *         name: maxPrice
 *         schema: { type: number }
 *       - in: query
 *         name: sort
 *         schema: { type: string, enum: [price_asc, price_desc, newest] }
 *     responses:
 *       200:
 *         description: List of products with pagination
 *         content:
 *           application/json:
 *             schema:
 *               type: object
 *               properties:
 *                 success: { type: boolean }
 *                 data: { type: array, items: { $ref: '#/components/schemas/Product' } }
 *                 pagination:
 *                   type: object
 *                   properties:
 *                     page: { type: integer }
 *                     limit: { type: integer }
 *                     total: { type: integer }
 *                     pages: { type: integer }
 */
exports.getProducts = asyncHandler(async (req, res) => {
  // ... controller code
});
```

And a POST endpoint with auth:

```js
/**
 * @swagger
 * /products:
 *   post:
 *     summary: Create a product
 *     tags: [Products]
 *     security:
 *       - bearerAuth: []
 *     requestBody:
 *       required: true
 *       content:
 *         application/json:
 *           schema:
 *             type: object
 *             required: [name, description, price, stock, category]
 *             properties:
 *               name: { type: string }
 *               description: { type: string }
 *               price: { type: number }
 *               stock: { type: number }
 *               category: { type: string }
 *               brand: { type: string }
 *     responses:
 *       201:
 *         description: Product created
 *       401:
 *         description: Not authenticated
 *       403:
 *         description: Not authorized (admin only)
 */
```

## Why This Matters

Writing docs feels tedious at first. But when someone (or future you) needs to integrate with this API, those Swagger docs save hours of guesswork. The interactive UI lets them try endpoints without writing a single curl command. Document as you build, not after.
