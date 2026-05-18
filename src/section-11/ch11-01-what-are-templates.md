# What Are Template Engines?

Template engines let you write HTML with dynamic placeholders. Instead of sending a static page, you fill in the blanks with data from your server and send the result.

## SSR vs CSR

There are two main ways to build web pages:

**Server-Side Rendering (SSR):** The server generates the full HTML and sends it to the browser. The user sees content immediately. Template engines do SSR.

**Client-Side Rendering (CSR):** The server sends a mostly empty HTML page and JavaScript. The browser downloads the JS, which then builds the page. React and Vue work this way.

Neither is better in all cases. SSR is simpler, faster for initial load, and better for SEO. CSR is better for highly interactive apps where the page changes a lot without reloading.

## Setting a View Engine

Express makes it easy to use any template engine. You just need to set two things:

```js
const express = require('express');
const app = express();

// Tell Express which engine to use
app.set('view engine', 'ejs');

// Tell Express where your templates live
app.set('views', './views');
```

The `views` setting defaults to a `views` directory in your project root. You only need to set it if your templates are somewhere else.

## How It Works

When you call `res.render()`, Express does this:

1. Looks for the template file in your `views` directory
2. Passes your data to the template engine
3. The engine processes the template and produces HTML
4. Express sends that HTML to the browser

```js
app.get('/', (req, res) => {
  res.render('index', { title: 'My App', user: 'Alice' });
});
```

Express finds `views/index.ejs`, injects the data, and sends the resulting HTML. The `.ejs` extension is added automatically because you set `view engine` to `ejs`.

## Installing a Template Engine

Each engine is a separate npm package:

```bash
npm install ejs     # For EJS
npm install pug     # For Pug
npm install hbs     # For Handlebars
```

You only need to install the ones you want to use. Express detects the engine from your `view engine` setting.

## Choosing an Engine

Here is my quick comparison:

| Engine | Syntax Style | Logic in Templates | Learning Curve |
|--------|-------------|-------------------|----------------|
| EJS | HTML-like with `<% %>` tags | Full JavaScript | Low |
| Pug | Indentation-based, no brackets | Full JavaScript | Medium |
| Handlebars | `{{ }}` mustache syntax | Minimal (logic-less) | Low |

If you want something familiar, pick EJS. If you hate typing angle brackets, try Pug. If you want strict separation of logic and presentation, go with Handlebars.

I started with EJS because it looked like normal HTML with some extra tags. That made the learning curve almost flat.
