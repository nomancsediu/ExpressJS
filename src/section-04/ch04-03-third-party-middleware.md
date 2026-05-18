# Third-party Middleware

Built-in middleware covers the basics, but real projects need more. That is where third-party middleware comes in. The Express ecosystem has thousands of packages, but a handful show up in almost every project.

## morgan

Morgan logs HTTP requests. It tells you what came in, what went out, and how long it took. This is the first thing I add to any new Express project.

```js
const morgan = require('morgan');

app.use(morgan('dev'));
// GET /api/users 200 5.234 ms - 42
```

The `dev` format is great for development. For production, `combined` gives you more detail in Apache log format.

## cors

The CORS middleware handles Cross-Origin Resource Sharing. If your front-end is on a different domain than your API, you need this. Without it, browsers will block requests.

```js
const cors = require('cors');

app.use(cors()); // allow all origins
app.use(cors({ origin: 'https://myapp.com' })); // allow one origin
```

I spent a whole afternoon debugging a "network error" once, and it turned out I just forgot to add CORS headers. Never again.

## helmet

Helmet sets various HTTP headers to secure your app. It protects against well-known web vulnerabilities like clickjacking and sniffing.

```js
const helmet = require('helmet');

app.use(helmet());
```

One line and your app is significantly safer. Helmet is so important that I consider it mandatory for any production server.

## cookie-parser

This parses the `Cookie` header and populates `req.cookies` with an object.

```js
const cookieParser = require('cookie-parser');

app.use(cookieParser('my-secret-key'));

app.get('/', (req, res) => {
  console.log(req.cookies); // unsigned cookies
  console.log(req.signedCookies); // signed cookies
  res.send('Check your cookies');
});
```

Pass a secret string to enable signed cookies. Signed cookies cannot be tampered with because they carry a cryptographic signature.

## compression

This compresses response bodies using gzip. It makes your responses smaller and your app faster, especially for JSON APIs.

```js
const compression = require('compression');

app.use(compression());
```

Most clients support gzip decompression automatically, so this is essentially free performance.

## Installing Them All

```bash
npm install morgan cors helmet cookie-parser compression
```

I usually add all of these on day one of a new project. They are lightweight, well-maintained, and solve problems you will definitely run into. Later in this section, I will go deeper into configuring each one.
