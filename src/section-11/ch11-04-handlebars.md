# Handlebars Templates

Handlebars takes a logic-less approach. It keeps templates simple and pushes complex logic to your JavaScript code. If you believe templates should not contain programming logic, Handlebars is for you.

## Installation and Setup

```bash
npm install hbs
```

```js
const express = require('express');
const app = express();

app.set('view engine', 'hbs');
app.set('views', './views');
```

## Basic Syntax

Handlebars uses double curly braces `{{ }}` for expressions:

`views/index.hbs`:

```handlebars
<!DOCTYPE html>
<html>
<head>
  <title>{{title}}</title>
</head>
<body>
  <h1>{{title}}</h1>
  <p>{{message}}</p>
</body>
</html>
```

Handlebars escapes HTML automatically. Use triple braces `{{{ }}}` for raw output:

```handlebars
<p>{{escapedContent}}</p>     <!-- HTML escaped -->
<p>{{{rawContent}}}</p>       <!-- Unescaped -->
```

## Conditionals

Handlebars has built-in `if` and `unless` helpers:

```handlebars
{{#if user}}
  <p>Welcome, {{user.name}}!</p>
{{else}}
  <p>Please log in</p>
{{/if}}

{{#unless loggedIn}}
  <p>You must log in first</p>
{{/unless}}
```

Important: `{{#if}}` only checks for truthy values. It does not support comparisons like `{{#if age > 18}}`. For that, you need a custom helper.

## Loops

```handlebars
<ul>
  {{#each items}}
    <li>{{this.name}} - ${{this.price}}</li>
  {{/each}}
</ul>
```

Inside an `{{#each}}` block, `this` refers to the current item. You can also use `@index` for the loop index and `@first`/`@last` for edge detection:

```handlebars
{{#each users}}
  <p>{{@index}}. {{this.name}}</p>
  {{#if @first}}<span>Newest!</span>{{/if}}
{{/each}}
```

## Partials

Partials are reusable template snippets. Register them in your app:

```js
const hbs = require('hbs');
hbs.registerPartials('./views/partials');
```

Then use them in templates:

```handlebars
{{> header}}

<main>
  <h1>{{title}}</h1>
</main>

{{> footer}}
```

`views/partials/header.hbs`:

```handlebars
<header>
  <nav>
    <a href="/">Home</a>
    <a href="/about">About</a>
  </nav>
</header>
```

Pass data to partials:

```handlebars
{{> postCard post=this}}
```

## Custom Helpers

This is where Handlebars shines. Instead of putting logic in templates, you write helpers in JavaScript:

```js
hbs.registerHelper('formatDate', function(date) {
  return new Date(date).toLocaleDateString();
});

hbs.registerHelper('eq', function(a, b) {
  return a === b;
});
```

Use them in templates:

```handlebars
<p>Published: {{formatDate post.createdAt}}</p>

{{#if (eq user.role 'admin')}}
  <a href="/admin">Admin Panel</a>
{{/if}}
```

The `eq` helper lets you do comparisons that `{{#if}}` cannot do alone. I register helpers for common operations like formatting dates, comparing values, and generating URLs.

Handlebars keeps your templates clean because the logic lives in JavaScript, not in the HTML. This separation makes templates easier to read and maintain.
