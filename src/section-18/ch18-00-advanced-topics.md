# Advanced Topics

You know the basics. You can build an Express app, connect it to a database, add auth, deploy it. Now it is time to go deeper. This section covers the topics that separate a beginner from someone who really understands what they are doing.

## What Makes These Topics Advanced?

These are not harder for the sake of being harder. They are advanced because they matter more at scale. When your app has thousands of users, or when you need to maintain it for years, these patterns and techniques make the difference.

## What We Will Cover

- **Behind proxies**: How Express handles requests when it sits behind Nginx or a load balancer. Understanding `X-Forwarded-For` and `trust proxy`.

- **Graceful shutdown**: What happens when your server needs to restart. How to close connections properly instead of cutting them off.

- **Health checks**: Building endpoints that tell monitoring systems if your app is healthy. Including checking dependencies like databases.

- **Microservices**: Breaking a big app into smaller services that communicate over the network. When to do it and when not to.

- **GraphQL**: An alternative to REST that lets clients ask for exactly the data they need. Using express-graphql or Apollo Server.

- **Event-driven architecture**: Using Node.js EventEmitter and custom events to decouple parts of your application.

- **Message queues**: Background processing with RabbitMQ and BullMQ. Offloading slow work from request handlers.

- **Clean architecture**: Organizing your code so it is maintainable and testable. Separation of concerns, dependency injection, SOLID principles.

- **TypeScript**: Adding type safety to Express. Types for requests, responses, and middleware. Making refactoring safe.

- **Project structure**: How to organize a growing codebase. Feature-based vs layer-based. Barrel exports. Scaling patterns.

## A Note on Learning These

You do not need all of these for every project. A small API does not need microservices. A weekend project does not need clean architecture. But when the time comes, knowing these patterns gives you options.

I learned most of these the hard way: by building something, hitting a wall, and then discovering there was a better way. You do not have to learn the hard way. That is what this section is for.

Let us go beyond the basics.
