# Vendor Change Requests in a Client Org: Decision Rights, Sandbox Parity, and the Production Exception

*How to handle a vendor who, in the middle of investigating a defect, asks to change a setting in a client's org or asks for a freshly refreshed sandbox: who decides, what the request has to contain, and why a stale sandbox is a governance problem, not just an inconvenience.*

## Why This Needs a Policy

A vendor working a defect in a client's org will eventually send one of two messages. The first is "we noticed this setting is configured a certain way; may we change it?" The second is "your sandbox doesn't match production, so we can't reproduce the issue; can you refresh it?"

Both sound like small, reasonable asks, and usually they are. The risk is in how they get answered when a client is waiting:

1. **Approval comes from whoever replied first,** not from the person who owns the client's data and configuration. Nobody can later say who decided.
2. **The change lands in production because the sandbox can't reproduce the problem,** and the sandbox-first rule quietly becomes "sandbox first when convenient."
3. **The sandbox request sits unowned,** because refreshing it is a decision with consequences and nobody is sure it is theirs to make.

This page sets the rules ahead of time, so the urgent version of the conversation is a lookup instead of a negotiation. It pairs with the escalation philosophy in the [Third-Party Integrator Security Playbook](03-integrator-security-playbook.md): route by category, not by relationship.

## Step 1: Classify the Request

Every vendor ask during a defect investigation falls into one of these categories. The category decides who answers it.

| Category | Examples | Who decides | Default answer |
|---|---|---|---|
| **Access to observe** | Read access to a sandbox, a debug log, a screenshot of a setting | Implementation contact, within the vendor's agreed scope | Yes, scoped and time-bound |
| **Configuration change in a sandbox** | Change a default value, a page layout assignment, or a permission in the sandbox | Implementation contact, with the client owner informed | Yes, once the change is described in writing |
| **Configuration change in production** | The same changes, in the live org | Client data owner, in writing | Only after it is confirmed in a sandbox, or by the explicit exception in Step 4 |
| **Code, package, or automation change** | A package upgrade, a trigger or flow change, a new field | Client data owner plus whoever owns the affected automation | Sandbox first, no exceptions |
| **Sandbox refresh or new sandbox** | "Please refresh the sandbox so we can reproduce" | Client data owner, after hearing what the refresh will discard | Decide within a day; never leave it unowned |

Two rules sit on top of the table:

- **The vendor never decides for itself,** however obvious the change looks from their side. They know their product; they do not know what else in the org depends on that setting.
- **The implementation contact routes, and does not absorb, the production decision.** Saying "yes" on the client's behalf because the client is anxious feels helpful and leaves you owning a change you did not have the authority to approve.

## Step 2: Require a Complete Request

Before anyone approves a configuration change, the vendor's request should answer five questions. If it doesn't, ask once for the missing pieces. This is the part that protects the client a month from now, when someone asks why a setting changed.

1. **What exactly changes,** named down to the object, the setting, and the profile or permission set it applies to.
2. **The current value and the proposed value,** so the change can be reversed precisely.
3. **Why the vendor believes it is related to the defect,** in a sentence or two. "It might help" is a reason to test it in a sandbox, not in production.
4. **Who and what else it affects.** A default value or assignment set at the profile level can reach every user on that profile, not just the integration, so confirm in your org where the setting lives and who it touches.
5. **How it will be reversed,** and who will do it, if it doesn't fix the problem or causes a new one.

The [Vendor Change Authorization](../templates/vendor-change-authorization.md) template turns these into a record you can paste into the support case or defect ticket.

## Step 3: Check Sandbox Parity Before You Answer

The sandbox-first rule only works if the sandbox can actually reproduce the problem. When a vendor says it can't, believe them and check why, because the answer decides what happens next.

| Sandbox state | What it means | Next move |
|---|---|---|
| **Matches production for the components involved** | The vendor can reproduce and test there | Test the change in the sandbox first, then take it to the client owner for production |
| **Missing the package version, components, or configuration involved** | The sandbox predates the change that introduced the defect | Raise a refresh decision with the client owner (Step 5) |
| **Has the components but not representative data** | Some sandbox types copy configuration only, or only a sample of data | Load or create a small, scrubbed data set that reproduces the case, rather than refreshing a larger sandbox |
| **Unknown** | Nobody has checked when it was last refreshed | Find out first. Sandbox age is a fact you should be able to answer in minutes |

Salesforce sandbox types differ in what they copy. Developer and Developer Pro sandboxes copy metadata and no production data, Partial Copy sandboxes copy a sample of data defined by a sandbox template, and Full sandboxes copy all production data unless a template narrows it. Each type also has a minimum interval between refreshes, from daily for Developer sandboxes to roughly monthly for Full sandboxes. Confirm the current figures for your edition in Salesforce's sandbox documentation before you promise a vendor a timeline.

## Step 4: The Production Exception, Narrowly Defined

Sometimes the client is losing something that matters, such as revenue, members, or a deadline, the sandbox can't reproduce the issue today, and the vendor has a specific, reversible configuration change they believe fixes it. That is the only situation where a production configuration change before sandbox confirmation is on the table, and it is still never "testing in production."

All of the following must be true:

1. **It is a configuration setting, not code, a package change, or automation.** Those stay sandbox first, without exception.
2. **The request is complete** (Step 2), including the current value and the reversal plan.
3. **The client data owner approves it in writing,** after hearing in plain language that it is going to production without a sandbox test, and why.
4. **Someone other than the vendor confirms the change afterward,** by checking the setting itself and the Setup Audit Trail entry for who changed it and when. Salesforce keeps Setup Audit Trail history for 180 days, so record the confirmation in the ticket rather than relying on the trail later.
5. **A sandbox refresh or rebuild is scheduled anyway,** so the fix can be reproduced and the root cause confirmed outside production.

If any one of these is missing, the answer is sandbox first, and the honest message to the client is about the timeline, not the rule.

## Step 5: Make the Refresh Decision Explicit

A refresh request should never sit waiting for someone to own it. Put the decision in front of the client data owner with the trade-offs stated plainly, and ask for an answer by a specific time.

What the owner needs to hear before they decide:

- **What will be lost.** Plan as if anything created only in the sandbox, including test records, in-progress configuration, and users added there for the vendor, will not survive the refresh. Confirm with the sandbox's admin what is in flight before approving.
- **What changes afterward.** Salesforce assigns a refreshed sandbox a new org ID, and copied users have `.invalid` appended to their email addresses. Any vendor setup that references the old org ID, and any user who needs sandbox email, needs updating after the refresh.
- **How long it will take,** including the refresh itself and the time to restore vendor access.
- **Whether a smaller option exists,** such as refreshing a Developer sandbox for a metadata-only reproduction instead of a Full sandbox the rest of the team depends on.

Once the owner decides, name who runs the refresh, who restores the vendor's access, and when the vendor will hear back. Then tell the vendor, even if the answer is "not until next week."

## Step 6: Treat Sandbox Parity as an Operating Standard

The urgent version of this page happens because a sandbox drifted months behind production and nobody noticed until a vendor needed it. Preventing that is a practice-level decision, not a ticket-level one.

1. **Keep a short register** of each client's sandboxes: type, last refresh date, purpose, and owner.
2. **Refresh, or deliberately decide not to, after significant production changes** such as a package upgrade or a new integration going live. Write the decision down either way.
3. **Review the register on a regular cadence** you set in advance, and flag any sandbox older than the last significant production change.
4. **Report it upward as risk, not housekeeping.** "Two clients cannot reproduce a production defect in a sandbox today; refresh decisions pending with their owners" is a sentence leadership can act on. See the [Weekly Integration Status Report](../templates/weekly-integration-status-report.md) for the format.

## Common Failure Modes

| What went wrong | Consequence | Prevented by |
|---|---|---|
| Vendor change approved by whoever answered first | No clear record of who decided; the wrong person owns the outcome | Step 1 |
| Request approved without the current value | The change can't be reversed precisely if it misfires | Step 2 |
| Change made in production because the sandbox was stale | Sandbox-first erodes into "when convenient" | Steps 3 and 4 |
| Production exception used for code or automation | An untested change in the live org with a wide blast radius | Step 4 |
| Vendor's change not independently confirmed | The org's state is assumed rather than known | Step 4 |
| Refresh request left unowned | The investigation stalls while the client keeps waiting | Step 5 |
| Refresh approved without knowing what was in flight | Someone else's sandbox work is wiped out | Step 5 |
| No sandbox register | The next urgent defect hits the same stale sandbox | Step 6 |

Related: [Governing Interim Workarounds](14-interim-workaround-governance.md) for keeping the fix moving while a client waits, and the [Defect Escalation Brief](../templates/defect-escalation-brief.md) for reporting the underlying defect so it is prioritized on the first read.
