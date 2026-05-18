# Order Module

Orders turn a shopping cart into a real purchase. This module connects the cart, the user, and (later) the payment system. Let me build it out.

## Order Model

An order stores the purchased items, shipping address, payment info, and status. I snapshot the product details because prices and stock can change after the order is placed.

```js
// src/modules/order/order.model.js
const mongoose = require('mongoose');

const orderItemSchema = new mongoose.Schema({
  product: { type: mongoose.Schema.Types.ObjectId, ref: 'Product' },
  name: { type: String, required: true },
  price: { type: Number, required: true },
  quantity: { type: Number, required: true },
});

const orderSchema = new mongoose.Schema({
  user: { type: mongoose.Schema.Types.ObjectId, ref: 'User', required: true },
  items: [orderItemSchema],
  totalAmount: { type: Number, required: true },
  shippingAddress: {
    street: { type: String, required: true },
    city: { type: String, required: true },
    state: { type: String },
    zipCode: { type: String, required: true },
    country: { type: String, required: true },
  },
  paymentMethod: { type: String, enum: ['card', 'cash'], default: 'card' },
  paymentStatus: { type: String, enum: ['pending', 'paid', 'failed', 'refunded'], default: 'pending' },
  orderStatus: {
    type: String,
    enum: ['processing', 'confirmed', 'shipped', 'delivered', 'cancelled'],
    default: 'processing',
  },
  paidAt: { type: Date },
  deliveredAt: { type: Date },
}, { timestamps: true });

module.exports = mongoose.model('Order', orderSchema);
```

The status flow goes from `processing` to `confirmed` to `shipped` to `delivered`. It can also be `cancelled` at any point before shipping.

## Creating an Order from Cart

When the user checks out, I take their cart and turn it into an order.

```js
// src/modules/order/order.controller.js
const Order = require('./order.model');
const Cart = require('../cart/cart.model');
const Product = require('../product/product.model');
const ApiError = require('../../utils/ApiError');
const asyncHandler = require('../../middleware/asyncHandler');

exports.createOrder = asyncHandler(async (req, res) => {
  const { shippingAddress, paymentMethod } = req.body;

  const cart = await Cart.findOne({ user: req.user._id }).populate('items.product');
  if (!cart || cart.items.length === 0) {
    throw new ApiError(400, 'Cart is empty');
  }

  // Check stock and snapshot item details
  const orderItems = [];
  for (const item of cart.items) {
    const product = item.product;
    if (product.stock < item.quantity) {
      throw new ApiError(400, `${product.name} is out of stock`);
    }
    orderItems.push({
      product: product._id,
      name: product.name,
      price: item.price,
      quantity: item.quantity,
    });
    product.stock -= item.quantity;
    await product.save();
  }

  const order = await Order.create({
    user: req.user._id,
    items: orderItems,
    totalAmount: cart.totalAmount,
    shippingAddress,
    paymentMethod,
  });

  // Clear the cart
  cart.items = [];
  cart.totalAmount = 0;
  await cart.save();

  res.status(201).json({ success: true, data: order });
});
```

I check stock for every item, decrement it, snapshot the product details into the order, and then clear the cart. This is all in one request so nothing gets out of sync.

## Order Status Updates

Admins can update the order status:

```js
exports.updateOrderStatus = asyncHandler(async (req, res) => {
  const { orderStatus } = req.body;
  const order = await Order.findById(req.params.id);
  if (!order) throw new ApiError(404, 'Order not found');

  order.orderStatus = orderStatus;
  if (orderStatus === 'delivered') order.deliveredAt = new Date();

  await order.save();
  res.json({ success: true, data: order });
});
```

## Order History

Users can see their own orders. Admins can see all orders.

```js
exports.getMyOrders = asyncHandler(async (req, res) => {
  const orders = await Order.find({ user: req.user._id }).sort({ createdAt: -1 });
  res.json({ success: true, data: orders });
});

exports.getAllOrders = asyncHandler(async (req, res) => {
  const { page = 1, limit = 20, status } = req.query;
  const filter = {};
  if (status) filter.orderStatus = status;

  const orders = await Order.find(filter)
    .sort({ createdAt: -1 })
    .skip((page - 1) * limit)
    .limit(Number(limit))
    .populate('user', 'name email');

  res.json({ success: true, data: orders });
});
```

## Order Routes

```js
const router = require('express').Router();
const { protect, restrictTo } = require('../user/auth.middleware');

router.post('/', protect, createOrder);
router.get('/my-orders', protect, getMyOrders);
router.get('/', protect, restrictTo('admin'), getAllOrders);
router.patch('/:id/status', protect, restrictTo('admin'), updateOrderStatus);
```

The order module ties everything together. Cart becomes order, order gets a status, and the status tracks the delivery journey.
