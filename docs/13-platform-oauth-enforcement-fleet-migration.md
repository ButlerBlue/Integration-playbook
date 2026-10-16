# Leading an Integration Fleet Through a Platform OAuth Enforcement Change

*How to run the program when your CRM platform tightens the rules for how outside applications sign in, so every integration is moved on purpose before the enforcement date instead of discovered after it stops syncing.*

## Why This Needs a Program, Not a Ticket

Platform vendors periodically harden OAuth for third-party connections. In Salesforce, the recent pattern has several parts: refresh token rotation (every refresh returns a new token and invalidates the old one), revocation of refresh tokens that sit unused past an idle period, retirement of older sign-in flows such as the OAuth username-password flow, and the move from Connected Apps to External Client Apps as the supported app record for integrations.

Each change is simple on its own. What makes the work hard is that it lands on every integration at once, across vendors, clients, internal systems, and client-facing staff, with dates you did not choose. Treated as a ticket, it becomes a series of surprise outages. Treated as a program, it becomes a planned migration with an owner for every row.

The failure signature is also the reason to start early. **Nothing crashes and no alarm goes off.** Data quietly stops arriving, and the first sign is usually a client asking why something is missing, days or weeks later. The vendor then reports `invalid_grant` or an authentication failure, which looks like a bad password and almost never is.

## Step 1: Translate the Vendor Notice into Exposure Patterns

Before contacting anyone, sort integrations by how they sign in, because that decides what each enforcement change does to them.

| Sign-in pattern | Refresh token rotation | Idle token revocation | Older flow retirement | Recommended end state |
|---|---|---|---|---|
| Username, password, and security token (OAuth username-password flow or a legacy SOAP login) | Not applicable | Not applicable | **Stops working on the retirement date** | Client Credentials or JWT Bearer |
| Authorization code with a stored refresh token | **Breaks if the code reuses a refresh token**; presenting an old token can revoke the whole connection | **Breaks if the integration goes quiet** past the idle period; a person must re-authorize | Not applicable | Client Credentials or JWT Bearer for unattended syncs |
| Several services sharing one stored refresh token | **Breaks first.** Each service invalidates the others' tokens unless one component brokers and saves every rotated token | Breaks if all of them go quiet | Not applicable | One credential per integration, on JWT Bearer or Client Credentials |
| Client Credentials on a dedicated API-only integration user | Not affected (no refresh token) | Not affected | Not affected | Already there |
| JWT Bearer with a certificate | Not affected (no stored refresh token) | Not affected | Not affected | Already there |
| Manual report export and file upload by a person | Not affected (uses the person's own session, no stored credential) | Not affected | Not affected | No change required |

Two things to say plainly to anyone who asks:

1. **Saving rotated tokens correctly does not protect against idle revocation.** Only regular use does. Seasonal integrations that run a few months a year, annual processes, and paused clients are the highest-risk rows, because the first day they are needed again is the first day they fail.
2. **Manual file imports are not in scope.** People often confuse "integration" with "anything that moves data." If a person runs an export and uploads it by hand, there is no stored credential for the platform to revoke, and nothing about the change requires retiring that process.

## Step 2: Build One Inventory, One Row per Connection

1. **Collect from every source:** the app records in each org, the platform's login history (filter by login type to find password-based and token-based sign-ins), vendor records, ticket history, and what client-facing staff know about tools a client added on their own.
2. **Record one row per connection, not per vendor.** One vendor often connects to many orgs, sometimes with different sign-in methods in each.
3. **Capture for each row:** org, vendor or internal system, app record type (legacy Connected App or External Client App), sign-in flow, whether the credential is shared, run frequency and seasonality, technical contact, and owner.
4. **Mark every unconfirmed value as unconfirmed.** "As I understand it" is fine in a working list and dangerous in a leadership chart. Publish status externally only after the technical owner confirms it, so client-facing staff never have to walk a change back.

The doc on [Retiring the OAuth Implicit Flow](12-retiring-oauth-implicit-flow.md) has a fuller inventory method for vendor-facing changes; the same discipline applies here.

## Step 3: Assign Ownership Before the First Message

The program stalls when every group assumes another one has it. Settle the split in writing.

| Work | Owner | Not their job |
|---|---|---|
| Interpreting the vendor notice, the exposure matrix, and the target standard | Integration or implementation lead | Writing vendor code |
| Code changes to the token request and token storage | Each vendor, or the internal engineering team for internal systems | Deciding the org-side standard |
| New app records, integration users, permission sets, and credentials | Integration team | Changing the vendor's code |
| Credential delivery | Integration team, over a secure credential-exchange channel only | Email, chat, or tickets |
| Client awareness and roadmap conversations | Client-facing staff, using a shared brief | Contacting vendors or changing anything in the org |
| New builds started during the program | Implementation team, on the target standard from day one | Building anything that will need migrating next year |
| Prioritization of internal fixes against existing roadmap dates | Product and engineering leadership, on a written ask | Being asked informally in a meeting |

Vendor agreements often already say vendors are responsible for keeping their integrations current with platform changes. That is true and worth citing. It is also not enough on its own: a client who loses data blames the platform team first, so the integration team runs the program even where the code change belongs to someone else.

## Step 4: Set the Standard for Everything New

Freeze the target before you migrate anything, so no new build becomes next year's migration.

1. **Every new integration gets an External Client App,** never a Connected App.
2. **Server-to-server data integrations use the Client Credentials flow** on a dedicated API-only integration user, or JWT Bearer where higher assurance is needed. Neither stores a refresh token, so rotation and idle revocation do not apply. See the [OAuth flow guidance](03-integrator-security-playbook.md#oauth-flow-selection-general-guidance) and [Configuring an API-Only Integration User](04-api-only-user-setup.md).
3. **Hold credentials** for any vendor that asks for the username-password flow or for a person to authorize an unattended sync through a browser, and point them to your developer documentation first.
4. **Nobody completes a vendor's authorization with their own login,** least of all an administrator. The integration then runs as that person, with all of their access, and breaks the day their password or role changes.
5. **Keep single sign-on and data integration as two separate builds.** Sign-on is how people walk in the front door; the data integration is the loading dock. Platform OAuth changes to data connections usually do not touch the sign-on layer, and saying so early prevents a wave of unnecessary worry.

## Step 5: Migrate One Connection at a Time

The sequence below keeps every client running throughout.

1. **Inventory with the vendor.** Ask which orgs they connect to and exactly which flow each one uses today. Match the answer against your own app records.
2. **Sandbox first.** Set up one org the new way in a sandbox and have the vendor confirm a token request and a test record before anything touches production. If the platform enforces in sandboxes before production, use that gap: it is a free rehearsal.
3. **One org at a time in production, in an agreed window.** Create the External Client App and integration user, deliver credentials over the secure channel, let the vendor swap the token request, and confirm records are flowing.
4. **Leave the old connection live until the new one is confirmed,** so there is never a gap.
5. **Retire the old app record and connector user** for that org once everything checks out. A migration that leaves the old credential active has added risk, not removed it.
6. **Plan around the calendar.** Ask each vendor and client which weeks are off limits. A hard retirement date that lands in a client's busiest season means finishing well before it, not on it. Vendor code changes take weeks, not hours.

## Step 6: Escalate Internal Gaps Early, in Writing

Internal systems are usually the hardest rows, because their fixes compete with an existing roadmap. When one of them is exposed:

1. **Name the risk in plain terms,** with the mechanism in one paragraph a non-engineer can follow.
2. **Show it is not hypothetical.** If the same failure has already happened in a related system, say when and what it took to recover.
3. **List who is exposed** and mark it as your current understanding, with a request to correct it.
4. **Split the work by owner** and say what you are already doing on your side.
5. **Make numbered asks with names on them:** confirm the sign-in flow, open or accept the ticket, test in sandbox by a date, give a target date for clients, and confirm app record type per org.
6. **Ask leadership for one explicit decision:** whether to pull the fix ahead of its current roadmap date. A yes or no is a decision; silence is a risk nobody accepted.

## Step 7: Brief Client-Facing Staff Once, Well

Client-facing staff need a short brief they can use in roadmap conversations, not a technical document. Cover:

1. **Nothing is broken today,** and why you are raising it now.
2. **What it looks like when it hits a client:** missing data, discovered late.
3. **What they own:** awareness and the client conversation. What they do not own: contacting vendors or changing the org.
4. **Questions worth asking clients:** what is connected to their org that you may not know about, which integrations run only part of the year, whether a hard date lands in their busy season, and whether a paid custom integration may need re-scoping.
5. **Reassurance:** what members and end users will not notice.

Frame the vendor-facing message to match the real impact. If most vendors will not need to change anything, send it as an alert with a clear "no action needed if" line, not as a change notice that implies everyone has work to do. Overstating impact costs vendor goodwill and floods the team with questions.

## Step 8: Report Status to Leadership

A one-line status per row rolls up into a view leadership can read in a minute.

| State | Meaning |
|---|---|
| Not exposed | Already on the target standard, or not an API connection |
| Exposed, not contacted | Owner has not reached the vendor or internal team yet |
| Contacted | Vendor or team has acknowledged; no plan yet |
| Scheduled | Sandbox test and production window agreed |
| Migrated | New connection confirmed in production; old one retired |
| At risk | Will not finish before a hard date, against a threshold you define in advance |

"Contacted" is not "ready." Report the count in each state, name every at-risk row with its owner and next step, and bring a proposed path for each, not just the problem.

## Common Failure Modes

| What went wrong | Consequence | Prevented by |
|---|---|---|
| Treating rotation as the only change | Seasonal integrations die from idle revocation after the code was "fixed" | Step 1 |
| Several services sharing one refresh token | They revoke each other, and the whole connection goes down | Step 1 |
| Inventory from one source | Unknown integrations fail on enforcement day | Step 2 |
| Unconfirmed status published | Client-facing staff walk back what they told clients | Step 2 |
| No written ownership split | Each group waits on another until the date passes | Step 3 |
| New builds started on the old pattern | Next year's migration list grows during this one | Step 4 |
| Old credential left active after cutover | Risk added rather than removed | Step 5 |
| Internal fix left on its original roadmap date | The internal system becomes the outage | Step 6 |
| Every vendor told they must act | Lost goodwill and a flood of unnecessary questions | Step 7 |

Related: [Third-Party Integrator Security Playbook](03-integrator-security-playbook.md), [Configuring an API-Only Integration User](04-api-only-user-setup.md), [Vendor and Integration Offboarding](05-vendor-offboarding.md) for retiring old connections cleanly, and the [Scoped Working Session Agenda](../templates/scoped-working-session-agenda.md) for vendor cutover calls.
