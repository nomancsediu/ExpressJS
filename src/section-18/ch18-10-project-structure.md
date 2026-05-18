# Project Structure

How you organize your files matters more than you think. A bad structure makes code hard to find and harder to change. A good structure scales as the project grows. Let me compare the common approaches and pick what works.

## Layer-Based Structure

Group files by type. All models together, all controllers together, all routes together.

```
src/
  controllers/
    userController.js
    productController.js
    orderController.js
  models/
    User.js
    Product.js
    Order.js
  routes/
    userRoutes.js
    productRoutes.js
    orderRoutes.js
  middleware/
    auth.js
    errorHandler.js
```

This is how most tutorials organize code. It works for small projects. But as the app grows, I end up jumping between folders to work on a single feature. Adding a "reviews" feature means touching four different folders.

## Feature-Based Structure

Group files by feature. Everything related to users is in one folder.

```
src/
  modules/
    user/
      user.model.js
      user.controller.js
      user.routes.js
      user.validation.js
      auth.middleware.js
    product/
      product.model.js
      product.controller.js
      product.routes.js
      product.validation.js
    order/
      order.model.js
      order.controller.js
      order.routes.js
    review/
      review.model.js
      review.controller.js
      review.routes.js
  middleware/
    errorHandler.js
    asyncHandler.js
  config/
    db.js
    stripe.js
  utils/
    ApiError.js
    ApiResponse.js
```

When I need to work on the product feature, everything is in one folder. I can also easily move or remove a feature without hunting through the codebase.

## Barrel Exports

Barrel exports use `index.js` files to re-export from a module. This keeps import paths clean:

```js
// src/modules/user/index.js
const userRoutes = require('./user.routes');
module.exports = userRoutes;
```

```js
// src/app.js
const userRoutes = require('./modules/user');
const productRoutes = require('./modules/product');

app.use('/api/users', userRoutes);
app.use('/api/products', productRoutes);
```

Without barrel exports, imports look like this:

```js
const userRoutes = require('./modules/user/user.routes');
```

Barrel exports hide the internal file structure. I can rename or move files inside a module without breaking imports in other modules.

## Shared Code

Some code does not belong to any feature. I put shared code in dedicated folders:

```
src/
  middleware/       # Shared middleware (error handler, async handler)
  config/           # Configuration (db, stripe, swagger)
  utils/            # Utilities (ApiError, ApiResponse)
  events/           # Event bus
  queues/           # Job queues
  types/            # TypeScript types
```

## Scaling the Structure

As the project grows, I add more organization:

```
src/
  modules/
    user/
      user.model.js
      user.controller.js
      user.service.js      # Business logic layer
      user.repository.js   # Database queries
      user.routes.js
      user.validation.js
      auth.middleware.js
      __tests__/           # Tests next to the code
```

Adding a service layer between the controller and the model keeps controllers thin. The controller handles HTTP. The service handles business logic. The repository handles database queries.

## My Recommendation

Start with feature-based structure. It is the best balance of simplicity and scalability. Add layers (services, repositories) only when the controller gets too big. Use barrel exports from the beginning to keep imports clean. And remember: the best structure is one your team can follow consistently.
