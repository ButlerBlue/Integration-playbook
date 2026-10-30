# Debugging Authorization Code + PKCE: A Field Guide to the Errors You'll Actually See

*Four errors you'll hit testing a platform's native OAuth 2.0 Authorization Code flow with PKCE, decoded, and the one change to how you test it that makes most of them stop happening.*

## The Short Version

PKCE (Proof Key for Code Exchange) is now the default expectation for a public-facing Authorization Code flow, and most modern platforms enforce it on their native OAuth endpoints whether or not you asked for it. The errors it produces when something's off don't say "PKCE problem." They say things like `missing required code challenge`, `redirect_uri_mismatch`, or `invalid code verifier`, each of which reads like a different, unrelated bug.

They're not unrelated. All four errors below come from the same small set of moving parts: a verifier, a challenge derived from it, an exact-match redirect URL, and how your client sends its credentials. Get any one of them wrong and the platform tells you about a symptom, not the cause.

The fastest way out of all four is the same: stop hand-building the authorize and token requests, and let an OAuth-aware API client (this guide uses Insomnia, but the same logic applies to Postman or any client with a built-in OAuth 2.0 flow) own the verifier/challenge pairing and the request construction. Most of what follows exists because hand-built requests are where these errors actually come from.

## Why Hand-Building These Requests Goes Wrong

PKCE needs two values that have to survive two separate requests without drifting apart:

1. A `code_verifier`: a random string you generate.
2. A `code_challenge`: `base64url(SHA256(code_verifier))`, sent on the authorize request.

The authorization server stores the challenge against the code it issues. When you exchange that code for a token, you send the verifier, and the server recomputes the hash to confirm it matches. If the verifier used at token exchange isn't the one that produced the challenge used at authorization, the exchange fails by design.

That's easy to get right with tooling and surprisingly easy to get wrong by hand: regenerate the pair between the two requests, reuse a stale authorization code from an earlier attempt, or let a copy-paste drop a character, and you get `invalid code verifier`. The other common hand-built mistake is simpler: pasting a header value or a query parameter into the URL field itself, which doesn't produce an OAuth error at all. It produces a platform error page that looks like the endpoint doesn't exist, because the request path is now wrong.

## Configuring the Flow in an OAuth-Aware API Client

Using Insomnia as the concrete example (the field names below are Insomnia's; the same settings exist in some form in Postman and similar tools):

1. Create a request. The method and URL in the main address bar don't matter for fetching tokens; that happens entirely from the Auth tab's own settings.
2. Open the Auth tab, set Auth Type to **OAuth 2.0**, Grant Type to **Authorization Code**.
3. Fill in:
   - Authorization URL: the platform's own authorize endpoint (for Salesforce: `https://[MyDomain].my.salesforce.com/services/oauth2/authorize`)
   - Access Token URL: the matching token endpoint (`https://[MyDomain].my.salesforce.com/services/oauth2/token`)
   - Client ID and Client Secret from your registered app
   - Redirect URL, matching the callback URL registered on the app exactly, byte for byte, including trailing slash and scheme
   - Scope, as configured on the app
4. Set **Credentials** to **In Request Body**. Many clients default to sending `client_id`/`client_secret` as an HTTP Basic Auth header. Salesforce's token endpoint (and several other platforms) expects them as body parameters instead; leaving the default in place produces an authentication failure even when PKCE and the redirect URL are both correct, and the error won't mention credentials at all.
5. Turn on **PKCE**, Code Challenge Method **S256**. Leave the verifier field blank. The client generates it.
6. Click **Fetch Tokens**. The client opens the consent screen in a browser; once you authorize, it captures the code and exchanges it for a token automatically, sending the matching verifier with that exchange. You never see or type either value.
7. Confirm an `access_token` comes back, then reuse it as a Bearer token on other requests in the same collection.

PKCE, the credentials setting, and an exact redirect URL match all have to be correct at the same time, and none of them fails with an error that names it. Work through the table below by the exact error text rather than assuming you already know which of the three is wrong.

## When the Connecting User Can't Reach a Login Screen: Client Credentials Flow

An integration user on a license or role that can't reach an interactive login screen (Salesforce's API-only integration user license is the common case; see [Configuring a Salesforce API-Only Integration User](04-api-only-user-setup.md)) will never complete the flow above, no matter how correctly PKCE is configured. The fix is a different grant type, not a retry: Client Credentials Flow skips the authorize step and the login screen entirely.

In an OAuth-aware client:

1. Auth tab > OAuth 2.0 > Grant Type: **Client Credentials**.
2. Access Token URL: the platform's own tenant-specific host, never a generic shared login host. There's no user in this request for a generic gateway to route on, so a shared host can't determine which org or tenant to authenticate against. A platform returning an HTML "page not found" style error from what should be a JSON API, instead of a JSON error, is the signature of this exact mistake.
3. Client ID and Client Secret from your registered app. Set Credentials to In Request Body here as well.
4. Fetch Tokens. No browser, no consent screen, no PKCE involved.

On Salesforce specifically, this also requires a designated run-as user under the app's OAuth policies. A missing run-as user and a login-screen-incapable license restriction produce similar-looking failures, so confirm which one you're dealing with before concluding it's the license.

## The Errors, Decoded

| Error | What's Actually Wrong | Fix |
|---|---|---|
| `invalid_request` / `missing required code challenge` on the authorize call | PKCE is required by the app's OAuth policy and wasn't included in the request | Enable PKCE in your client's Auth tab (Code Challenge Method S256) instead of hand-building the authorize URL |
| An HTML "page not found" style error instead of a JSON response from the token endpoint | The request never reached the endpoint as a proper POST, often because a header or parameter was typed directly into the URL field, or a GET was sent where a POST was required | Use your client's OAuth 2.0 helper instead of a hand-built request; if testing manually, keep the URL field to just the endpoint, headers in the headers section, and parameters in the body |
| `redirect_uri_mismatch` | The redirect URL in your client doesn't exactly match a registered callback URL on the app; trailing slash, http vs. https, case, or the platform normalized the value on save | Copy the registered callback URL directly out of the app's settings rather than retyping it; check whether more than one callback URL is registered |
| `invalid_grant` / `invalid code verifier` | The verifier sent at token exchange doesn't match the challenge used when the authorization code was issued, usually from reusing a stale code or regenerating the PKCE pair between the two requests | Let your client manage the verifier/challenge pairing automatically instead of running the authorize and token steps as separate manual requests |
| Authentication fails even though PKCE and the redirect URL are both confirmed correct | Client credentials are being sent as a Basic Auth header instead of in the request body | Set your client's Credentials option to In Request Body |

## Checklist Before You Call It Broken

- You're testing the platform's own native OAuth endpoint, not a separate custom identity layer your organization might also run in front of it. If there's a separate SSO system involved, confirm which one actually produced the error before troubleshooting PKCE.
- PKCE is enabled in your client, with Code Challenge Method S256, not typed in by hand.
- Credentials are set to send in the request body, not as a Basic Auth header.
- The redirect URL was copied from the app's registered callback URL, not retyped.
- The authorization code being exchanged is fresh, from the same attempt as the verifier being sent, not reused from an earlier test.
- If the connecting user can't complete an interactive login at all, you've switched to Client Credentials Flow rather than debugging PKCE on a flow that was never going to work for that user.

## Pairs with

- [Configuring a Salesforce API-Only Integration User](04-api-only-user-setup.md)
- [Retiring the OAuth Implicit Flow](12-retiring-oauth-implicit-flow.md)
