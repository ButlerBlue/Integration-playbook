# Runbook: Moving a Server-to-Server Integration from Refresh Tokens to Client Credentials

*The order of operations for switching a vendor's back-end integration to the OAuth 2.0 Client Credentials flow, so nothing breaks mid-cutover and no old token outlives the change.*

## Why This Move Happens

Many server-to-server integrations started life on a flow built for people: someone logged in once, the vendor stored a refresh token, and the vendor's server has been trading that refresh token for access tokens ever since. That works, but it leaves a long-lived credential tied to a human login sitting in the vendor's database.

The Client Credentials flow fits the job better. The vendor's server authenticates as itself, gets a short-lived access token, and asks for a new one when it expires. No login screen, and no refresh token to store or leak. Salesforce documents that this flow does not support refresh tokens, and the OAuth 2.0 specification (RFC 6749, section 4.4.3) says a refresh token should not be included in a client credentials response.

The flow itself is simple. The cutover is where things go wrong, because the old tokens, the new flow, and the shared client secret are all live at the same time.

## Before You Start

1. **Work in a sandbox first.** Every step below runs in sandbox, gets verified, and only then repeats in production.
2. **Set up the integration user and its permissions.** The flow runs as a designated integration user. Salesforce requires you to specify that user on the app. See [Configuring a Salesforce API-Only Integration User](04-api-only-user-setup.md).
3. **Use an External Client App** to hold the OAuth policies, scopes, and the run-as user. Never fall back to a legacy app type for a new configuration.
4. **Agree the sequence with the vendor in writing,** including who tells whom when each step is done. The secret rotation in Step 5 depends on it.

## Step 1: Enable the Flow and Test the Response

Enable Client Credentials on the External Client App, set the run-as user, and have the vendor request a token from the sandbox.

A healthy response contains an `access_token`, an `instance_url`, a `token_type` of `Bearer`, and the granted `scope`. It contains **no** `refresh_token`. That absence is the flow working correctly, not a missing feature.

If the vendor gets an error saying no client credentials user is enabled, the app has no run-as user set. Fix the app configuration rather than retrying the request. See [Debugging Authorization Code + PKCE Flows](15-debugging-oauth-pkce-flows.md) for the related license and login-screen failures.

## Step 2: Move Credentials Out of the URL

Some token endpoints will accept `client_id` and `client_secret` as query string parameters. Do not let a working test in that shape go to production.

Send the parameters as form fields in the POST body, with `Content-Type: application/x-www-form-urlencoded`. RFC 6749 (section 2.3.1) says client credentials must not be included in the request URI. The practical reason is logs: URLs are recorded by web servers, proxies, and monitoring tools, so a secret in a URL ends up stored in plain text in places nobody audits. The same rule applies to the revoke call in Step 4.

```
POST /services/oauth2/token
Content-Type: application/x-www-form-urlencoded

grant_type=client_credentials&client_id=[client id]&client_secret=[client secret]
```

## Step 3: Cut the Vendor Over Completely

The vendor switches every code path that calls your org to the new flow. Their existing refresh token calls keep working while this happens, which is what makes the cutover safe.

Ask the vendor to confirm in writing when nothing in their environment uses the old tokens anymore. Until then, do not revoke anything and do not rotate the secret.

## Step 4: Revoke the Old Tokens

Vendors often ask how to list their issued refresh tokens so they can revoke each one. You generally do not need the token values, and you usually cannot retrieve them from the platform after they are issued; only the client that received a token holds it.

Two ways to revoke:

| Approach | How | When to use it |
|---|---|---|
| **Everything one user holds for this app** | On the connecting user's record in Setup, use Revoke next to each of this app's rows in the OAuth Apps related list. | The normal case. Clears every token, including ones the vendor no longer has stored. |
| **Individual tokens** | POST to `/services/oauth2/revoke` with the form field `token=[token value]` in the body | When the vendor still holds specific tokens and wants to revoke them itself |

The revoke parameter is named `token`, not `refresh_token`. Revoking a refresh token also revokes the access tokens issued from it, so one call per refresh token is enough.

### Reading the List Before You Revoke

Start by confirming when the cutover actually happened, from evidence rather than memory. The user's login history in Salesforce records a Login Subtype for each sign-in, with values such as OAuth Refresh Token and OAuth Client Credentials. The last refresh token login and the first client credentials login give you the date. Anything that still authorized the old way after that date needs a look before you revoke.

Each row in a user's OAuth Apps list is one approval. Four columns tell you whether it's still live:

- **Application.** An org can hold two apps for the same vendor with similar names, such as the one in use today and an older one from an earlier setup. Check the Application column in the user's login history to see which app recent logins actually came through, and judge every row against that.
- **Last Used.** Rows unused for months are leftovers. Under Salesforce's 30-day idle limit they have most likely expired already, and revoking just clears them off the list.
- **Use Count.** A count of zero is a sign-in that was never used, usually a retry during setup. Revoke it.
- **Created Date.** When several rows look alike, the one created and used most recently is normally the live one.

Check every user the vendor touched, not just the integration user. The vendor's own admin login collects tokens from developer tools during a build, such as the Salesforce CLI and Workbench. Keep the one their tooling uses today and revoke the rest; a revoked CLI login is fixed by signing in again. If a test user existed only to hold refresh tokens for the old flow, revoke every row on it once the vendor confirms no environment still uses it, then deactivate the user. Salesforce users cannot be deleted, but deactivating ends the user's access and keeps its history, and you can reactivate it later if you need a user to test against. Once an older app has no live rows on any user, retire the app itself too (see Step 5 of [Leading an Integration Fleet Through a Platform OAuth Enforcement Change](13-platform-oauth-enforcement-fleet-migration.md)), so only one app is left to reason about.

## Step 5: Rotate the Client Secret Last

Rotating the secret is good hygiene after a cutover, especially if the old secret ever appeared in a URL. Do it **after** Step 4, not before.

If the vendor's old refresh token calls authenticate with the same secret, rotating it early breaks them the moment it changes, in the middle of the cutover. Confirm in your org whether refresh requests for this app require the secret; plan as if they do.

The new connection uses the same secret, so rotating it also interrupts Client Credentials until the vendor updates its stored value. Agree a time with the vendor, rotate, and deliver the new secret straight away, so the gap is minutes rather than however long it takes them to notice.

Deliver the new secret through the vendor's secure exchange channel. Never by email.

## Step 6: Verify with the Final Permissions

1. **Run the vendor's real operations,** for example an export and a sign-in, with the final permission set assigned.
2. **Remove any interim or broader permission set** that was in place during the build.
3. **Run the same operations again.** If they still pass, the final permission set is complete on its own.
4. **Record the result:** flow, run-as user, permission set, test date, and who confirmed.

## Step 7: Repeat in Production

Run Steps 1 through 6 again in production, in the same order, with the vendor confirming each handoff. A sandbox success proves the sequence; it does not change production.

## Common Failure Modes

| What went wrong | Consequence | Prevented by |
|---|---|---|
| Treating the missing refresh token as a defect | Time lost chasing a problem that is the flow working correctly | Step 1 |
| Run-as user not set on the app | Token requests fail with an error that looks like a credential problem | Step 1 |
| Client secret sent in the URL in production | The secret is written to server and proxy logs | Step 2 |
| Old tokens revoked before the vendor finished switching | The vendor's integration goes down mid-cutover | Step 3 |
| Trying to list token values before revoking | Delay, and a request the platform usually cannot satisfy | Step 4 |
| Revoking by app name without checking which app recent logins used | The live connection is revoked along with the leftovers | Step 4 |
| Stale developer-tool tokens left on the vendor's admin login | Working credentials nobody is tracking | Step 4 |
| Old test user left active after the cutover | A second sign-in path stays open, and the next cutover starts with a confusing list | Step 4 |
| Secret rotated before old calls were retired | Remaining refresh calls fail the moment the secret changes | Step 5 |
| Secret rotated without warning the vendor | The new connection fails until the vendor notices and updates its stored secret | Step 5 |
| Interim permission set left in place | The integration runs with more access than it needs | Step 6 |

Related: [Retiring the OAuth Implicit Flow](12-retiring-oauth-implicit-flow.md), [Third-Party Integrator Security Playbook](03-integrator-security-playbook.md), and [Vendor and Integration Offboarding](05-vendor-offboarding.md) for revoking access when the relationship ends.
