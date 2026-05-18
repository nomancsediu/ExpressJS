# Payment with Stripe

Payments used to intimidate me. Credit cards, security, PCI compliance. But Stripe makes it surprisingly manageable. They handle the sensitive stuff on their servers. We just need to integrate their API.

## Setup

Install the Stripe SDK and set the API keys in `.env`:

```bash
STRIPE_SECRET_KEY=sk_test_xxx
STRIPE_WEBHOOK_SECRET=whsec_xxx
```

Then initialize Stripe in the app:

```js
// src/config/stripe.js
const stripe = require('stripe')(process.env.STRIPE_SECRET_KEY);
module.exports = stripe;
```

## Creating a Checkout Session

When the user wants to pay, I create a Stripe Checkout Session. This redirects them to a Stripe-hosted payment page.

```js
// src/modules/order/payment.controller.js
const stripe = require('../../config/stripe');
const Order = require('../order/order.model');
const Cart = require('../../modules/cart/cart.model');
const ApiError = require('../../utils/ApiError');
const asyncHandler = require('../../middleware/asyncHandler');

exports.createCheckoutSession = asyncHandler(async (req, res) => {
  const cart = await Cart.findOne({ user: req.user._id }).populate('items.product');
  if (!cart || cart.items.length === 0) {
    throw new ApiError(400, 'Cart is empty');
  }

  const lineItems = cart.items.map(item => ({
    price_data: {
      currency: 'usd',
      product_data: { name: item.product.name },
      unit_amount: Math.round(item.price * 100), // Stripe uses cents
    },
    quantity: item.quantity,
  }));

  const session = await stripe.checkout.sessions.create({
    payment_method_types: ['card'],
    line_items: lineItems,
    mode: 'payment',
    success_url: `${process.env.CLIENT_URL}/order/success?session_id={CHECKOUT_SESSION_ID}`,
    cancel_url: `${process.env.CLIENT_URL}/cart`,
    metadata: { userId: req.user._id.toString() },
  });

  res.json({ success: true, data: { sessionId: session.id, url: session.url } });
});
```

Stripe works in cents, so I multiply by 100. The `metadata` field lets me pass the user ID so I can connect the payment back to the order later.

## Webhook Handler

Stripe sends webhook events when payment status changes. This is more reliable than checking in the success redirect.

```js
exports.stripeWebhook = async (req, res) => {
  const sig = req.headers['stripe-signature'];

  let event;
  try {
    event = stripe.webhooks.constructEvent(req.body, sig, process.env.STRIPE_WEBHOOK_SECRET);
  } catch (err) {
    return res.status(400).send(`Webhook Error: ${err.message}`);
  }

  if (event.type === 'checkout.session.completed') {
    const session = event.data.object;
    const userId = session.metadata.userId;

    const order = await Order.findOne({ user: userId, paymentStatus: 'pending' }).sort({ createdAt: -1 });
    if (order) {
      order.paymentStatus = 'paid';
      order.orderStatus = 'confirmed';
      order.paidAt = new Date();
      await order.save();
    }
  }

  res.json({ received: true });
};
```

Important: the webhook route needs raw body parsing, not JSON. I configure Express to skip body parsing for that route:

```js
// In app.js, before other middleware
app.post('/api/payment/webhook', express.raw({ type: 'application/json' }), stripeWebhook);
```

## Handling Refunds

Sometimes orders need to be refunded. Stripe makes this straightforward.

```js
exports.refundOrder = asyncHandler(async (req, res) => {
  const order = await Order.findById(req.params.id);
  if (!order) throw new ApiError(404, 'Order not found');
  if (order.paymentStatus !== 'paid') throw new ApiError(400, 'Order not paid');

  // Find the payment intent from Stripe
  const sessions = await stripe.checkout.sessions.list({
    metadata: { userId: order.user.toString() },
  });

  const session = sessions.data.find(s =>
    s.payment_status === 'paid'
  );

  if (session && session.payment_intent) {
    await stripe.refunds.create({
      payment_intent: session.payment_intent,
    });
  }

  order.paymentStatus = 'refunded';
  order.orderStatus = 'cancelled';
  await order.save();

  res.json({ success: true, data: order });
});
```

## Payment Routes

```js
const router = require('express').Router();
const { protect, restrictTo } = require('../user/auth.middleware');

router.post('/checkout', protect, createCheckoutSession);
router.post('/webhook', stripeWebhook);
router.post('/:id/refund', protect, restrictTo('admin'), refundOrder);
```

Stripe handles the hard parts. We create sessions, listen for webhooks, and process refunds. The payment flow is secure because credit card data never touches our server.
