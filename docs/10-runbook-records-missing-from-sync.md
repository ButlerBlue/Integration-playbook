# Runbook: Records Missing from an Incremental Sync, with No Errors

*What to do when a downstream system is missing a batch of records that clearly should have synced, and the sync logs show nothing wrong.*

## The Symptom

A client or partner reports that some records never arrived downstream: a few dozen, or a few hundred, often clustered around one group, region, or date range. Everything else is syncing normally. The error report is empty.

That last detail is the clue. **An empty error report doesn't mean the sync tried and succeeded. Often it means the sync never tried.**

## Why This Happens

Most CRM-to-downstream syncs are incremental. On each run, the sync picks up only records whose modification timestamp is newer than the last successful run, sometimes called the watermark. In Salesforce, that timestamp is typically `LastModifiedDate` or `SystemModstamp`.

So a record gets considered only if something touched it after the watermark. The common ways records fall through:

1. **They predate the sync.** The sync was turned on, or a new group was added to its scope, but the existing records weren't backfilled. Nobody has edited them since, so no run has ever had a reason to look at them.
2. **They newly qualify because of a formula.** If the sync's inclusion criteria depend on a formula field, a change in the formula's result doesn't update the record's timestamps. The record now qualifies, but nothing tells the sync to look.
3. **The qualification logic changed.** The criteria were widened, but only records edited after that change get evaluated under the new rules.
4. **The watermark moved past a failed batch.** Less common, and it usually does leave a trace in the logs, but worth ruling out.

Causes 1 through 3 produce no errors for the same reason: a record that's never attempted can't fail.

## Diagnose

1. **Get a concrete sample.** Ask for a handful of specific missing records by ID, not a description of the group. You need exact records to compare timestamps against.

2. **Compare their timestamps to the sync's history.**

   ```sql
   SELECT Id, Name, LastModifiedDate, SystemModstamp
   FROM Contact
   WHERE Id IN ('003XXXXXXXXXXXXXXX', '003YYYYYYYYYYYYYYY')
   ```

   If every missing record was last modified **before** the sync was enabled for that group (or before the last change to its qualification logic), cause 1 or 3 is confirmed.

3. **Check whether they actually qualify.** Evaluate each sample record against the sync's inclusion criteria as they exist today. If a record doesn't qualify, this isn't a timestamp problem. It's a data or criteria question, and it goes to whoever owns that logic.

4. **Check for formula-driven qualification.** If any inclusion criterion is a formula field, assume cause 2 is in play until proven otherwise.

5. **Write down what you confirmed and what you're inferring.** A clean-sounding root cause that was never verified sends the next person down the wrong path.

## Fix

The fix for causes 1 through 3 is to **touch** the affected records: run an update that changes no field values but moves their modification timestamps to now. In Salesforce, an update that writes back only the `Id` still updates `LastModifiedDate` and `SystemModstamp`, which puts the records in front of the next sync run.

Before you run it:

- **Do it in a sandbox first**, against the same automation, to see what fires.
- **Check what else listens for updates.** Triggers, flows, and other integrations watching the same object will all see these as real edits. A "no-op" update can still send emails, recalculate values, or wake up a second sync.
- **Respect data freezes.** If a migration or ETL window is in progress, wait. A single "harmless" update during a freeze can invalidate the load everyone else is validating.
- **Batch it.** Use a bulk tool or the Bulk API in reasonable batch sizes, not thousands of single-record calls against a shared daily API limit.
- **Exclude anything being handled by hand.** If some records in the set are being corrected manually, leave them out so the touch doesn't overwrite that work downstream.

## What to Expect

Allow at least one full sync cycle, then re-check the sample records downstream. Anything still missing after that is almost certainly not a watermark issue; go back to step 3 of Diagnose.

Communicate the timing up front: "Give it until [time], then check again, and send over anything still missing." That's more useful than "it should be fixed now."

## Prevent It Next Time

- **Plan a backfill whenever a sync is turned on for existing data,** or whenever its scope is widened. The watermark only covers the future.
- **Plan a backfill whenever qualification logic changes,** especially logic built on formula fields.
- **Reconcile counts on a schedule.** Compare "records that qualify at the source" with "records present downstream." A gap that grows quietly is this exact failure mode, caught before a client finds it.
- **Define an escalation threshold in advance** for how long a record can go without syncing before it becomes an incident instead of a wait-and-see. Decide it once, write it down, and hold it.

Related: [Responding to an API Limit Alert](07-api-limit-alert-response.md), since a large touch or backfill is one of the easiest ways to trigger one.
