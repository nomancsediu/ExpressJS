# Why Security Is Not Optional

I used to think security was something you add later. Ship the feature first, worry about security later. That mindset nearly cost me a project when someone dropped my entire database through a simple injection attack.

Security is not a feature. It is a foundation. Every Express app that goes on the internet will be probed, scanned, and attacked. Not maybe. Not eventually. Within hours of deployment. Bots crawl the web looking for vulnerable servers constantly.

This section covers the most important security topics for Express applications:

- **Helmet.js** for setting security headers automatically
- **CORS** for controlling who can access your API
- **CSRF protection** for preventing forged requests
- **XSS prevention** for stopping script injection
- **Injection prevention** for protecting your database
- **Validation and sanitization** for cleaning user input
- **Rate limiting** for preventing abuse
- **Security headers** for hardening your HTTP responses
- **Environment secrets** for keeping credentials safe
- **A security checklist** for deploying with confidence

The uncomfortable truth is that security is a moving target. New vulnerabilities are discovered all the time. What is safe today might not be safe tomorrow. But there is a big difference between an app with basic protections and one with none. Most attacks are automated and target known vulnerabilities. Basic protections stop 90% of them.

I am not a security expert. I am a developer who has made security mistakes and learned from them. This section shares what I have learned so you can avoid the same pitfalls.

Think of security like locking your doors. It will not stop a determined burglar, but it stops opportunistic attacks. And most attacks on the web are opportunistic.

Let us build safer apps together.
