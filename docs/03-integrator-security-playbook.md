# Third-Party Integrator Security Playbook (Salesforce + SSO)

*A framework for provisioning access when a client-selected third-party developer needs to build an integration against your platform, without becoming their support desk or inheriting their security risk.*

## The Responsibility Line

Read this before provisioning anything. Most escalations on integrator projects aren't technical, they happen because provisioning was completed before scope was signed, or because someone answered a question that wasn't theirs to answer and set a precedent for the rest of the project.

The line that matters isn't "internal vs. external." It's whether your team can actually audit and patch the integrator's code. If you can't, you provision the environment and scope the access, and the integrator owns the build, the maintenance, and the debugging of their own logic.

**What you deliver:**
- A sandbox seeded with representative configuration data
- A dedicated, API-only connector user and a separate QA user for testing
- A custom profile and permission sets scoped to exactly what was approved, with field-level security set explicitly
- An OAuth-based external app configured for the agreed flow, credentials delivered over a secure exchange channel (never email or chat)
- Verification that every object/field in the scope document is reachable before handoff
- One scoped working session at provisioning, if needed, with an agenda circulated in advance
- Coordination of the production cutover against the integrator's launch plan

**What is explicitly NOT yours to own:**
- The integration itself: connector logic, data mapping, transformation, error handling, retry logic, scheduling
- Token generation (you issue the client ID/secret; they generate their own tokens)
- Their infrastructure, hosting, monitoring, or uptime
- Data accuracy on their end
- Their QA process (you provide the test path and a QA user; running their tests is theirs)
- Credential custody after handoff
- Debugging their connector logic (you confirm permissions are correct; you don't read their code)
- Consenting to an OAuth grant using any administrator identity, yours or the client's

## Gate Checks Before You Provision

These are standards, not case-by-case calls. If an integrator pushes on any of these, the answer is no and it doesn't need to escalate for a decision:

- Never grant broad "view/modify all data" or admin-equivalent access to an integrator
- Never grant temporary elevated permissions on a "remove it after go-live" basis
- Never send credentials by email, ticket, chat, or shared document, use a secure credential-exchange channel
- Never perform OAuth consent using an administrator login, yours or the client's
- Never point an integrator's dev/staging environment at production
- Never issue production access before sandbox testing is signed off

Before provisioning, confirm: written client approval is on file, the scoping document is finalized (every object, every field, read vs. write, expected volumes), the OAuth flow is agreed in advance, and you know whether the integrator has stable IPs to restrict access to (if not, document that your remaining controls are the callback URI allowlist and the connector user's permission set).

## Provisioning Approach

Work sequentially, each phase depends on the one before it. A scope that arrives after development starts means rework.

1. **Sandbox first, always.** Seed it with representative configuration, never hand a third party a copy of real member/customer data as a convenience. That's a client decision made in writing, not a default.
2. **External app / connected app configuration.** Split "what the app is" from "what it's allowed to do." Scope OAuth permissions to the minimum needed, enable only the flow you've agreed to, and leave PKCE enabled unless the integrator has a documented, tooling-based reason they can't implement it (PKCE only applies to authorization-code-style flows; it doesn't affect client-credentials or JWT-bearer flows, so leaving it on typically costs nothing).
3. **Profile and permission sets.** Clone from your platform's most restrictive integration-user template. Keep the profile minimal; put substantive access in permission sets, where it's easier to audit and revoke. Set field-level security explicitly on every field in scope rather than relying on defaults.
4. **Connector user (production-equivalent permissions, no interface access) and a separate QA user (sandbox-only, for generating test data).** One connector user per integrator, always, so you can attribute API usage and revoke access cleanly if something goes wrong. Never grant a QA user code-deployment privileges; creating test data doesn't require it.
5. **Test before handoff.** Run queries covering every object/field in scope as a permission-set-only user, not as an administrator, administrator testing proves nothing about what the integrator will actually experience. Confirm a token request against the sandbox environment actually succeeds using the agreed flow.
6. **Single, complete handoff message.** Sandbox details, both usernames, which permission sets map to which access, the agreed OAuth flow and what the integrator is responsible for implementing, links to your developer documentation, and the refresh/cutover cadence. Credentials referenced but delivered separately over your secure channel. Then stop, the next move is theirs.
7. **Production promotion only after signed-off sandbox testing and a launch plan from the integrator**, not from you. Schedule cutover outside any data-freeze or maintenance window, and confirm production callback URIs and credentials are separate from sandbox.

## OAuth Flow Selection (General Guidance)

| Flow | Use when | Integrator stores |
|---|---|---|
| Client Credentials | Server-to-server, no human in the loop | client ID, client secret |
| JWT Bearer | Server-to-server, higher assurance, no shared secret at rest | private key |
| Authorization Code (Web Server) | A real user authorizes access to their own data | client ID, secret, refresh token |

For unattended data syncs, push back on authorization code and be specific why: refresh tokens are often single-use per writer, and an integrator running multiple workers will pass a single-process test and then break in production in a way that looks like a platform failure. If authorization code truly is required, the consent should be performed by the dedicated connector identity, never an administrator login, since the resulting token inherits whoever consented.

## Handling Predictable Pushback

Set the expectation once, warmly, and hold it consistently. Answering the same question twice teaches that your standards are negotiable, and every project after this one inherits that expectation.

| They ask | Answer |
|---|---|
| Can you build/configure the integration for us? | No, provisioning is your scope; the build is theirs. |
| Can we get elevated/admin access temporarily? | No, not even "remove it after go-live." |
| Can you complete OAuth consent with your admin login? | No, consent happens as the connector identity. |
| Can you send credentials by email? | No, secure exchange channel only. |
| Can we test against production instead? | No, sandbox first, always. |
| Can we have a call to walk through it? | Yes, as a scoped session with an agenda, not an open-ended call. |

## Escalation Philosophy

Route by category, not by relationship: requests for elevated permissions or security exceptions go to implementation leadership, not answered informally on a call. Requests to modify core qualification/sync logic get escalated rather than handled ad hoc. Pressure for early production access is a scope conversation, not a technical one. Scope expansion after sign-off goes back to the client as a formal change request.

## Best Practices Summary

- One connector user per integrator; never reuse a human user's license for an integration
- Never assign an administrator-equivalent profile to an integration user
- Prefer OAuth over legacy password-based API authentication
- Cache tokens until expiry rather than requesting one per call
- Use bulk/incremental sync patterns appropriate to data volume rather than full re-pulls
- Handle authentication failures by refreshing once, then failing loudly, a retry loop against a revoked credential can trigger account lockouts
- Track your platform's API version deprecation schedule as a shared dependency, not a surprise
