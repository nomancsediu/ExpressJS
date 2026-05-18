# EJS Templates

EJS stands for Embedded JavaScript. It is the template engine that looks the most like regular HTML. You just add special tags where you want dynamic content.

## Installation and Setup

```bash
npm install ejs
```

```js
const express = require('express');
const app = express();

app.set('view engine', 'ejs');
app.set('views', './views');
```

## EJS Tags

EJS has a few tag types, and they each do something different:

| Tag | Purpose | Output |
|-----|---------|--------|
| `<%= %>` | Output value (HTML escaped) | Shows in page |
| `<%- %>` | Output raw HTML (unescaped) | Shows in page |
| `<% %>` | Execute JavaScript | No output |
| `<%# %>` | Comment | Nothing |
| `-%>` | Trim trailing newline | No output |

## Basic Example

```js
app.get('/', (req, res) => {
  res.render('index', {
    title: 'My Blog',
    posts: [
      { title: 'Learning EJS', body: 'EJS is simple and fun' },
      { title: 'Express Tips', body: 'Always handle errors' }
    ]
  });
});
```

`views/index.ejs`:

```html
<!DOCTYPE html>
<html>
<head>
  <title><%= title %></title>
</head>
<body>
  <h1><%= title %></h1>

  <% posts.forEach(post => { %>
    <article>
      <h2><%= post.title %></h2>
      <p><%= post.body %></p>
    </article>
  <% }); %>
</body>
</html>
```

Notice how `<% %>` runs JavaScript (the forEach loop) without outputting anything, while `<%= %>` outputs the value. This separation keeps things readable.

## Conditionals

```html
<% if (user) { %>
  <p>Welcome back, <%= user.name %>!</p>
<% } else { %>
  <p>Please log in</p>
<% } %>
```

## Includes (Partials)

EJS lets you split templates into reusable pieces:

`views/partials/header.ejs`:

```html
<header>
  <nav>
    <a href="/">Home</a>
    <a href="/about">About</a>
  </nav>
</header>
```

Using it in another template:

```html
<%- include('partials/header') %>
<main>
  <h1><%= title %></h1>
</main>
```

Use `<%-` (unescaped) for includes because `include()` returns raw HTML. Passing data to includes works too:

```html
<%- include('partials/post-card', { post: post }) %>
```

## Escaping

EJS escapes HTML by default with `<%= %>`. This prevents XSS attacks:

```js
// If user input is: <script>alert('hack')</script>
// <%= userInput %> outputs: &lt;script&gt;alert('hack')&lt;/script&gt;
```

Only use `<%- %>` for trusted content or includes. Never use it for user input.

I like EJS because it is predictable. What you write looks like HTML with some JavaScript sprinkled in. No magic, no surprises.
