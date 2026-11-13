# Triaging a Platform Security Roadmap Against Your Integration Fleet

*How to turn a platform's security announcements into a short, owned list of the identities that will actually break, so the team prepares for the one change that matters instead of reacting to all of them or none of them.*

## Why This Needs a Practice, Not a Read-Through

CRM platforms now publish security changes as a roadmap: several articles, each with its own sandbox and production dates, updated separately as the dates move. Most items will not touch your integrations. One of them might, and it is rarely the one with the most alarming headline.

Getting this wrong costs you in either direction:

1. **Treat every item as urgent,** and you pull engineers, client-facing staff, and vendors into work that was never needed. The next time you raise a real risk, it carries less weight.
2. **Skim and move on,** and the change that does apply surfaces months later, usually when someone is re-authorizing a connection during an incident and the login simply does not complete.

The practice below is how an integration lead reads a roadmap once, carefully, and hands the team a list with owners and dates. It complements [Leading an Integration Fleet Through a Platform OAuth Enforcement Change](13-platform-oauth-enforcement-fleet-migration.md), which covers running a single large migration once you know you need one.

## Step 1: Reconcile the Sources Before You Size Anything

1. **Collect every source:** the roadmap page, its change log, and each linked article. Note the publish date and last-updated date of each.
2. **When two sources disagree on dates, find out which is newer.** A roadmap change log can move a date before the individual article is updated. A colleague who finds different dates is usually reading a different article, not reading it wrong.
3. **Record each item in one line:** what changes, who it applies to in the platform's own words, the sandbox date, the production date, and the source link.
4. **Use the platform's definitions, not intuition.** The same phrase can describe different populations in different announcements. In Salesforce, for example, the announcement for the upcoming block on direct login for single sign-on users without MFA defines an SSO sign-in to include sign-ins that use a SAML bearer or JWT bearer assertion. A server-to-server integration that never shows anyone a login page can still make its user an "SSO user" under that definition.

## Step 2: Ask One Question per Item

For each item, the question is not "is this bad?" but "which identities does this touch?" Send the list to the technical owner of each system and ask for an answer per item, including a plain "does not apply, because..." A good triage produces more "no" answers than "yes" answers, and the reasons are worth keeping.

| Kind of change | The question to ask | Where the answer usually lives |
|---|---|---|
| Retirement of an older sign-in flow | Does any connection still use this flow? | The platform's login history, filtered by login type, and the app records in each org |
| Token lifetime or idle revocation | Which connections can go quiet long enough to be revoked? | Your connection inventory, with run frequency and seasonality |
| Interactive login rules (MFA, single sign-on only, passkeys) | Which accounts does a person ever sign in as interactively, including to authorize a connector? | Your credential vault and the setup records of each connector |
| Permission or license model migrations | Does anything we run assign, depend on, or check this permission? | Packaged permission sets, provisioning scripts, and automation that checks permissions |
| App record changes | Which app records are on the type being retired? | App records per org (see [Retiring the OAuth Implicit Flow](12-retiring-oauth-implicit-flow.md) for an inventory method) |

## Step 3: Look for Shared Identities

The highest-risk pattern in any security roadmap is one platform user serving two different sign-in patterns. Each pattern is fine on its own. Together, a rule aimed at one of them breaks the other.

A concrete example, confirmed in Salesforce's own announcement for that change:

1. A JWT bearer sign-in counts as an SSO sign-in, and it sets a preference on the user that neither the user nor an admin can change.
2. Under the announced enforcement, a user with that preference and no registered Salesforce MFA method is blocked when Salesforce would otherwise prompt them to register one. The passkey registration screen does not appear, and the sign-in does not complete.
3. If the same service account is also the identity a data sync connector was authorized as through an ordinary interactive login, the next re-authorization of that connector fails.

To find these:

1. **List every user that an automated credential signs in as,** per org.
2. **List every user that a connector was authorized as interactively,** per org.
3. **Intersect the two lists.** Record one row per connection, not per user, because one user can sit behind several connections.

Expect the overlap in older setups. It usually dates from before dedicated per-integration users were the standard, when one admin-level service account did the setup login for everything in an org.

## Step 4: Size It with Evidence You Already Have

Before asking anyone to open Setup in dozens of orgs, shrink the list.

1. **Check whether the platform exposes the answer through the API.** In Salesforce, the `TwoFactorMethodsInfo` object records which verification methods each user has registered, and it can be queried. Querying it requires the Manage MFA in API user permission; without it, the query is refused with an "sObject type not supported" error. Granting that permission across many orgs can cost more than a manual check, so decide deliberately rather than by default.
2. **Use indirect evidence to triage, then confirm.** If your team's credential vault already stores a one-time-code seed on a login, MFA is very likely registered for that user. Treat that as a signal that moves the row to "confirm only," not as proof. Confirm on the user's record in Setup; check where your org's version shows registered verification methods.
3. **Split the list into three groups:** confirm only, needs work, and already inactive (connections that are no longer authorized only matter if someone re-authorizes them).
4. **Order by the earliest deadline.** Sandbox enforcement usually comes first, and sandbox connections are the ones most likely to have been set up informally, with logins nobody wrote down.

## Step 5: Choose the Fix That Removes the Cause

| Option | What it does | Choose it when | Watch for |
|---|---|---|---|
| **Move the connector to a dedicated user** that no automated credential ever signs in as | Removes the overlap, so the rule never applies to that connector again | The default for anything that needs work, and the only practical path when the old login's password is lost | Validation rules, triggers, or flows that exempt the old integration user by Id or profile will not exempt the new one. Check available integration licenses before creating users. See the license note in [Packaging a Partner Integration](18-packaging-integration-for-reuse.md) before assigning permission sets. |
| **Register MFA on the shared account** | Lets the shared account keep completing direct logins | The login is held, documented, and moving the connector is not possible yet | Use a method the whole team can use through the shared vault, such as an authenticator app seed stored on the login. A passkey on one person's laptop turns a vacation into an outage. |
| **Confirm and leave as is** | Nothing changes | The account already has a registered method | The overlap is still there. Put these on the list to move later, once the urgent rows are done. |

Be explicit about ownership of the fix. Service accounts your team created to run its own integrations are your team's to repair, not the client's. Saying so early keeps clients out of a fix they did not cause and could not make.

## Step 6: Assign Owners and a Finish Date Ahead of the Window

| Work | Owner | Not their job |
|---|---|---|
| Reading the roadmap, reconciling dates, and the exposure list | Integration or implementation lead | Making changes in client orgs |
| Pulling the list of affected connections | Owner of the system that holds the credentials | Deciding the remediation standard |
| Confirming registered methods on users in client orgs | Client-facing admins with access to those orgs, from a checklist split by owner | Choosing the fix |
| Team-owned and internal accounts | Integration lead | Waiting on client-facing staff |
| Moving connectors to dedicated users | Technical owner of the connector tooling, sandboxes first | Assigning owners across teams |
| Assigning an owner where access is split across people | Lead of the client-facing team | Doing the moves |

Set the finish date well before the sandbox window, not on it, and account for holidays and any data freeze. Sandbox moves are also your rehearsal: they are where the exemption rules and license limits from Step 5 show up cheaply.

**Be precise about urgency.** If the evidence suggests running connections keep syncing and only a re-authorization fails, say "this is readiness, not an outage," and label it as your reading until the platform confirms it. Overstating urgency burns credibility. Understating it leaves a re-authorization to fail in the middle of an incident.

## Step 7: Verify the Fix Stuck

A connector moved to a dedicated user can be moved back by anyone who later re-authorizes it under the old account, often while fixing something unrelated. If your tooling records "authorized as the new user" as a checkbox someone ticks, it is taking the step on trust.

1. **Compare the connector's actual authorizing user with the intended one** right after each move.
2. **Repeat the comparison on a schedule** until the enforcement date has passed in production.
3. **Ask for the check to be built into the tooling,** so drift is flagged automatically instead of found by the next audit.

## Step 8: Report It Upward in One Paragraph

Leadership needs the exposure, the decision, and the date, not the article. A format that survives being forwarded:

> **[Platform] security roadmap: our exposure.** We reviewed [number] items. [Number] do not affect us: [one short reason each]. One does: [the mechanism in one sentence]. It touches [number] connections across [number] orgs. [Number] need work; the rest need a confirmation only. Sandbox enforcement starts [date] and production [date]. Owners are assigned, and the target is to finish sandboxes by [date]. If we miss it, [what fails, and when someone would notice]. Decision needed: [none, or the specific decision and from whom].

## Common Failure Modes

| What went wrong | Consequence | Prevented by |
|---|---|---|
| Dates taken from one article without checking the change log | Work scheduled against a window that already moved | Step 1 |
| A phrase read by intuition instead of the platform's definition | The affected population is missed entirely | Step 1 |
| Every item treated as urgent | Lost credibility for the item that matters | Step 2 |
| Each sign-in pattern reviewed separately | The shared identity that breaks is never found | Step 3 |
| Manual checks across every org before shrinking the list | Weeks of effort spent on accounts that were already fine | Step 4 |
| MFA registered on a device only one person holds | The next re-authorization depends on one person being available | Step 5 |
| Clients asked to fix team-owned service accounts | Confusion, and a fix nobody on the client side can make | Step 5 |
| No owner where access is split | Rows sit untouched until the window opens | Step 6 |
| Moves recorded on trust | Connectors drift back to the shared account unnoticed | Step 7 |

Related: [Leading an Integration Fleet Through a Platform OAuth Enforcement Change](13-platform-oauth-enforcement-fleet-migration.md), [Third-Party Integrator Security Playbook](03-integrator-security-playbook.md), [Configuring an API-Only Integration User](04-api-only-user-setup.md), and [Runbook: Intermittent Authentication Failures Isolated to One Org](17-runbook-auth-failures-one-org.md).
