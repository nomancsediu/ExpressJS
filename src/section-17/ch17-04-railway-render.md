# Deploying to Railway and Render

Not every project needs AWS. Sometimes you just want to push code and have it live on the internet in minutes. Railway and Render are platform-as-a-service (PaaS) options that do exactly that.

## Railway

Railway is fast. You connect your GitHub repo, and it deploys on every push.

### Setup Steps

1. Go to [railway.app](https://railway.app) and sign in with GitHub
2. Click "New Project" and select "Deploy from GitHub repo"
3. Pick your repository
4. Railway detects the Node.js app and builds it automatically

### Configuration

Railway needs a few settings to run Express properly:

```bash
# In Railway dashboard, set these environment variables
NODE_ENV=production
PORT=3000
MONGODB_URI=mongodb+srv://...
JWT_SECRET=your-production-secret
JWT_REFRESH_SECRET=your-production-refresh-secret
```

Railway assigns a random port internally, so I update my server to use it:

```js
const PORT = process.env.PORT || 3000;
```

Railway sets the `PORT` env var automatically. My app reads it and works.

### Adding a MongoDB Database

Railway offers managed databases. I can add one right in the dashboard:

1. Click "New" and select "Database" then "MongoDB"
2. Railway provisions the database and gives me a connection string
3. I copy the connection string into my `MONGODB_URI` env var

That is it. No separate database server to manage.

### Custom Domain

In the Railway project settings, I can add a custom domain. Railway handles SSL automatically.

## Render

Render is similar to Railway but with a slightly different workflow.

### Setup Steps

1. Go to [render.com](https://render.com) and sign in with GitHub
2. Click "New" and select "Web Service"
3. Connect your repository
4. Configure the build and start commands:
   - **Build Command**: `npm install`
   - **Start Command**: `node src/server.js`

### Environment Variables

Set the same variables in the Render dashboard under "Environment":

```bash
NODE_ENV=production
MONGODB_URI=mongodb+srv://...
JWT_SECRET=your-production-secret
```

### Adding a Managed Database

Render offers managed PostgreSQL and Redis but not MongoDB. For MongoDB, I use MongoDB Atlas (free tier) and put the connection string in the Render env vars.

### Render Specifics

Render gives each web service a `.onrender.com` domain with free SSL. The free tier has some limitations:

- The service spins down after 15 minutes of inactivity
- First request after spin-down takes about 30 seconds
- 750 hours of free runtime per month

For a portfolio project or low-traffic app, this is fine. For production, upgrade to a paid plan.

## Which One to Pick?

Both platforms are excellent for getting started:

- **Railway** is faster to set up and includes MongoDB. Great for quick prototypes.
- **Render** has a more generous free tier and better documentation. Good for learning.

Neither requires you to manage servers. You push code, they deploy. When you outgrow them, you can move to AWS or your own infrastructure with the same Docker setup we covered earlier.
