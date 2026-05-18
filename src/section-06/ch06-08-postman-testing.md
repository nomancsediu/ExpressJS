# Testing with Postman

Writing an API is only half the job. You need to test it. Postman is the tool I use most for manual and automated API testing. It is free, powerful, and saves me a ton of time.

## Creating a Collection

A Postman collection is a group of related requests. I create one collection per API:

1. Open Postman and click "New Collection"
2. Name it "My API"
3. Add requests for each endpoint

Each request stores the method, URL, headers, and body. You can organize them into folders:

```
My API/
  ├── Users/
  │   ├── GET /users
  │   ├── POST /users
  │   ├── GET /users/:id
  │   ├── PUT /users/:id
  │   └── DELETE /users/:id
  └── Auth/
      ├── POST /login
      └── POST /register
```

## Using Variables

Hardcoding URLs and tokens everywhere is annoying. Postman environments let you define variables:

Create an environment called "Development":

```
baseUrl: http://localhost:3000
token: (leave empty, we will set it later)
```

Now use `{{baseUrl}}` in your requests instead of typing the full URL. When you switch to production, just change the environment.

## Test Scripts

Postman lets you write JavaScript that runs after each request. This is where automated testing happens:

```javascript
// Test script for GET /users
pm.test('Status code is 200', () => {
  pm.response.to.have.status(200);
});

pm.test('Response has data array', () => {
  const json = pm.response.json();
  pm.expect(json.data).to.be.an('array');
});

pm.test('Users have required fields', () => {
  const json = pm.response.json();
  json.data.forEach(user => {
    pm.expect(user).to.have.property('id');
    pm.expect(user).to.have.property('name');
    pm.expect(user).to.have.property('email');
  });
});
```

## Auto-Setting Auth Tokens

After logging in, save the token automatically:

```javascript
// Test script for POST /login
pm.test('Login successful', () => {
  pm.response.to.have.status(200);
});

const json = pm.response.json();
if (json.token) {
  pm.environment.set('token', json.token);
}
```

Now use `{{token}}` in the Authorization header of other requests. No more copying and pasting tokens manually.

## Pre-Request Scripts

Scripts that run before a request. Useful for generating dynamic data:

```javascript
// Pre-request script for POST /users
const randomName = 'user_' + Math.random().toString(36).substring(7);
pm.environment.set('randomName', randomName);
pm.environment.set('randomEmail', randomName + '@test.com');
```

Then in the request body:

```json
{
  "name": "{{randomName}}",
  "email": "{{randomEmail}}"
}
```

## Running Collections

Click "Run" on a collection to execute all requests in order. Postman shows a report of which tests passed and failed. This is great for regression testing after making changes.

## Newman: CLI Testing

Newman lets you run Postman collections from the command line, perfect for CI/CD:

```bash
npm install -g newman
newman run my-api-collection.json -e development-env.json
```

This runs all your tests without opening Postman. If any test fails, Newman exits with a non-zero code, which triggers a build failure in your CI pipeline.

## My Testing Workflow

1. Build an endpoint
2. Test it manually in Postman
3. Write test scripts for the happy path and error cases
4. Save the request to the collection
5. Run the full collection before deploying

This workflow catches bugs early and gives me confidence that my API works as expected. I used to skip testing and pay for it later. Now I test as I go and save hours of debugging.
