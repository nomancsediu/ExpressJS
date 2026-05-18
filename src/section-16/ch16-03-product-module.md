# Product Module

Products are the heart of any e-commerce app. Without products, there is nothing to sell. Let me build the product module with full CRUD, image upload, search, and filtering.

## Product Model

A product needs a name, description, price, stock quantity, category, brand, and images. I also add a flag for featured products.

```js
// src/modules/product/product.model.js
const mongoose = require('mongoose');

const productSchema = new mongoose.Schema({
  name: { type: String, required: true, trim: true },
  description: { type: String, required: true },
  price: { type: Number, required: true, min: 0 },
  stock: { type: Number, required: true, min: 0, default: 0 },
  category: { type: mongoose.Schema.Types.ObjectId, ref: 'Category', required: true },
  brand: { type: mongoose.Schema.Types.ObjectId, ref: 'Brand' },
  images: [{ type: String }],
  featured: { type: Boolean, default: false },
}, { timestamps: true, toJSON: { virtuals: true } });

productSchema.index({ name: 'text', description: 'text' });

module.exports = mongoose.model('Product', productSchema);
```

I added a text index on name and description. This makes text search fast.

## CRUD Controller

I build standard CRUD operations plus a few extras.

```js
// src/modules/product/product.controller.js
const Product = require('./product.model');
const ApiError = require('../../utils/ApiError');
const asyncHandler = require('../../middleware/asyncHandler');

exports.createProduct = asyncHandler(async (req, res) => {
  const product = await Product.create(req.body);
  res.status(201).json({ success: true, data: product });
});

exports.getProducts = asyncHandler(async (req, res) => {
  const { page = 1, limit = 10, sort, search, category, minPrice, maxPrice } = req.query;

  const filter = {};
  if (category) filter.category = category;
  if (minPrice || maxPrice) {
    filter.price = {};
    if (minPrice) filter.price.$gte = Number(minPrice);
    if (maxPrice) filter.price.$lte = Number(maxPrice);
  }
  if (search) filter.$text = { $search: search };

  const sortBy = sort === 'price_asc' ? { price: 1 }
    : sort === 'price_desc' ? { price: -1 }
    : sort === 'newest' ? { createdAt: -1 }
    : { createdAt: -1 };

  const products = await Product.find(filter)
    .sort(sortBy)
    .skip((page - 1) * limit)
    .limit(Number(limit))
    .populate('category', 'name')
    .populate('brand', 'name');

  const total = await Product.countDocuments(filter);

  res.json({
    success: true,
    data: products,
    pagination: { page: Number(page), limit: Number(limit), total, pages: Math.ceil(total / limit) },
  });
});

exports.getProduct = asyncHandler(async (req, res) => {
  const product = await Product.findById(req.params.id).populate('category').populate('brand');
  if (!product) throw new ApiError(404, 'Product not found');
  res.json({ success: true, data: product });
});

exports.updateProduct = asyncHandler(async (req, res) => {
  const product = await Product.findByIdAndUpdate(req.params.id, req.body, { new: true, runValidators: true });
  if (!product) throw new ApiError(404, 'Product not found');
  res.json({ success: true, data: product });
});

exports.deleteProduct = asyncHandler(async (req, res) => {
  const product = await Product.findByIdAndDelete(req.params.id);
  if (!product) throw new ApiError(404, 'Product not found');
  res.json({ success: true, message: 'Product deleted' });
});
```

The getProducts endpoint handles filtering by category and price range, text search, sorting, and pagination all in one.

## Image Upload

I use Multer to handle image uploads. Each product can have multiple images.

```js
// src/modules/product/upload.middleware.js
const multer = require('multer');
const path = require('path');

const storage = multer.diskStorage({
  destination: (req, file, cb) => cb(null, 'uploads/products'),
  filename: (req, file, cb) => {
    const ext = path.extname(file.originalname);
    cb(null, `product-${Date.now()}-${Math.round(Math.random() * 1e9)}${ext}`);
  },
});

const fileFilter = (req, file, cb) => {
  if (file.mimetype.startsWith('image/')) cb(null, true);
  else cb(new Error('Only images allowed'), false);
};

exports.uploadProductImages = multer({ storage, fileFilter }).array('images', 5);
```

Then in the route, I chain the upload middleware before the controller:

```js
// src/modules/product/product.routes.js
const router = require('express').Router();
const { uploadProductImages } = require('./upload.middleware');

router.post('/', protect, restrictTo('admin'), uploadProductImages, createProduct);
router.patch('/:id', protect, restrictTo('admin'), uploadProductImages, updateProduct);
```

This keeps image handling separate from business logic. Clean and reusable.
