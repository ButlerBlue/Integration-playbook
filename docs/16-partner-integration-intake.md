# Partner Integration Intake: One Front Door and Clear Readiness Gates

*How to run intake for integration requests so every one arrives through the right door, is ready before anyone builds, and lands with the person who owns it, instead of being relayed through the client and rediscovered by the wrong team.*

## Why Intake Needs a Design

Most integration delays don't start in the build. They start in the first message. A few patterns come up again and again:

1. **The request is relayed through the client.** The client tells the partner they have "already been in touch," the partner reaches out on the client's behalf, and the request travels client to partner to platform team. Every hop drops context, and the request is easy to route to the wrong person internally.
2. **Someone new reaches out cold.** A partner contact nobody has worked with before emails about a client, outside the usual channel. Until the origin is confirmed, nobody knows whether the request is real, approved, or already in motion with someone else.
3. **The build starts before it is ready.** No signed agreement between the client and the partner, a target described as a season rather than a date, or no named technical contact on the partner's side. Work begun on those terms tends to stall halfway and then compete with work that was ready.
4. **A nonstandard request is treated as a standard one.** A request that looks familiar but differs in scope, for example one application serving several client organizations at once, gets a standard scope document and a data release before anyone has thought through what is different.

None of these are anyone's fault. They are what happens when intake is implicit. This page makes it explicit, so the team can hold the same line every time and leadership can see where requests are waiting and why.

## Step 1: Define One Front Door per Request Type

Each kind of request has exactly one entry point and one owner. Publish this table where client-facing staff and partners can see it.

| Request type | Front door | Who starts it | The client's role |
|---|---|---|---|
| Established partner building a new integration for a client | The partner's integration lead contacts the integration team lead directly, in the shared partner channel | The partner | Sign the agreement with the partner, then use and accept the finished tool |
| Client-selected vendor with no partner relationship | The client's request, with written approval, through the process in [Vendor Onboarding](01-vendor-onboarding.md) | The client | Approve the vendor in writing and own the vendor relationship |
| Custom application or independent developer | The parent organization, through the process in [Custom Application Requests](02-custom-application-requests.md) | The parent organization | Decide whether to sponsor the request at all |
| Anything that differs from the standard pattern in scope, data, or audience | Internal scoping review before any scope document or data goes out | The integration team lead | Answer scoping questions; wait for a decision before committing to dates |

The key decision is the first row: **partner integrations are coordinated between the partner and the integration team, not routed through the client.** The client's part is the commercial agreement and the finished product. Asking the client to manage the handoff between two technical teams adds a relay that helps nobody.

## Step 2: Hold Readiness Gates Before Anything Is Scheduled

A request can be acknowledged right away. It is scheduled only when every gate below is met. Each gate exists because skipping it produces a specific, predictable failure.

| Gate | What "met" looks like | If it is missing |
|---|---|---|
| **Agreement in place** | The partner confirms the client has signed with them | Work begun before signature is often paused or abandoned, and the setup has to be cleaned up or tracked indefinitely |
| **A real target date** | A calendar date the partner commits to, not a season or "as soon as possible" | Nothing to plan backward from, so the request can't be slotted against work that does have a date |
| **Named technical contact** | One person on the partner side who will build and test | Questions go to whoever replies first, and decisions get made twice |
| **Pattern classified** | Confirmed as the standard pattern, or flagged as nonstandard | A nonstandard build goes out with standard assumptions and a data release that doesn't match its scope |
| **Scope signed for nonstandard requests** | An [Integration Scope Document](../templates/integration-scope-document.md) reviewed internally and signed | Provisioning outruns the agreement about what the integration is allowed to do |

Work backward from the target date using your published standard implementation timeline. If the date leaves less time than that, say so on day one, not two weeks before the date.

When a partner's team is unavailable for a stretch, hold the request rather than starting your half alone. Half-built setups that sit idle are where stale credentials and forgotten access come from.

## Step 3: Handle Requests That Arrive Around the Front Door

Requests will still arrive the other way. The response is warm, specific, and the same every time.

1. **Acknowledge quickly and positively.** The person reaching out is trying to help their client.
2. **Confirm the origin before acting.** For a cold contact, check with the partner's known integration lead that the request is real and that this person is authorized to make it. Do not share credentials, data, or environment details until that is confirmed.
3. **Name the front door once, plainly, and explain why.** Not as a correction, but as the way to make the next request faster.
4. **Answer what you can.** State the next step on your side and the gates still open.
5. **Log it.** A one-off is a message. The same pattern from the same partner twice is a process gap worth raising with the partner's leadership, in writing.

A redirect message that holds the line without friction:

> Great news on [Client]; excited to get this moving.
>
> One thing I want to flag so future requests go smoothly: for new integrations, it works best when you (or whoever is leading it on your side) reach out to me directly here rather than through the client. When a request goes from the client to you to us, it is easy for it to land with the wrong person internally. Partner integrations are something our two teams coordinate directly. The client's part is signing the agreement with you and using the finished tool.
>
> Once they have signed with you, I will set up the Salesforce side, including the external app and your sandbox and production credentials through our secure exchange channel. From there your team finishes the build and lets me know when it is ready so I can enable it in production.
>
> Two things that would help me plan: have they signed with you yet, and what date are we building toward? Our standard implementation runs about [N] weeks end to end, and I want to get this on the roadmap with enough lead time.

For a cold contact from someone new at a known partner:

> Thanks for reaching out about [Client]. Before we get started, I want to make sure this lines up with what [Known Partner Lead] and I have in motion, so I am looping them in here. Once we have confirmed the scope and who is leading on your side, I will send next steps.

## Step 4: Settle the Division of Work at Kickoff

Write the split down once per request. It prevents both "I thought you had it" and a partner waiting on something the platform team never knew it owned.

| Work | Owner | Not their job |
|---|---|---|
| External app record, integration user, and permission set | Integration team | The partner's code |
| Sandbox and production credentials | Integration team, over a secure credential-exchange channel only | Email, chat, or tickets |
| Building and testing the integration | Partner | Deciding the org-side security standard |
| Telling the integration team the build is ready | Partner | Enabling it in production |
| Sandbox validation, then production enablement | Integration team | Changing the partner's build |
| Acceptance and day-to-day use | Client | Managing the handoff between the two technical teams |
| Awareness and roadmap conversations | Client-facing staff, using a shared summary | Contacting the partner about technical scope |

Sandbox always comes before production. A partner asking to "just test it in production" is a scoping conversation, not a shortcut.

## Step 5: Make Intake Visible to Leadership

Intake is where capacity problems show up first, so report it with the same discipline as the build work. A short weekly view is enough.

| Signal | Why it matters |
|---|---|
| Requests waiting on a readiness gate, by gate | Shows whether delays sit with the team or upstream, without anyone having to argue it |
| Requests that arrived around the front door | Tells you which partners need a conversation at their leadership level |
| Requests with a target date inside the standard timeline | Surfaces date risk while there is still time to reset expectations |
| Nonstandard requests waiting on internal scoping | Keeps decisions that need a leader from sitting in an individual's inbox |

Report counts and name the items that need a decision, with a proposed path for each. The [Weekly Integration Status Report](../templates/weekly-integration-status-report.md) template has a format.

## Common Failure Modes

| What went wrong | Consequence | Prevented by |
|---|---|---|
| Partner request relayed through the client | Request lands with the wrong person; context lost at each hop | Step 1 |
| Build started before the agreement was signed | Setup paused or abandoned halfway, with access left behind | Step 2 |
| Target given as a season, not a date | Request can't be prioritized against dated work and slips quietly | Step 2 |
| Nonstandard request handled with the standard scope document | Data released or access granted beyond what was actually agreed | Steps 1 and 2 |
| Cold contact acted on before origin was confirmed | Work duplicated, or access granted to someone not authorized | Step 3 |
| Division of work assumed, not written | Both sides wait on each other | Step 4 |
| Intake delays invisible above the team | Capacity looks like a performance problem instead of an upstream one | Step 5 |

Related: [Third-Party Integrator Security Playbook](03-integrator-security-playbook.md) for what happens once a request is ready to provision, and [Scoped Working Session Agenda](../templates/scoped-working-session-agenda.md) for turning a kickoff into a focused session.
