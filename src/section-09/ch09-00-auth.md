# Authentication and Authorization

If there is one topic I wish I had taken seriously from the start, it is authentication. I used to think "I will add auth later" and then later never came. Or worse, I added it poorly and left security holes everywhere.

Authentication is not optional for most apps. Any app with user accounts, private data, or admin panels needs it. And doing it wrong is worse than not doing it at all, because a broken auth system gives you a false sense of security.

Here is what I have learned the hard way:

- **Never store passwords in plain text.** This should be obvious, but I have seen it in production code. Always hash passwords.
- **Never roll your own crypto.** Use established libraries and algorithms. bcrypt for passwords, JWT or sessions for tokens.
- **Never trust client data.** Just because a request says "I am user 42" does not mean it is true. Verify everything server-side.
- **Always use HTTPS.** Without it, tokens and passwords travel in plain text across the network.

This section covers everything I know about auth in Express:

- The difference between authentication (who are you?) and authorization (what can you do?)
- Password hashing with bcrypt
- Token-based auth with JWT
- Session-based auth with express-session
- OAuth and social login with Passport.js
- Role-based access control
- API key authentication
- Rate limiting to prevent brute force attacks

By the end, you will have a complete auth system that handles login, registration, token management, and access control. Let me start with the fundamentals.
