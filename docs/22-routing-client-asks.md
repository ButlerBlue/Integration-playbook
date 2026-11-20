# Routing a Client Ask Before It Becomes a Roadmap Request

*How to sort a client's "can the system do this?" into what it really is, so delivered features get found, configuration gets fixed by the people who own it, and only genuinely new needs reach the product roadmap.*

## Why This Needs a Practice

Client asks arrive in client language: "we need billed and collected totals by unit," "this person keeps losing access when roles change," "can the payer get a verification email?" The fastest response is often to forward the ask to the product team as a feature request. That feels responsive. It is frequently the slowest path to an answer.

When asks go to the roadmap unsorted, a few things happen by default:

1. **Features that already exist get requested again,** sometimes for the same client that asked for them last year. The product team spends time rediscovering what it built.
2. **Configuration problems wait in a backlog** when the fix was a mapping change someone could make this week.
3. **Unconfirmed answers reach the client** because nobody separated "what we know" from "what we need the vendor to confirm."
4. **Decisions land with the wrong person,** because the ask was routed by who heard it, not by who owns the outcome.

None of this is anyone's fault. It is what happens when every ask looks like a feature request. This page is the sort that happens first.

## Step 1: Check What Is Already Delivered

Before anything is filed, look at what the client already has. Ask two questions:

1. **Is there a live integration, report, or field that already carries this data or behavior for this client?** Check the client's own records, not the general documentation. An earlier project may have delivered most of what is being asked for under a different name.
2. **Was a similar ask closed before?** If an earlier request was closed as covered, the new ask may really be "surface this better" rather than "build something new."

Send the person who holds the ask to look at the existing feature first, with a specific place to check. That turns a vague request into a concrete comparison: what is there, and what is missing.

## Step 2: Classify the Ask

Every ask gets exactly one label before it moves.

| The ask turns out to be | Signal | Route it to | What the client hears |
|---|---|---|---|
| **Already delivered** | The data or behavior exists and is live for this client | The client-facing owner, with where to find it | Where the feature is and how to use it |
| **Available, but not in the version this client runs** | The capability exists in a newer version of the integration | The upgrade program, plus a documented workaround for now | The workaround, and that the capability is part of a planned upgrade path |
| **Configuration or mapping** | The system works as designed; a setting or value translation is wrong or missing | The team that owns that configuration | What will change and when to check |
| **Defect** | The system is not doing what it was built to do | Defect process, see the [Defect Escalation Brief](../templates/defect-escalation-brief.md) | Confirmed status only, see [Status Discipline for Vendor Fixes](21-vendor-fix-status-discipline.md) |
| **Unconfirmed vendor behavior** | The answer depends on how a vendor's product behaves internally | The vendor, with numbered questions | What is confirmed now, what is being confirmed, and interim guidance |
| **Someone else's decision** | The outcome sits in another team's decision rights | That decision owner | Nothing until the owner decides |
| **Genuinely new** | Nothing existing covers it, and it is not a defect | The product roadmap | That it has been submitted, without a date |

The second row is the one most often missed. If the capability exists in a newer version, it is not a roadmap request. Filing it as one asks the product team to build something twice.

## Step 3: Decide Who Files It

The person who holds the client relationship and the client's exact words files the request. Implementation advises on shape and classification, but does not file on someone else's behalf.

This matters for two reasons. The filer is the one who will be asked follow-up questions, so it should be the person who can answer them. And requests filed second-hand lose the client's actual need on the way.

## Step 4: Write the Request So It Can Be Acted On

A request the product team can act on names the shape of the data, not just the topic:

1. **The grain:** per record, per group, per organization.
2. **The period:** fiscal year, calendar month, a rolling window.
3. **The fields:** each value as its own field, not one combined total, if the client will want to report on them separately.
4. **What already exists,** from Step 1, and exactly what it does not cover.
5. **The client's own words,** quoted, so the product team can tell need from interpretation.

## Step 5: Answer the Client in Outcomes, Not Internal Names

Internal version numbers, project names, and system codes help your team route work. To a client, they read as tiers of service, and they invite the question "why don't we have the newer one?" before anyone is ready to answer it.

Describe what the client can do now, what the workaround is, and what the outcome of a future change would be. Keep version and project names internal until there is a plan the client can act on.

For unconfirmed vendor behavior, answer in three parts: what you can confirm now, what you have asked the vendor to confirm, and what the client's users should do in the meantime. Do not reassure on a point you have not confirmed. A reassurance you later walk back costs more than the wait.

## Step 6: When the Decision Belongs to Someone Else, Ask Them to Spell It Out

When an ask lands in another owner's decision rights, defer to them openly, then make their decision easy to execute. Ask the owner to state:

1. **Exactly which items** need action.
2. **Which account, environment, or credential** to use.
3. **Any deadline** they want it done by.
4. **When and where to escalate** if something blocks the work.

This keeps the decision with its owner while turning it into steps the doing team can complete without a second round of questions.

## Step 7: Make the Pattern Visible to Leadership

Count asks by classification each month. The counts tell a story individual tickets cannot:

- **Many "already delivered" asks** point to an enablement or documentation gap, not a product gap.
- **Many "available in a newer version" asks** are evidence for the business case to move clients to that version, with a documented cost of staying behind.
- **Many configuration asks for one client** point to a configuration review, not more one-off fixes.

Report the counts with one recommendation each. That turns support traffic into planning input.

## Common Failure Modes

| What went wrong | Consequence | Prevented by |
|---|---|---|
| Ask filed before checking what the client already has | Product rediscovers a delivered feature; weeks lost | Step 1 |
| Version gap filed as a roadmap request | Duplicate build work, and a request that never fits the backlog | Step 2 |
| Configuration fix sent to the roadmap | A one-day change waits behind feature work | Step 2 |
| Request filed second-hand | Follow-up questions go to someone who cannot answer them | Step 3 |
| Request names a topic, not a shape | Product builds the wrong grain or period | Step 4 |
| Internal version names shared with the client | Pressure for an upgrade nobody has scheduled | Step 5 |
| Unconfirmed vendor behavior stated as fact | A correction later, and lost trust in every update | Step 5 |
| Another team's decision made by whoever heard the ask | Rework, and friction over decision rights | Step 6 |

Related: [Partner Integration Intake](16-partner-integration-intake.md) for requests from partners rather than clients, [Governing Interim Workarounds](14-interim-workaround-governance.md) for the workaround that bridges a version gap, and the [Weekly Integration Status Report](../templates/weekly-integration-status-report.md) for carrying the counts upward.
