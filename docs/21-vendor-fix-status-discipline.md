# Status Discipline for Vendor Fixes: What Counts as Fixed, and How to Correct the Record

*How to report on a defect in a vendor's product so the client hears only what has been confirmed, and what to do the same day an earlier update turns out to be wrong.*

## Why This Needs a Standard

When a defect lives in a vendor's product, status passes through several hands before it reaches the client: the vendor's support team, your implementation lead, your client-facing staff, and finally the client. Every hop is a chance for a reasonable guess to become a statement of fact. "The upgrade should cover it" becomes "the upgrade fixed it," and the client is told the problem is resolved.

The cost shows up later and lands hard:

1. **The client sees the problem again** after being told it was fixed, and starts discounting every update that follows, including the accurate ones.
2. **The vendor stops feeling urgency,** because your own side has already announced the fix.
3. **The people affected downstream,** such as the client's customers or members, may have been told to take an action that does not work yet.

None of this needs anyone to be careless. It happens by default when nobody has agreed what "fixed" means. This page is that agreement.

## Step 1: Define What Counts as Fixed

Match every status statement to the evidence behind it.

| Evidence you have | Example | What you may tell the client |
|---|---|---|
| Vendor confirms in writing that a named release fixes this specific issue, and you have seen it work in the client's org | Written confirmation, the issue listed in release notes, and new records behaving correctly | "Resolved" |
| Vendor confirms in writing that the cause is identified and a fix is in development | A support case reply naming the bug and saying a fix is being built | "The vendor has confirmed the cause and is building a fix. We do not have a release date yet." |
| Vendor made a change and is still testing | A setting corrected in production, results pending | "A change was made on [date]. Testing is still in progress, and we will confirm once it is verified." |
| Someone on your side inferred it | "We upgraded, so it is probably fixed" | Nothing yet. This is a hypothesis to test, not a status. |
| Real records created since the change show the correct behavior | Records created after the upgrade have the right value on the field that was wrong | Strong supporting evidence. Pair it with the vendor's written confirmation before saying "resolved." |

Two rules sit on top of the table:

- **Release notes that do not list the issue are not confirmation,** however likely the fix seems.
- **Track related-looking problems as separate issues.** One release can fix a different defect with similar symptoms. Label them separately in every update (Problem 1, Problem 2) so a fix for one never closes both.

## Step 2: Write Every Update in Three Parts

Status that survives being forwarded has the same shape every time:

1. **What is confirmed,** and by whom, in writing.
2. **What is not confirmed yet,** said plainly.
3. **What happens next,** with an owner and the exact question that has been asked.

An internal example:

> What the vendor has confirmed in writing: the existing affected records cannot be converted, so those users will need to re-enter their details. What is not confirmed: whether new records are being created correctly since the upgrade; the vendor has not said that is fixed, and the release notes do not list it. Next: I have asked the vendor for a written yes or no on new records and a target date. [Name], can you check records created since [date] for the field that was wrong? That is the fastest real evidence we have.

## Step 3: Ask for the Fastest Real Evidence

Waiting on a vendor's answer is slow. Your own data is often faster.

1. **Check real records created since the change** for the specific field or attribute that was wrong. Name the field in the request so nobody checks the wrong thing.
2. **Ask the vendor narrow yes or no questions in writing,** for example: "Until the fix is released, will a new record created through [process] still have [the defect]?" and "Do you have a target date for the fix?"
3. **Put the questions in the vendor's case,** not only in chat, so the answer is on record where the next person will look.

## Step 4: Correct the Record the Same Day, Once, Without Blame

When an earlier update turns out to be wrong, the correction matters more than how it happened.

1. **Correct it the same day you learn it.** Every day a wrong status stands, more people act on it.
2. **Settle the facts internally, not who said what.** If the conversation turns to who meant what, redirect it: what matters now is where things stand and what the client was told.
3. **If you repeated the wrong status yourself, say so once,** and move on. One clear acknowledgment builds trust; repeating it undermines it.
4. **Send the correction through the person who owns the client relationship,** with the wording agreed internally first.
5. **Lead with the correction, then give the current confirmed status,** what the client does not need to do, and what comes next.

A client-facing correction:

> We need to correct an earlier update. The [release or change] fixed a separate issue, but [the vendor] has since confirmed that [the client's issue] is a different defect, and their fix for it is still in development. We do not have a release date yet and will share it as soon as we have one. Your existing [records] do not need to be recreated. [The vendor] has given us a way to [restore the affected items], and we will test it before we send you the steps.

Keep it short and calm. A correction that sounds alarmed reads as a bigger problem than it is.

## Step 5: Validate a Vendor Workaround Before Anyone Outside Uses It

Vendors often answer a defect with a workaround for the people already affected, such as sending each affected user a link to re-enter information. Before that reaches anyone outside your team:

1. **Separate what is yours to build from what the client sees.** If the workaround needs a page, a component, or a link configured in the org, that is setup work. Give the client the outcome and the timeline, not the setup steps.
2. **Run one complete case in a sandbox that matches production,** ideally in a short [scoped working session](../templates/scoped-working-session-agenda.md) with whoever on the vendor's team tested it. Walk the full path, then check the specific attribute that caused the defect on the resulting record.
3. **Ask whether new records still hit the defect until the fix ships.** If they do, the workaround is a bridge, not a resolution, and it needs an owner and an end date. See [Governing Interim Workarounds](14-interim-workaround-governance.md).
4. **Only then send it to the affected users,** through the client, with a clear note about what to do if it does not work.

Sending a link to a group of people and finding out it fails is far more expensive than one sandbox test, because every one of them now needs a second message.

## Step 6: Close It Out with Evidence

When the vendor says the fix has shipped:

1. **Confirm it in a sandbox, then in production,** against the same cases that were reported.
2. **Check new records for the attribute that was wrong,** not just that no one has complained.
3. **Tell the client "resolved," with what you checked,** in one or two sentences.
4. **Retire any workaround explicitly,** so nobody keeps running it after the fix.
5. **Update the defect record** with the release, the confirmation, and the date, for the next issue that looks similar.

## Common Failure Modes

| What went wrong | Consequence | Prevented by |
|---|---|---|
| "Fixed" reported from an upgrade alone | The client sees the defect again and discounts later updates | Step 1 |
| Two related problems tracked as one | A fix for one closes both, and the other resurfaces | Step 1 |
| Updates without a "not confirmed" line | Readers fill the gap with the most optimistic reading | Step 2 |
| Waiting on the vendor for evidence your own data already holds | Days lost, and a status nobody can verify | Step 3 |
| Correction delayed, or framed around who was wrong | More people act on the wrong status, and the team spends energy on blame | Step 4 |
| Workaround sent to affected users untested | A second round of messages, and lost confidence in the fix | Step 5 |
| "Resolved" declared because complaints stopped | A quiet defect left running | Step 6 |

Related: [Governing Interim Workarounds](14-interim-workaround-governance.md), [Vendor Change Requests in a Client Org](19-vendor-change-requests-in-client-orgs.md), the [Defect Escalation Brief](../templates/defect-escalation-brief.md) for reporting the defect itself, and the [Weekly Integration Status Report](../templates/weekly-integration-status-report.md) for carrying confirmed and unconfirmed status upward.
