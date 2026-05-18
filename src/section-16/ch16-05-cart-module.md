# Cart Module

The shopping cart is where users collect items before buying. It needs to handle adding products, removing them, updating quantities, and calculating totals. Let me build it step by step.

## Cart Model

Each user has one cart. The cart holds an array of items with product references and quantities.

```js
// src/modules/cart/cart.model.js
const mongoose = require('mongoose');

const cartItemSchema = new mongoose.Schema({
  product: { type: mongoose.Schema.Types.ObjectId, ref: 'Product', required: true },
  quantity: { type: Number, required: true, min: 1, default: 1 },
  price: { type: Number, required: true, min: 0 },
});

const cartSchema = new mongoose.Schema({
  user: { type: mongoose.Schema.Types.ObjectId, ref: 'User', required: true, unique: true },
  items: [cartItemSchema],
  totalAmount: { type: Number, default: 0 },
}, { timestamps: true });

cartSchema.methods.calculateTotal = function () {
  this.totalAmount = this.items.reduce((sum, item) => sum + item.price * item.quantity, 0);
  return this.totalAmount;
};

module.exports = mongoose.model('Cart', cartSchema);
```

I store the price at the time the item is added. This is important because product prices can change, but the cart should show the price the user saw when they added it.

## Cart Controller

### Adding an Item

When a user adds a product, I check if it already exists in the cart. If it does, I increase the quantity. If not, I add a new item.

```js
// src/modules/cart/cart.controller.js
const Cart = require('./cart.model');
const Product = require('../product/product.model');
const ApiError = require('../../utils/ApiError');
const asyncHandler = require('../../middleware/asyncHandler');

exports.addToCart = asyncHandler(async (req, res) => {
  const { productId, quantity = 1 } = req.body;

  const product = await Product.findById(productId);
  if (!product) throw new ApiError(404, 'Product not found');
  if (product.stock < quantity) throw new ApiError(400, 'Not enough stock');

  let cart = await Cart.findOne({ user: req.user._id });

  if (!cart) {
    cart = await Cart.create({ user: req.user._id, items: [] });
  }

  const existingItem = cart.items.find(
    item => item.product.toString() === productId
  );

  if (existingItem) {
    existingItem.quantity += quantity;
  } else {
    cart.items.push({ product: productId, quantity, price: product.price });
  }

  cart.calculateTotal();
  await cart.save();

  await cart.populate('items.product', 'name price images');

  res.json({ success: true, data: cart });
});
```

### Updating Quantity

The user might want more or fewer of a specific item.

```js
exports.updateCartItem = asyncHandler(async (req, res) => {
  const { quantity } = req.body;
  const cart = await Cart.findOne({ user: req.user._id });
  if (!cart) throw new ApiError(404, 'Cart not found');

  const item = cart.items.id(req.params.itemId);
  if (!item) throw new ApiError(404, 'Item not in cart');

  const product = await Product.findById(item.product);
  if (product.stock < quantity) throw new ApiError(400, 'Not enough stock');

  item.quantity = quantity;
  cart.calculateTotal();
  await cart.save();

  await cart.populate('items.product', 'name price images');
  res.json({ success: true, data: cart });
});
```

### Removing an Item

```js
exports.removeFromCart = asyncHandler(async (req, res) => {
  const cart = await Cart.findOne({ user: req.user._id });
  if (!cart) throw new ApiError(404, 'Cart not found');

  cart.items = cart.items.filter(item => item._id.toString() !== req.params.itemId);

  cart.calculateTotal();
  await cart.save();

  await cart.populate('items.product', 'name price images');
  res.json({ success: true, data: cart });
});
```

### Getting the Cart

```js
exports.getCart = asyncHandler(async (req, res) => {
  let cart = await Cart.findOne({ user: req.user._id }).populate('items.product', 'name price images');
  if (!cart) {
    cart = await Cart.create({ user: req.user._id, items: [] });
  }
  res.json({ success: true, data: cart });
});
```

## Cart Routes

```js
const router = require('express').Router();
const { protect } = require('../user/auth.middleware');

router.use(protect);

router.get('/', getCart);
router.post('/add', addToCart);
router.patch('/item/:itemId', updateCartItem);
router.delete('/item/:itemId', removeFromCart);
```

I protect all cart routes with the auth middleware. Only logged-in users can have carts. The `calculateTotal` method runs after every change, so the total is always up to date.
