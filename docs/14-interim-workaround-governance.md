# Governing Interim Workarounds: Ownership, Exit Criteria, and Who Doesn't Inherit Them

*How to run a manual workaround while a platform fix is pending, so it stays a bridge with an end date instead of quietly becoming a permanent process someone else now owns.*

## Why This Needs a Policy

When an integration can't do something yet, the usual answer is a workaround: a manual import, a one-time data correction, a checkbox that excludes a record from a sync, a report someone runs by hand. That is often the right call. Clients keep working while the real fix moves through the roadmap.

The risk is what happens next. A workaround that works gets repeated. Once it has been repeated a few times, someone proposes handing it to the support team as a standard procedure. At that point three things have changed without anyone deciding them:

1. **The pressure to ship the real fix drops,** because the symptom has gone quiet.
2. **A team that didn't design the workaround now owns it,** often without the context to know when it is safe to run and when it isn't.
3. **The workaround starts to look like the product,** and clients begin to expect it.

None of that is anyone's fault. It is what happens by default when nobody writes the rules down. This page is the rules.

## Step 1: Confirm the Root Cause Before You Choose a Workaround

A workaround chosen before the root cause is confirmed tends to treat the symptom in one place while the same cause breaks something else.

1. **Trace one concrete case end to end,** from the record the client reported up through the integration logic. Write down what you confirmed and what you are inferring.
2. **Check whether other reports share the cause.** Two clients with "different" problems frequently turn out to have the same one. Saying so early changes the conversation from "two tickets" to "one platform gap with two known cases," which is what gets a fix prioritized.
3. **Confirm the gap is in the system, not the data.** If a record failed to sync because a required value was cleared in the source, the integration did exactly what it should. The fix is the data, not a workaround. See [Runbook: Records Missing from an Incremental Sync](10-runbook-records-missing-from-sync.md).
4. **Check whether the case needs a workaround at all.** Sometimes an existing control, such as a field that excludes one record from a sync, solves the specific case with no new process.

## Step 2: Classify the Workaround

Every workaround gets exactly one of these labels, in writing, when it is approved.

| Type | What it is | Who can run it | Ends when |
|---|---|---|---|
| **One-time cleanup** | A bounded correction for a known set of records or clients | Named owners only, usually the team that diagnosed the gap | The set is done, or the platform fix ships |
| **Interim bridge** | A repeating manual step while a fix is in the roadmap | Named owners only, with a documented runbook | The platform fix ships and is confirmed in production |
| **Standard procedure** | A supported, repeatable process with a runbook any trained team member can follow | The team that formally accepted it | Never, unless retired by a decision |

The default for anything created because a system can't do something yet is **one-time cleanup** or **interim bridge**, never standard procedure. Moving a workaround to standard procedure is a deliberate decision with a named approver, not something that happens because it has been done three times.

## Step 3: Write the Workaround Record

Keep one short record per workaround, linked from the defect or enhancement ticket for the real fix.

| Field | Why it matters |
|---|---|
| Root cause, in one paragraph | Anyone reading it later can tell whether the workaround still applies |
| Affected clients or record sets | Bounds the work and stops it spreading to cases it wasn't designed for |
| Type (from Step 2) | Sets who can run it and when it ends |
| Owners | Named people, not a team inbox |
| Linked fix ticket and current status | Keeps the real fix visible |
| Exit criteria | The specific event that ends the workaround |
| Known risks of running it | What breaks if it is run wrong, twice, or during a data freeze |
| Review date | When someone checks that it is still needed and still safe |

## Step 4: Hold the Ownership Line

The hardest moment is the polite request to hand the workaround to someone else. The table below is the answer most of the time.

| Request | Answer | Why |
|---|---|---|
| "Can support run this going forward?" | Not while it is a one-time cleanup or interim bridge | The diagnosing team knows when it is unsafe; support would inherit risk without context |
| "Can we offer this to other clients with a similar issue?" | Only after confirming the root cause matches | A workaround applied to the wrong cause hides a different defect |
| "Can we keep doing this so the fix can wait?" | That is a prioritization decision for product leadership, in writing | Silence on the fix is a risk nobody accepted |
| "Does the platform change mean we have to stop this?" | Check, then say so plainly | A manual export and upload is often unaffected by changes to automated connections, and stopping it would remove the only thing keeping a client whole |

Say no to the handoff warmly and once, and offer what you can: a clear status update, a date for the next review, and visibility into the fix ticket. Firm and gracious holds better than a reluctant yes that has to be unwound later.

## Step 5: Keep the Real Fix Moving

A workaround buys time. Spend it.

1. **Add every new confirmed case to the fix ticket,** with evidence. "Second confirmed case" carries more weight than a comment saying it happened again. The [Defect Escalation Brief](../templates/defect-escalation-brief.md) template covers the format.
2. **Name the open questions only the product owner can settle,** such as which rule takes precedence or which of two documented values is correct, and ask for them by name.
3. **Watch for events that raise the stakes.** A platform enforcement date, a new client about to hit the same gap, or a busy season approaching all justify asking to pull the fix forward. See [Leading an Integration Fleet Through a Platform OAuth Enforcement Change](13-platform-oauth-enforcement-fleet-migration.md) for an example.
4. **Report it with the cost visible.** "Two clients on a manual process, roughly [N] hours per cycle, fix in requirements but not resourced" is a sentence leadership can act on.

## Step 6: Close It Out

When the fix ships:

1. **Confirm it in a sandbox, then in production,** against the same cases the workaround covered.
2. **Run one reconciliation** between what the workaround produced and what the fixed integration produces, so nothing is duplicated or lost in the handover.
3. **Retire the workaround explicitly.** Mark the record closed, tell everyone who knew about it, and remove any scheduled reminders or saved reports that would invite someone to run it again.
4. **Note what you learned** for the next gap of the same shape.

## Common Failure Modes

| What went wrong | Consequence | Prevented by |
|---|---|---|
| Workaround chosen before the root cause was confirmed | Symptom treated in one place, same cause breaks elsewhere | Step 1 |
| Data problem treated as a system gap | A process built around a record that only needed correcting | Step 1 |
| No type assigned | Workaround drifts into a permanent process by repetition | Step 2 |
| Owned by a team, not named people | Nobody knows when it is safe to run | Step 3 |
| Handed to support informally | Support inherits risk without the context to manage it | Step 4 |
| Fix ticket left with one case and no questions answered | The fix never gets prioritized | Step 5 |
| Workaround left running after the fix ships | Duplicate or conflicting data | Step 6 |

Related: [Third-Party Integrator Security Playbook](03-integrator-security-playbook.md) for the escalation philosophy this page builds on, and [Runbook: Records Missing from an Incremental Sync](10-runbook-records-missing-from-sync.md) for telling a data problem from a system gap.
