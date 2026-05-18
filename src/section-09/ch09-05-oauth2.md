# OAuth 2.0

I remember the first time a user asked me to add "Sign in with Google" to my app. I had no idea how it worked. It felt like magic. Then I learned about OAuth 2.0, and it made sense.

## Why OAuth Exists

Before OAuth, I asked users for their Google username and password so I could access their data. That is terrifying. I should never have access to someone else's password. OAuth was created to solve this exact problem: letting one app access another app's data without ever seeing the user's password.

With OAuth, the user authenticates directly with Google. Google then gives my app a token that grants limited access. My app never sees the password. The user can revoke access at any time from their Google settings.

## The Authorization Code Flow

This is the most common OAuth 2.0 flow for server-side apps. Here is how it works:

```
User        My App        Google
  |            |             |
  |--- "Sign in with Google" -->|
  |            |             |
  |<-- Redirect to Google -----|
  |                          |
  |--- User logs in at Google -->|
  |                          |
  |<-- Redirect back with code --|
  |            |             |
  |            |--- Exchange code for token -->|
  |            |<-- Access token ------------|
  |            |             |
  |            |--- Fetch user info with token -->|
  |            |<-- User profile ----------------|
  |            |             |
  |<-- Logged in!            |
```

Let me break down each step:

1. **User clicks "Sign in with Google"**: My app redirects to Google's authorization URL with my client ID and a redirect URI.

2. **User authenticates at Google**: The user logs in to Google and consents to sharing their info with my app.

3. **Google redirects back**: Google sends an authorization code to my redirect URI. This code is temporary and single-use.

4. **My app exchanges the code for a token**: My server sends the code, along with my client secret, to Google's token endpoint. Google returns an access token.

5. **My app uses the token**: I call Google's API with the access token to get the user's profile info.

## Why the Code Exchange?

You might wonder: why not just give my app the token directly? Why the extra step with the authorization code?

The authorization code is sent through the browser (via the redirect URL). The browser is not fully secure. Someone could intercept the code. But the code is useless without the client secret, which never touches the browser. My server exchanges the code plus the client secret for the token. This way, even if someone steals the code, they cannot get the token.

## The Key Terms

- **Client ID**: Identifies my app to Google (public, safe in the browser)
- **Client Secret**: Proves my app's identity to Google (private, server-side only)
- **Redirect URI**: Where Google sends the user after they authenticate
- **Authorization Code**: Temporary code exchanged for a token
- **Access Token**: Token used to call Google's API
- **Scope**: What permissions I am requesting (email, profile, etc.)

## Registering My App

Before I can use OAuth, I register my app with the provider (Google, GitHub, etc.). I get a client ID and client secret, and I register my redirect URI. Every provider has a developer console where I do this.

OAuth 2.0 is a standard, but each provider implements it slightly differently. The concepts are the same, but the URLs, token formats, and profile responses vary. In the next chapter, I will implement Google OAuth with Passport.js.
