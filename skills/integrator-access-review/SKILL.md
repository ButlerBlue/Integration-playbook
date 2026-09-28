---
name: integrator-access-review
description: "Review a third-party vendor or integrator's request for CRM or API access (a scoping document, an email, a ticket, or a list of requested permissions) against least-privilege and security gate checks, then return a clear verdict and a reply the user can send. Use when asked to 'review this access request', 'check this vendor's scope', 'can we give them this', or whenever an integrator asks for permissions, credentials, OAuth setup, or production access."
---

# Integrator Access Review

Review a third-party integrator's access request before anything is provisioned, and give the user a verdict they can act on plus a reply they can send. The standard behind this skill is the [Third-Party Integrator Security Playbook](../../docs/03-integrator-security-playbook.md). Read it if it's available.

The goal is consistency. The same request should get the same answer every time, from anyone on the team, so integrators learn the standards once instead of negotiating them project by project.

## Step 1: Collect what's being asked for

From the request, extract:

- Who is asking, and on behalf of which client
- Whether written client approval is on file (if it isn't mentioned, treat it as missing)
- Every object and field requested, with read or write access
- The OAuth flow requested, callback URIs, and whether the integrator has stable egress IPs
- Which environments they want access to, and when
- How they expect to receive credentials
- Anything they're asking your team to build, configure, or test for them

If the request is vague ("we need access to member data"), that is the finding. A scope that can't be listed field by field can't be provisioned.

## Step 2: Run the gate checks

These are standards, not judgment calls. Any "fail" means the answer to that item is no, and it doesn't need escalating for a decision.

| # | Gate | Fails when |
|---|---|---|
| 1 | Client approval | No written approval from the client (not the vendor) on file |
| 2 | Defined scope | Objects or fields aren't listed individually, or read vs. write isn't specified |
| 3 | Least privilege | Admin-equivalent, "view all," or "modify all" access requested |
| 4 | No temporary elevation | "Just until go-live" elevated access requested |
| 5 | Secure credentials | Credentials requested by email, chat, ticket, or shared doc |
| 6 | Consent identity | OAuth consent to be performed with an administrator's login |
| 7 | Sandbox first | Production access requested before sandbox testing is signed off |
| 8 | Environment separation | Integrator's dev or staging environment pointed at production |
| 9 | Dedicated user | Integration would run as a shared or human user rather than its own connector user |
| 10 | Responsibility line | Your team is being asked to build, debug, or maintain the integrator's logic |

## Step 3: Review the judgment items

These aren't automatic no's, but each needs a stated position:

- **OAuth flow fit.** Unattended server-to-server syncs should use Client Credentials or JWT Bearer. If Authorization Code is requested for an unattended sync, flag it and explain the risk: refresh tokens used by multiple workers tend to pass single-process testing and then break in production in a way that looks like a platform failure.
- **IP restrictions.** If the integrator has no stable egress IPs, note which controls remain (callback URI allowlist, connector user permissions) and record that in the scope.
- **Identifiers.** If the integration involves SSO users, confirm it uses a permanent identity key for authentication and the CRM record ID for matching, never email. See [Identity Keys vs. Record Keys](../../docs/09-identity-vs-record-keys.md).
- **Volume.** If expected volumes are missing or large, flag the API limit risk and ask for estimates.
- **Meeting requests.** A call is fine as a scoped working session with an agenda sent in advance. An open-ended call isn't.

## Step 4: Return the review

Lead with the verdict, then the reasoning.

```
Verdict: [Ready to provision / Ready with changes / Not ready]

Blocking (must be resolved before provisioning):
1. [Gate #] [What's wrong] [What's needed instead]

Changes to agree before provisioning:
1. [Item] [Recommended position and why]

Approved as requested:
- [Items that pass]

Open questions for the integrator:
1. [Question]
```

Then draft the reply to the integrator:

- Warm, direct, and specific. Name each item that can't be approved and what you can offer instead.
- State standards as standards ("we don't grant temporary elevated access"), not as preferences or case-by-case decisions. An exception granted once becomes the expectation for every project after it.
- Point to the documentation for anything it already covers instead of re-explaining it.
- Never include credentials in the reply. If credentials are next, say they'll arrive through the secure channel.
- Close with the next step and who owns it.

## Escalation

Route by category, not relationship:

- Requests for security exceptions or elevated permissions go to implementation leadership, in writing. They don't get answered informally on a call.
- Requests to change core sync or qualification logic are escalated, never handled ad hoc.
- Pressure for early production access is a scope conversation, not a technical one.
- Scope added after sign-off goes back to the client as a formal change request.

## Don't

- Don't approve anything on the vendor's word that the client agreed. Approval comes from the client.
- Don't soften a failed gate into a "maybe." A clear no with a clear alternative is kinder than a maybe that turns into a no three weeks later.
- Don't describe the integrator as inexperienced, even when the request suggests it. The material is complex; say what's needed.
