# Deep Dive: Common Middleware Libraries

Earlier I introduced morgan, cors, and helmet. Now I want to go deeper into how to configure each one for real projects. The defaults are fine for getting started, but production needs more care.

## Morgan Formats and Options

Morgan comes with several predefined formats:

```js
morgan('dev');      // colored, concise, great for development
morgan('combined'); // Apache standard log format, great for production
morgan('common');   // Apache standard without response headers
morgan('short');    // shorter than common
morgan('tiny');     // method, url, status, response length, time
```

You can also create custom formats:

```js
morgan(':method :url :status :res[content-length] - :response-time ms');
```

Available tokens include `:method`, `:url`, `:status`, `:response-time`, `:date`, `:user-agent`, and many more.

I like to skip logging for health checks in production:

```js
app.use(morgan('combined', {
  skip: (req, res) => req.url === '/health',
}));
```

You can also write logs to a file instead of the console:

```js
const fs = require('fs');
const stream = fs.createWriteStream('access.log', { flags: 'a' });
app.use(morgan('combined', { stream }));
```

## CORS Options

The default `cors()` allows everything. In production, you should be more specific:

```js
const corsOptions = {
  origin: ['https://myapp.com', 'https://admin.myapp.com'],
  methods: ['GET', 'POST', 'PUT', 'DELETE'],
  allowedHeaders: ['Content-Type', 'Authorization'],
  credentials: true,
  maxAge: 86400, // preflight cache for 24 hours
};

app.use(cors(corsOptions));
```

Dynamic origin checking is useful when you have multiple allowed domains:

```js
const corsOptions = {
  origin: (origin, callback) => {
    const allowed = ['https://myapp.com', 'https://staging.myapp.com'];
    if (!origin || allowed.includes(origin)) {
      callback(null, true);
    } else {
      callback(new Error('Not allowed by CORS'));
    }
  },
};
```

The `credentials: true` option is required if your front-end sends cookies or authorization headers. Without it, browsers will not include credentials in cross-origin requests.

## Helmet Configuration

Helmet is actually a collection of smaller middleware functions. You can enable or disable each one:

```js
app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      styleSrc: ["'self'", 'cdn.jsdelivr.net'],
      scriptSrc: ["'self'"],
    },
  },
  referrerPolicy: { policy: 'strict-origin-when-cross-origin' },
  hsts: { maxAge: 31536000, includeSubDomains: true },
}));
```

If you need to disable a specific header:

```js
app.use(helmet({ noSniff: false }));
```

Content Security Policy (CSP) is the most powerful and most complex helmet feature. It tells the browser which sources of content are allowed. Start simple and expand as needed.

These three libraries form the backbone of most Express servers. Taking the time to configure them properly makes your app more secure, more observable, and more production-ready.
