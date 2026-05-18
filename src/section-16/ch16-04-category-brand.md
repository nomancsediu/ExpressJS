# Category and Brand Modules

Products need organization. Categories group similar products together. Brands tell you who made them. Both are simple modules but they make the store actually usable.

## Category Model

Categories can have subcategories. I handle this with a self-referencing parent field.

```js
// src/modules/category/category.model.js
const mongoose = require('mongoose');

const categorySchema = new mongoose.Schema({
  name: { type: String, required: true, unique: true, trim: true },
  slug: { type: String, required: true, unique: true, lowercase: true },
  parent: { type: mongoose.Schema.Types.ObjectId, ref: 'Category', default: null },
  description: { type: String },
}, { timestamps: true });

module.exports = mongoose.model('Category', categorySchema);
```

The `parent` field is what makes subcategories work. A category with no parent is a top-level category. One with a parent is a subcategory.

## Category Controller

I want to list categories in a tree structure, not just a flat list. This makes the frontend easier to build.

```js
// src/modules/category/category.controller.js
const Category = require('./category.model');
const ApiError = require('../../utils/ApiError');
const asyncHandler = require('../../middleware/asyncHandler');

exports.createCategory = asyncHandler(async (req, res) => {
  const slug = req.body.name.toLowerCase().replace(/[^a-z0-9]+/g, '-');
  const category = await Category.create({ ...req.body, slug });
  res.status(201).json({ success: true, data: category });
});

exports.getCategories = asyncHandler(async (req, res) => {
  const categories = await Category.find().populate('parent', 'name slug');

  // Build a tree structure
  const tree = categories.filter(c => !c.parent);
  tree.forEach(parent => {
    parent._doc.subcategories = categories.filter(c =>
      c.parent && c.parent._id.toString() === parent._id.toString()
    );
  });

  res.json({ success: true, data: tree });
});

exports.getCategory = asyncHandler(async (req, res) => {
  const category = await Category.findById(req.params.id).populate('parent');
  if (!category) throw new ApiError(404, 'Category not found');
  res.json({ success: true, data: category });
});

exports.updateCategory = asyncHandler(async (req, res) => {
  const slug = req.body.name ? req.body.name.toLowerCase().replace(/[^a-z0-9]+/g, '-') : undefined;
  const category = await Category.findByIdAndUpdate(
    req.params.id,
    { ...req.body, ...(slug && { slug }) },
    { new: true, runValidators: true }
  );
  if (!category) throw new ApiError(404, 'Category not found');
  res.json({ success: true, data: category });
});

exports.deleteCategory = asyncHandler(async (req, res) => {
  const category = await Category.findByIdAndDelete(req.params.id);
  if (!category) throw new ApiError(404, 'Category not found');
  res.json({ success: true, message: 'Category deleted' });
});
```

The tree-building logic in `getCategories` turns a flat list into a nested structure. The frontend gets a ready-to-use tree.

## Brand Model

Brands are simpler than categories. Just a name, slug, and optional logo.

```js
// src/modules/brand/brand.model.js
const mongoose = require('mongoose');

const brandSchema = new mongoose.Schema({
  name: { type: String, required: true, unique: true, trim: true },
  slug: { type: String, required: true, unique: true, lowercase: true },
  logo: { type: String },
  description: { type: String },
}, { timestamps: true });

module.exports = mongoose.model('Brand', brandSchema);
```

## Linking Products to Categories and Brands

In the product model, I already added `category` and `brand` as ObjectId references. When I query products, I populate these fields:

```js
const products = await Product.find()
  .populate('category', 'name slug')
  .populate('brand', 'name slug logo');
```

This gives me the category and brand details right inside the product response. No extra requests needed on the frontend.

## Filtering by Category or Brand

The product module already handles this. When the frontend sends `?category=abc123` or `?brand=xyz789`, the filter kicks in:

```js
if (category) filter.category = category;
if (brand) filter.brand = brand;
```

Simple and effective. The user can browse by category, drill into subcategories, or filter by brand. Each module stays focused on its own job.
