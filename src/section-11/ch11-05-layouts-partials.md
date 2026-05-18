# Layouts and Partials

No one wants to copy and paste the same header, footer, and sidebar into every template. Layouts and partials solve this problem. They let you define reusable pieces and compose pages from shared components.

## What Are Layouts?

A layout is a wrapper template that holds the common structure of your pages. The header, footer, navigation, and script tags usually go in the layout. Only the unique content changes between pages.

## What Are Partials?

Partials are smaller reusable chunks. A navigation bar, a product card, a comment section. You include them wherever you need them.

## EJS Partials

EJS does not have built-in layout support, but it handles partials well with `include`:

`views/partials/header.ejs`:

```html
<header class="site-header">
  <nav>
    <a href="/">Home</a>
    <a href="/blog">Blog</a>
    <a href="/contact">Contact</a>
  </nav>
</header>
```

`views/partials/footer.ejs`:

```html
<footer>
  <p>&copy; 2024 My App</p>
</footer>
```

`views/index.ejs`:

```html
<!DOCTYPE html>
<html>
<head><title><%= title %></title></head>
<body>
  <%- include('partials/header') %>
  <main>
    <h1><%= title %></h1>
  </main>
  <%- include('partials/footer') %>
</body>
</html>
```

For layout support in EJS, use the `express-ejs-layouts` package:

```bash
npm install express-ejs-layouts
```

```js
const expressLayouts = require('express-ejs-layouts');

app.use(expressLayouts);
app.set('layout', 'layouts/main');
```

`views/layouts/main.ejs`:

```html
<!DOCTYPE html>
<html>
<head><title><%= title %></title></head>
<body>
  <%- body %>
</body>
</html>
```

Now your route templates only need the unique content. The layout wraps them automatically.

## Pug Layouts

Pug has built-in layout support with `extends` and `block`:

`views/layout.pug`:

```pug
doctype html
html
  head
    title= title
    block head
  body
    include partials/nav
    main
      block content
    include partials/footer
```

`views/index.pug`:

```pug
extends layout

block content
  h1= title
  p Welcome to the homepage
```

## Handlebars Partials

Register partials and use `{{> name }}`:

```js
const hbs = require('hbs');
hbs.registerPartials('./views/partials');
```

```handlebars
{{> header}}
<main>
  {{> postCard post=featuredPost}}
</main>
{{> footer}}
```

## Nested Layouts

For complex apps, you might need nested layouts. A main layout wraps an admin layout, which wraps individual admin pages.

In Pug this is natural:

```pug
// views/admin-layout.pug
extends layout

block content
  .admin-wrapper
    include partials/admin-sidebar
    .admin-content
      block adminContent
```

```pug
// views/admin/dashboard.pug
extends admin-layout

block adminContent
  h1 Dashboard
```

In EJS with `express-ejs-layouts`, you can set different layouts per route:

```js
app.get('/admin', (req, res) => {
  res.render('admin/dashboard', { layout: 'layouts/admin' });
});
```

Partials and layouts save you from repeating yourself. Start with a simple layout and add partials as your templates grow. I wish I had learned this pattern earlier instead of duplicating HTML across dozens of files.
