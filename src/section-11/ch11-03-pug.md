# Pug Templates

Pug (formerly Jade) takes a completely different approach. No angle brackets, no closing tags. You use indentation to structure HTML. It feels weird at first, but once you get used to it, templates become very concise.

## Installation and Setup

```bash
npm install pug
```

```js
const express = require('express');
const app = express();

app.set('view engine', 'pug');
app.set('views', './views');
```

## Basic Syntax

`views/index.pug`:

```pug
doctype html
html
  head
    title= title
  body
    h1= title
    p Welcome to my app
```

This compiles to standard HTML. Indentation determines nesting. No closing tags needed. Attributes go in parentheses:

```pug
a(href="/about") About Us
img(src="/logo.png", alt="Logo")
input(type="text", placeholder="Search")
```

## Variables and Expressions

Use `=` to output escaped content, `!=` for unescaped:

```pug
h1= title
p= message
div!= rawHTML
```

You can also use inline interpolation:

```pug
p Hello, #{user.name}!
p Your balance is $#{user.balance.toFixed(2)}
```

## Conditionals

```pug
if user
  h2 Welcome back, #{user.name}
  a(href="/logout") Log Out
else
  h2 Welcome, stranger
  a(href="/login") Log In
```

There is also `unless` for the opposite of `if`:

```pug
unless loggedIn
  p Please log in first
```

## Loops

```pug
ul
  each item in items
    li= item.name

each post in posts
  article
    h2= post.title
    p= post.body
```

You can get the index too:

```pug
each user, index in users
  li #{index + 1}. #{user.name}
```

## Mixins

Mixins are reusable template blocks, like functions for HTML:

```pug
mixin card(title, description)
  .card
    h3= title
    p= description

+card('Feature One', 'This is the first feature')
+card('Feature Two', 'This is the second feature')
```

Mixins can also take blocks of content:

```pug
mixin sidebar
  .sidebar
    if block
      block
    else
      p Default sidebar content

+sidebar
  p Custom sidebar content here
```

## Extends and Blocks

Pug supports template inheritance with `extends`:

`views/layout.pug`:

```pug
doctype html
html
  head
    title= title
    block head
  body
    block content
    footer
      p &copy; 2024 My App
```

`views/index.pug`:

```pug
extends layout

block content
  h1= title
  p Welcome
```

Child templates override blocks from the parent. This is powerful for maintaining consistent layouts.

Pug takes some getting used to. The indentation-based syntax can trip you up if your editor has inconsistent tabs and spaces. I set my editor to convert tabs to spaces, which fixed most of my Pug headaches.
