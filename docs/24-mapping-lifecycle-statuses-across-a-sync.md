# Mapping Lifecycle Statuses Across a Sync: Exit Windows, Unmapped Values, and the Order of Fixes

*How to map a CRM's status values into a downstream system so every status change arrives, every arrival is understood, and a correction never leaves records stranded halfway.*

## Why This Needs a Standard

Many CRM-to-downstream syncs decide two things from a record's status. First, whether the record is sent at all. Second, what the downstream system does with it once it arrives: bill it, grant access, deactivate it, keep it for history.

A common design looks like this:

1. **A sync list.** Records whose status is on the list keep syncing.
2. **An exit window.** When a record moves off the list, the sync keeps sending it for a set period, so the downstream system hears the final status and can stop billing or remove access.
3. **A mapping table downstream.** Each source status value is translated into a downstream status with its own behavior.

Each part is reasonable on its own. The failures come from the gaps between them, and most of those gaps are silent: nothing errors, the record just ends up in the wrong state.

## Rule 1: Map Every Status, Not Only the Ones That Keep a Record Syncing

It is natural to build the mapping from the sync list, because those are the statuses that "sync." But the exit window means every status a record can move into gets sent **at least once.** A departure status that has no downstream mapping fails on arrival, or lands as an unhandled value, exactly when the downstream system most needs to hear it.

Build the mapping from the full list of source statuses, then mark which ones are on the sync list.

| Source status | On the sync list? | Downstream status | Downstream behavior | Variant for open obligations | Confirmed by |
|---|---|---|---|---|---|
| [Active status] | Yes | [Active] | Billed, access granted | Not needed | [Business owner] |
| [Departure status] | No | [Inactive] | Not billed, access removed | [Inactive, balance owed] | [Business owner] |
| [Transitional status] | [Yes or No] | [Decide explicitly] | [Decide explicitly] | [If needed] | [Business owner] |

## Rule 2: Give Departure Statuses a Variant for Open Obligations

A person leaving does not always mean the relationship is finished. If someone departs while still owing a balance, a single "inactive, not billed" mapping either stops statements they still need or keeps billing them for things they no longer receive.

Map a second downstream status for each departure status where obligations can remain open, such as "inactive, balance owed," set up so it continues statements for the existing balance only.

## Rule 3: Check That Transitional Statuses Map to the Behavior the Business Means

The costliest single mistake is a status the business treats as inactive that maps downstream to something billable. If that status is also on the sync list, the record is re-sent every run and billed every cycle. Over time it can even create downstream accounts for people who left long before the sync existed.

For every status that sits between active and departed ("requested to leave," "pending," "on hold"), ask the business owner one direct question: should this person be billed and have access, yes or no? Map to the answer, in writing.

## Rule 4: Backfill Records That Left Before the Sync Started

The exit window only covers changes the sync was running for. A record that moved to a departure status before go-live never got its window, so the downstream system kept whatever it had before.

When a sync is turned on for existing data, plan a one-time pass for records already in departure statuses. See [Runbook: Records Missing from an Incremental Sync](10-runbook-records-missing-from-sync.md) for the general backfill pattern.

## Rule 5: Correct a Status by Adding a Change, Not Deleting a Record

When a status was entered wrong, the instinct is to delete the wrong status record. Depending on how the sync detects changes, a delete may not count as a status change at all. Nothing starts the exit window, and the correction never goes downstream.

Confirm how your sync treats deletes. Unless you know a delete is handled, correct a status by adding a new status change with the right value, and write that into the guidance for whoever maintains the data.

## Rule 6: One Person, One Synced Role per Run

Roles often travel with status. If the downstream system accepts one record per person per run, and a person holds two qualifying roles at once, the sync may send two records for the same person. Without a rule that picks one, the downstream system can reject both, and nothing about that person updates until one role ends, including the role that was already working.

1. **Prefer a ranking rule** that picks the highest role when more than one qualifies, if your integration supports one.
2. **Without one, keep one qualifying role active per person,** and end the overlapping role in the source rather than overriding it downstream, where the next sync may overwrite the manual change.
3. **Run a report of people with more than one qualifying role** before predictable transition periods, such as annual leadership turnover. It catches the conflict before it blocks anyone's access.

## The Order of Fixes

When a mapping turns out to be wrong for records already in flight, the order matters more than any single step.

1. **Confirm the correct downstream status** with the business owner. Do not guess from a similar status.
2. **Fix the mapping** in every environment and every organization record where that mapping lives. A mapping fixed in one place and missed in another produces a split no one can explain later.
3. **Resend the affected records** with a no-change update, in sandbox first, respecting any data freeze. See [Runbook: Records Missing from an Incremental Sync](10-runbook-records-missing-from-sync.md).
4. **For records that no longer qualify,** use a temporary override to send them once, and note each one so it can be reverted.
5. **Verify downstream:** the new status on a sample of records, the sync history, the sync error report, and side effects such as open invoices the downstream system may void on its own when status changes. Decide what to do about anything already billed or paid.
6. **Revert every temporary override.**
7. **Only then remove a status from the sync list,** if the business agrees it should not keep syncing. Removing it earlier drops those records out of the sync with no exit window, and they stay stuck in the wrong downstream status.

Step 7 is the one that most often happens too early, because it looks like the real fix. It is the last step, not the first.

## Common Failure Modes

| What went wrong | Consequence | Prevented by |
|---|---|---|
| Mapping built only from the sync list | Departure statuses fail on arrival, at the moment they matter | Rule 1 |
| One departure mapping for everyone | Balances go unbilled, or departed people keep getting billed | Rule 2 |
| Inactive status mapped to a billable one | Repeated billing, and accounts created for people long gone | Rule 3 |
| No backfill at go-live | Long-departed records keep their old downstream status | Rule 4 |
| Wrong status deleted instead of corrected | The correction never reaches the downstream system | Rule 5 |
| Two qualifying roles for one person | Both records rejected; nothing updates for that person | Rule 6 |
| Status removed from the sync list before the resend | Records stranded with no exit window | The Order of Fixes |

Related: [Identity Keys vs. Record Keys](09-identity-vs-record-keys.md) for how records are matched once they arrive, and [Governing Interim Workarounds](14-interim-workaround-governance.md) for keeping temporary overrides on a clock.
