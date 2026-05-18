# Request and Response Deep Dive

I used Express for months without really understanding the `req` and `res` objects. I knew the basics: `req.body`, `res.json()`, and that was about it. But these two objects carry so much more. The deeper I dig, the more I appreciate how much Express does for me.

The request object represents everything about what the client is asking for. The response object is how I send something back. Together, they are the core of every Express handler.

In this section, I want to go through both objects thoroughly. Not just listing properties and methods, but understanding when and why to use each one. Here is my plan:

- The request object: params, query, body, headers, and more
- The response object: all the ways to send data back
- Working with request headers and content negotiation
- Body parsing in depth: JSON, URL-encoded, raw, multipart
- Cookies: reading, setting, signing
- Sessions: stateful data on the server
- HTTP status codes and how to use them properly
- Content negotiation and response formatting
- File downloads and streaming responses

My goal is that after this section, I will never have to guess which property or method to use. I will just know. And if you are learning along with me, I hope this helps you get there too.

Let us start with the request object, since that is where every interaction begins.
