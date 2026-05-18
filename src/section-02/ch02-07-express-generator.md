# Express Generator

Express comes with a command-line tool that scaffolds a complete project for you. It is called `express-generator`, and it creates a working Express application with a reasonable structure in seconds. Let me show you how to use it and what it creates.

## Installing the Generator

You do not need to install it globally. You can run it with npx:

```bash
npx express-generator my-app
```

Or install it globally if you prefer:

```bash
npm install -g express-generator
express my-app
```

Either way, you get the same result.

## Command Options

The generator accepts several options:

```bash
# Use a specific template engine
express my-app --view=ejs
express my-app --view=pug
express my-app --view=hbs

# Use CSS preprocessor
express my-app --css=sass
express my-app --css=less

# Skip git initialization
express my-app --no-git

# Force overwrite if directory exists
express my-app --force
```

My typical command looks like this:

```bash
npx express-generator my-app --view=ejs --no-git
cd my-app
npm install
```

## What It Creates

Here is the directory structure the generator produces:

```
my-app/
  app.js
  package.json
  bin/
    www
  public/
    images/
    javascripts/
    stylesheets/
      style.css
  routes/
    index.js
    users.js
  views/
    error.ejs
    index.ejs
  package.json
```

Let me look at the key files.

**bin/www** is the server startup script. This is equivalent to the `server.js` file I showed you earlier, but separated into a `bin` directory:

```javascript
#!/usr/bin/env node

var app = require('../app');
var debug = require('debug')('my-app:server');
var http = require('http');

var port = normalizePort(process.env.PORT || '3000');
app.set('port', port);

var server = http.createServer(app);

server.listen(port);
server.on('error', onError);
server.on('listening', onListening);

function normalizePort(val) {
  var port = parseInt(val, 10);
  if (isNaN(port)) return val;
  if (port >= 0) return port;
  return false;
}

function onError(error) {
  if (error.syscall !== 'listen') throw error;
  var bind = typeof port === 'string' ? 'Pipe ' + port : 'Port ' + port;
  switch (error.code) {
    case 'EACCES':
      console.error(bind + ' requires elevated privileges');
      process.exit(1);
      break;
    case 'EADDRINUSE':
      console.error(bind + ' is already in use');
      process.exit(1);
      break;
    default:
      throw error;
  }
}

function onListening() {
  var addr = server.address();
  var bind = typeof addr === 'string' ? 'pipe ' + addr : 'port ' + addr.port;
  debug('Listening on ' + bind);
}
```

**app.js** is the Express configuration:

```javascript
var createError = require('http-errors');
var express = require('express');
var path = require('path');
var cookieParser = require('cookie-parser');
var logger = require('morgan');

var indexRouter = require('./routes/index');
var usersRouter = require('./routes/users');

var app = express();

// View engine setup
app.set('views', path.join(__dirname, 'views'));
app.set('view engine', 'ejs');

app.use(logger('dev'));
app.use(express.json());
app.use(express.urlencoded({ extended: false }));
app.use(cookieParser());
app.use(express.static(path.join(__dirname, 'public')));

app.use('/', indexRouter);
app.use('/users', usersRouter);

// Catch 404
app.use(function(req, res, next) {
  next(createError(404));
});

// Error handler
app.use(function(err, req, res, next) {
  res.locals.message = err.message;
  res.locals.error = req.app.get('env') === 'development' ? err : {};
  res.status(err.status || 500);
  res.render('error');
});

module.exports = app;
```

**routes/index.js** is the home page route:

```javascript
var express = require('express');
var router = express.Router();

router.get('/', function(req, res, next) {
  res.render('index', { title: 'Express' });
});

module.exports = router;
```

## Running the Generated App

```bash
cd my-app
npm install
npm start
```

The `start` script runs `node ./bin/www`. Visit `http://localhost:3000` and you should see a "Welcome to Express" page.

For development with auto-restart:

```bash
npm install --save-dev nodemon
```

Then add a dev script to `package.json`:

```json
{
  "scripts": {
    "start": "node ./bin/www",
    "dev": "nodemon ./bin/www"
  }
}
```

## What I Change

The generated structure is a good starting point, but I always make some changes:

**1. Add a .env file and dotenv.** The generator does not include environment variable support.

```bash
npm install dotenv
```

**2. Add security middleware.** The generator does not include helmet or cors.

```bash
npm install helmet cors
```

**3. Convert to modern JavaScript.** The generator uses `var`. I change everything to `const` and `let`, and use arrow functions.

**4. Add a src directory.** I move routes, models, and middleware into a `src` folder for better organization.

**5. Add a controllers directory.** The generator puts handler logic directly in route files. I prefer to separate them.

**6. Remove the views if building an API.** If I am building a REST API, I do not need templates. I remove the `views` directory and the view engine setup, and replace `res.render()` calls with `res.json()`.

## When to Use the Generator

I use the generator when I want a quick starting point and I am building a traditional server-rendered app. For APIs, I usually prefer to set up the project manually because I end up removing most of what the generator creates.

The generator is not a magic solution. It gives you a working app in seconds, but you still need to understand what every file does so you can customize it to your needs. Think of it as a template, not a final product.
