# Deployment: Getting Your App Live

You built something. It works on your machine. Now what? It is time to put it on the internet where real people can use it. This section is all about deploying Express applications.

I used to think deployment was just uploading files to a server. It is way more than that. Deployment means making your app production-ready, secure, monitored, and able to handle real traffic. It also means you can push updates without breaking everything.

## What This Section Covers

We are going to walk through the full deployment journey:

- **Production best practices**: Things you must do before going live
- **Environment configuration**: Managing settings and secrets across environments
- **Docker**: Packaging your app so it runs the same everywhere
- **Platform deployment**: Quick deploys to Railway and Render
- **AWS deployment**: Full control with EC2, Elastic Beanstalk, RDS, and S3
- **Vercel deployment**: Serverless Express for simpler apps
- **Nginx reverse proxy**: Putting Nginx in front of Express
- **SSL and HTTPS**: Encrypting traffic with Let's Encrypt
- **CI/CD pipelines**: Automating tests and deploys with GitHub Actions
- **Logging and monitoring**: Knowing what is happening in production

## The Mindset Shift

Development is about making things work. Production is about keeping things working. In production, your app needs to:

- Start up reliably every time
- Handle errors without crashing
- Serve traffic under load
- Keep secrets safe
- Tell you when something goes wrong

A lot of what we do in this section is not glamorous. Config files, security headers, SSL certificates. But this is what separates a weekend project from a real application.

## A Quick Word on Costs

Some of the platforms we cover (Railway, Render) have free tiers. AWS gives you a free tier for the first year. You can follow along without spending money. When you are ready to go live for real, you will have a clear picture of what each option costs and offers.

Let me get your app out of localhost and onto the internet.
