# GraphQL

REST APIs have a problem: you get too much or too little data. Need a user's name? You get the whole user object. Need a list of product names with their categories? That might require multiple requests. GraphQL fixes this by letting the client ask for exactly what it needs.

## REST vs GraphQL

With REST, I need multiple endpoints:

```
GET /api/users/1          -> full user object
GET /api/users/1/orders   -> user's orders
GET /api/products?category=shoes -> filtered products
```

With GraphQL, I send one request:

```graphql
query {
  user(id: "1") {
    name
    email
    orders {
      id
      totalAmount
      items {
        product {
          name
          price
        }
        quantity
      }
    }
  }
}
```

I get exactly the fields I asked for. Nothing more, nothing less. One request.

## Setting Up with Express

I use Apollo Server with Express:

```bash
npm install @apollo/server express graphql cors
```

```js
// src/graphql/setup.js
const { ApolloServer } = require('@apollo/server');
const { expressMiddleware } = require('@apollo/server/express4');
const { typeDefs, resolvers } = require('./schema');

const setupGraphQL = async (app) => {
  const server = new ApolloServer({ typeDefs, resolvers });
  await server.start();

  app.use('/graphql', cors(), express.json(), expressMiddleware(server));
};

module.exports = setupGraphQL;
```

## Defining the Schema

The schema defines what data is available and how to query it:

```js
// src/graphql/schema.js
const typeDefs = `#graphql
  type User {
    id: ID!
    name: String!
    email: String!
    role: String!
    orders: [Order!]!
  }

  type Product {
    id: ID!
    name: String!
    description: String!
    price: Float!
    stock: Int!
    category: Category!
  }

  type Category {
    id: ID!
    name: String!
    products: [Product!]!
  }

  type Order {
    id: ID!
    totalAmount: Float!
    orderStatus: String!
    items: [OrderItem!]!
  }

  type OrderItem {
    product: Product!
    quantity: Int!
    price: Float!
  }

  type Query {
    user(id: ID!): User
    products(category: String, limit: Int): [Product!]!
    product(id: ID!): Product
    me: User!
  }

  type Mutation {
    createProduct(name: String!, price: Float!, stock: Int!, category: String!): Product!
    updateCart(productId: ID!, quantity: Int!): Cart!
  }
`;
```

## Writing Resolvers

Resolvers fetch the data for each field:

```js
const resolvers = {
  Query: {
    user: async (_, { id }) => User.findById(id),
    products: async (_, { category, limit = 10 }) => {
      const filter = category ? { category } : {};
      return Product.find(filter).limit(limit).populate('category');
    },
    product: async (_, { id }) => Product.findById(id).populate('category'),
    me: async (_, __, context) => {
      if (!context.user) throw new Error('Not authenticated');
      return User.findById(context.user.id);
    },
  },
  User: {
    orders: async (parent) => Order.find({ user: parent.id }),
  },
  Category: {
    products: async (parent) => Product.find({ category: parent.id }),
  },
};
```

Each resolver is responsible for fetching its own data. GraphQL only calls the resolvers for fields the client requested. If the client does not ask for `orders`, the `User.orders` resolver never runs.

## Authentication Context

I pass the authenticated user into the GraphQL context:

```js
app.use('/graphql', cors(), express.json(), expressMiddleware(server, {
  context: async ({ req }) => {
    const token = req.headers.authorization?.split(' ')[1];
    if (!token) return { user: null };

    try {
      const decoded = jwt.verify(token, process.env.JWT_SECRET);
      const user = await User.findById(decoded.id);
      return { user };
    } catch {
      return { user: null };
    }
  },
}));
```

## When to Use GraphQL

GraphQL is great when:

- Your frontend needs flexibility in what data it fetches
- You have complex nested data relationships
- Mobile apps need to minimize data transfer

It adds complexity though. Caching is harder. Error handling is different. File uploads need special handling. Start with REST and add GraphQL when you genuinely need its flexibility.
