# Review and Rating Module

Reviews help other buyers decide. Ratings give a quick sense of quality. Together they make the store trustworthy. Let me build a review system that only allows verified buyers to leave reviews.

## Review Model

Each review belongs to one product and one user. A user can only leave one review per product.

```js
// src/modules/review/review.model.js
const mongoose = require('mongoose');

const reviewSchema = new mongoose.Schema({
  user: { type: mongoose.Schema.Types.ObjectId, ref: 'User', required: true },
  product: { type: mongoose.Schema.Types.ObjectId, ref: 'Product', required: true },
  rating: { type: Number, required: true, min: 1, max: 5 },
  title: { type: String, trim: true },
  comment: { type: String, required: true, maxlength: 1000 },
}, { timestamps: true });

reviewSchema.index({ user: 1, product: 1 }, { unique: true });

module.exports = mongoose.model('Review', reviewSchema);
```

The compound unique index on `user` and `product` prevents duplicate reviews. One review per user per product. Simple.

## Review Controller

### Creating a Review

I check that the user actually bought the product before letting them review it.

```js
// src/modules/review/review.controller.js
const Review = require('./review.model');
const Order = require('../order/order.model');
const ApiError = require('../../utils/ApiError');
const asyncHandler = require('../../middleware/asyncHandler');

exports.createReview = asyncHandler(async (req, res) => {
  const { productId, rating, title, comment } = req.body;

  // Check if user purchased this product
  const hasPurchased = await Order.exists({
    user: req.user._id,
    'items.product': productId,
    orderStatus: { $in: ['confirmed', 'shipped', 'delivered'] },
  });

  if (!hasPurchased) {
    throw new ApiError(403, 'You can only review products you have purchased');
  }

  // Check for existing review
  const existingReview = await Review.findOne({
    user: req.user._id,
    product: productId,
  });

  if (existingReview) {
    throw new ApiError(400, 'You already reviewed this product');
  }

  const review = await Review.create({
    user: req.user._id,
    product: productId,
    rating,
    title,
    comment,
  });

  res.status(201).json({ success: true, data: review });
});
```

### Getting Reviews for a Product

I also calculate the average rating so the frontend can show stars.

```js
exports.getProductReviews = asyncHandler(async (req, res) => {
  const { page = 1, limit = 10 } = req.query;

  const reviews = await Review.find({ product: req.params.productId })
    .sort({ createdAt: -1 })
    .skip((page - 1) * limit)
    .limit(Number(limit))
    .populate('user', 'name');

  const stats = await Review.aggregate([
    { $match: { product: new mongoose.Types.ObjectId(req.params.productId) } },
    {
      $group: {
        _id: '$product',
        averageRating: { $avg: '$rating' },
        totalReviews: { $sum: 1 },
        ratingDistribution: {
          $push: '$rating',
        },
      },
    },
  ]);

  res.json({
    success: true,
    data: reviews,
    stats: stats[0] || { averageRating: 0, totalReviews: 0 },
  });
});
```

### Updating and Deleting Reviews

Users can edit their own reviews. Admins can delete any review.

```js
exports.updateReview = asyncHandler(async (req, res) => {
  let review = await Review.findById(req.params.id);
  if (!review) throw new ApiError(404, 'Review not found');

  if (review.user.toString() !== req.user._id.toString()) {
    throw new ApiError(403, 'Not your review');
  }

  review = await Review.findByIdAndUpdate(req.params.id, req.body, { new: true, runValidators: true });
  res.json({ success: true, data: review });
});

exports.deleteReview = asyncHandler(async (req, res) => {
  const review = await Review.findById(req.params.id);
  if (!review) throw new ApiError(404, 'Review not found');

  if (review.user.toString() !== req.user._id.toString() && req.user.role !== 'admin') {
    throw new ApiError(403, 'Not authorized');
  }

  await review.deleteOne();
  res.json({ success: true, message: 'Review deleted' });
});
```

## Review Routes

```js
const router = require('express').Router();
const { protect, restrictTo } = require('../user/auth.middleware');

router.post('/', protect, createReview);
router.get('/product/:productId', getProductReviews);
router.patch('/:id', protect, updateReview);
router.delete('/:id', protect, deleteReview);
```

The purchase check is the key feature here. It keeps reviews honest. Only people who actually bought the product can share their opinion. The aggregate query gives us the average rating without any extra computation on the frontend.
