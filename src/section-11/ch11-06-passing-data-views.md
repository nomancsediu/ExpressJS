# Passing Data to Views

Templates are useless without data. This chapter covers all the ways to get information from your Express routes into your templates.

## res.render with Data

The most common way is passing a data object as the second argument to `res.render()`:

```js
app.get('/profile', (req, res) => {
  res.render('profile', {
    title: 'My Profile',
    user: { name: 'Alice', email: 'alice@example.com' },
    posts: [
      { title: 'First Post', likes: 5 },
      { title: 'Second Post', likes: 12 }
    ]
  });
});
```

In your template, all these properties become variables:

```html
<!-- EJS -->
<h1><%= user.name %></h1>
<p><%= user.email %></p>

<% posts.forEach(post => { %>
  <div><%= post.title %> - <%= post.likes %> likes</div>
<% }); %>
```

## Using app.locals

`app.locals` makes data available in every template. Good for site-wide values like the app name or navigation links:

```js
app.locals.siteName = 'My Awesome App';
app.locals.navLinks = [
  { label: 'Home', href: '/' },
  { label: 'Blog', href: '/blog' },
  { label: 'About', href: '/about' }
];
```

These are available in all templates without passing them explicitly:

```html
<title><%= siteName %></title>
<% navLinks.forEach(link => { %>
  <a href="<%= link.href %>"><%= link.label %></a>
<% }); %>
```

## Using res.locals

`res.locals` is scoped to the current request. It persists through middleware and into the template. This is useful for middleware that sets data:

```js
app.use((req, res, next) => {
  res.locals.currentUser = req.user || null;
  res.locals.isLoggedIn = !!req.user;
  next();
});

app.get('/dashboard', (req, res) => {
  // currentUser and isLoggedIn are already available
  res.render('dashboard', { title: 'Dashboard' });
});
```

I use `res.locals` a lot for authentication state. The middleware sets it once, and every template can check `isLoggedIn` without me passing it in every route.

## Dynamic Data

You can pass computed values and functions results:

```js
app.get('/stats', async (req, res) => {
  const userCount = await db.users.count();
  const recentPosts = await db.posts.findRecent(5);
  const isAdmin = req.user && req.user.role === 'admin';

  res.render('stats', {
    title: 'Statistics',
    userCount,
    recentPosts,
    isAdmin,
    lastUpdated: new Date().toLocaleString()
  });
});
```

## Helper Functions

For formatting or transforming data, register helper functions instead of computing everything in your route:

```js
// Handlebars
const hbs = require('hbs');
hbs.registerHelper('json', function(context) {
  return JSON.stringify(context);
});

hbs.registerHelper('truncate', function(str, len) {
  if (str.length > len) return str.substring(0, len) + '...';
  return str;
});
```

```handlebars
<p>{{truncate post.body 100}}</p>
<script>var data = {{{json postData}}};</script>
```

In EJS, you can pass functions directly or use `app.locals`:

```js
app.locals.formatDate = (date) => new Date(date).toLocaleDateString();
app.locals.truncate = (str, len) => str.length > len ? str.slice(0, len) + '...' : str;
```

```html
<p><%= formatDate(post.createdAt) %></p>
<p><%= truncate(post.body, 100) %></p>
```

Keep your routes focused on fetching data. Let helpers and template logic handle the presentation. This separation keeps your code maintainable as your app grows.
