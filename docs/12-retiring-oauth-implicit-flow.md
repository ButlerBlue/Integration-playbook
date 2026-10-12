# Retiring the OAuth Implicit Flow: Moving Integrators to Authorization Code with PKCE

*How to plan and communicate the move when your identity provider stops supporting the implicit flow, so every integrator migrates on time and nobody learns about it from a failed login.*

## Why This Move Happens

In the implicit flow, the identity provider returns a token directly in the redirect URL, in the fragment after the `#`. It was designed for browser apps at a time when they couldn't safely make a back-channel request. The cost is that the token passes through the browser's address bar, where it can end up in browser history, logs, and referrer headers.

The current OAuth 2.0 security guidance recommends against the implicit flow and points to the authorization code flow with PKCE (Proof Key for Code Exchange) instead. In that flow, the redirect carries only a short-lived, single-use code. The application exchanges the code for tokens through the token endpoint, proving it started the request by presenting a code verifier only it knows. The member's sign-in looks the same; the credential never travels through the URL.

The technical change is well understood. What makes this project hard is everything around it: finding every integration that uses the old flow, deciding exactly what is being retired, and getting dozens of vendors to change their code on a schedule they didn't choose.

## Step 1: Settle the Scope Before Anything Goes Out

The first question decides how many integrators are affected, so answer it in writing before drafting a single message.

1. **Name the exact `response_type` values being retired.** In OAuth 2.0, `response_type=token` is the implicit flow. In OpenID Connect, `response_type=id_token` and `response_type=id_token token` are implicit flows too. Retiring one and not the others can be the difference between contacting a handful of vendors and contacting all of them.
2. **Check what your own documentation teaches.** If your published quickstart lists an implicit `response_type` as required, says the code flow is out of scope, or states that PKCE is unsupported, assume most integrators built exactly what it described. That changes the size of the project and the tone of every message.
3. **Find every place the old token is reused.** The common one is logout: OpenID Connect logout requests often pass `id_token_hint`, populated from the ID token the app received in the fragment. With the code flow the app still receives an ID token, from the token endpoint instead, so the logout instructions need updating alongside the sign-in instructions.
4. **Confirm the token endpoint is ready and documented.** It may already exist and simply never have been documented for integrators. Updated documentation has to be published before the first vendor message goes out, or the message points vendors at a page that contradicts it.
5. **Decide on a dual-support window.** Run both flows side by side for a defined period so vendors can migrate and test without a hard cutover. Pick the dates before announcing them, because every message will repeat them.

## Step 2: Build the Integration Inventory

You can't migrate what you haven't found.

1. **Assemble the list from every source you have:** the identity provider's registered clients, ticket history, email, chat, and your CRM's vendor and client records. No single source is complete.
2. **Record one row per integration, not per vendor.** One vendor often serves several clients, sometimes with separate registrations.
3. **Capture, for each row:** vendor, client, application, registered client ID, redirect URIs, current flow (if known), technical contact, and last confirmed date.
4. **Add a second view grouped by client.** Client messages are personalized from it, and a wrong row there means telling a client you contacted a vendor you didn't.
5. **Reconcile against your CRM records** and log every mismatch. Vendors missing from the list, links attached to the wrong client, and contacts that are only a generic inbox all need a resolution path before the first send.

| Field | Why it matters |
|---|---|
| Client ID and redirect URIs | Proves which registrations are live; stale ones can be retired in the same project |
| Current flow | Separates "no change needed" from "migration required" |
| Technical contact | A generic inbox is not a contact; find a named person before the send |
| Client mapping | Drives the per-client message and the readiness tracking |

## Step 3: Sequence the Communication

Vendors first, clients second. If clients hear first, client-facing staff get technical questions they can't answer yet, and clients start contacting vendors directly with partial information.

| Order | Audience | Purpose | Owner |
|---|---|---|---|
| 1 | Vendor technical contacts | Confirm current flow, effort, and timeline; refresh contact and redirect URI records | Implementation or integration team |
| 2 | Clients | Explain what's changing in plain language, list the vendors already contacted, and ask what's missing from the list | Client-facing staff, using a shared template |
| 3 | Readiness tracking | Track each integration to confirmed-migrated, and flag anything at risk early | Implementation or integration team |

## Step 4: Write the Vendor Message

The vendor message has to do four things: explain the change, give a reason that holds up, ask a specific question, and respect the vendor's time. If your documentation taught the implicit flow, say so plainly. A vendor who built what you told them to build shouldn't read the message as a correction.

> Subject: Action required: changes to how your application signs users in through [Identity Provider]
>
> Hi [Name],
>
> We're preparing an update to [Identity Provider], and because your application signs users in through it, there's a change coming that will need work on your side. I wanted to reach you early so you have time to plan rather than react.
>
> The change is to the authorization flow. Our documentation has directed integrators to the implicit flow, a request with `response_type=[value]` where the token comes back in the URL fragment. That flow has been deprecated across the industry, and the new version will no longer support it. The replacement is the authorization code flow with PKCE: your application receives a short-lived code and exchanges it for tokens at our token endpoint, using a code verifier only your application knows. This keeps tokens out of browser history, server logs, and referrer headers.
>
> To be clear: if you built on the implicit flow, you built it the way our documentation described. We're not asking you to correct a mistake. We're asking you to move to a path we can now offer. Updated documentation is at [link].
>
> Could you confirm which flow your application uses today, and whether you have any other integrations with us beyond the one we have on file for [Client]? If you're already on the authorization code flow, no change is needed and I'll mark you as ready. If you're on the implicit flow, a rough sense of the effort will help us make sure the timeline works for you.
>
> Both flows will be supported until [date], and we won't switch anything off without written confirmation from you first.
>
> While I have you, could you also confirm that [contact] is still the right technical contact and that your registered redirect URIs are current?
>
> If it would help to talk it through, I'm happy to set up a scoped working session. Send a few times that work and I'll send an agenda ahead of it.
>
> [Signature]

## Step 5: Write the Client Message

Clients need to know three things: nothing changes for their users, their vendors are being handled, and what you need from them. Personalize the vendor list for each client from the inventory.

> Hi [Name],
>
> We wanted to give you a heads up about an upgrade to [Identity Provider], the sign-in service your users rely on.
>
> For your users, nothing changes. Same account, same email address, same password, nothing to do.
>
> What's changing is behind the scenes. When a user clicks through to another application, the two systems perform a quick handshake to confirm who they are. We're moving that handshake to a newer, more secure method that's now the industry standard, and each vendor will make a small update on their side to match.
>
> These are the applications we have on file for you, and we've already contacted each vendor directly:
>
> - [Application], [purpose], built by [Vendor]
> - [Application], [purpose], built by [Vendor]
>
> You don't need to chase any of them. Two things would help us:
>
> - **If your users sign in to anything not on that list, please tell us.** That's the most useful thing you can send back, including anything added recently or used by only one team or location.
> - **Tell us if you route sign-ins through another identity service,** such as a separate single sign-on provider your office already uses. If you do, that connection needs to be reviewed as part of this work. Not sure? Just say so and we'll check.
>
> The change is planned for [date], and we won't switch anything over until we've confirmed your vendors are ready.
>
> [Signature]

Give client-facing staff a short internal note alongside the template: post it by a set date, don't contact vendors directly, and send any unlisted application or technical question straight to the integration team rather than answering it.

## Step 6: Track to Cutover

1. **Track each integration to one of four states:** not contacted, contacted, migrating, confirmed migrated. "Contacted" is not "ready."
2. **Watch the old flow's traffic** during the dual-support window. Anything still using it close to cutover is either an integration missing from your inventory or a vendor who hasn't finished.
3. **Escalate at-risk integrations early,** against a threshold you define in advance, with the client's account owner in the loop.
4. **Test each migrated integration in a sandbox or test tenant first.** Production cutover follows a confirmed test, never the other way around.
5. **Switch the old flow off only after every integration is confirmed** or the client has accepted the risk in writing.

## Common Failure Modes

| What went wrong | Consequence | Prevented by |
|---|---|---|
| Scope announced as `token` only when integrators use `id_token` | Most integrations break at cutover | Step 1 |
| Documentation updated after the vendor message | Vendors follow the old page; the message looks wrong | Step 1 |
| Logout not updated | Sign-in works, sign-out breaks | Step 1 |
| Inventory built from one source | Unknown integrations fail on cutover day | Step 2 |
| Clients told before vendors | Client staff field questions nobody can answer yet | Step 3 |
| Vendor message reads as blame | Slower responses, defensive conversations | Step 4 |
| "Contacted" treated as "ready" | Surprise failures at cutover | Step 6 |

Related: [Third-Party Integrator Security Playbook](03-integrator-security-playbook.md) for OAuth flow selection, [Identity Keys vs. Record Keys](09-identity-vs-record-keys.md) for what the integrator does with the identity once it has it, and the [Scoped Working Session Agenda](../templates/scoped-working-session-agenda.md) for vendor calls.
