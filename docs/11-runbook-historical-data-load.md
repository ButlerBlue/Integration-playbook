# Runbook: Pre-Load Checks for a Historical Data Load from a Vendor

*What to verify before a vendor's history file goes into your CRM, so a load of tens of thousands of rows lands once, cleanly, and never has to be unwound.*

## Why This Needs a Runbook

A historical load usually arrives at the start or end of a vendor relationship: the vendor exports everything they hold, and your team loads it into the CRM so the history lives alongside the live integration. It looks like a one-time data task. It behaves like an integration, because the file has to agree with the live integration on every key, every format, and every parent record it points at.

Most of what goes wrong in these loads produces no error at all. A row that attaches to the wrong parent, a date stamped one day off, or a duplicate created next to a record the live integration already wrote all succeed quietly. The first sign is usually a client asking why someone's history looks wrong months later, and by then the load is hard to separate from everything written since.

The checks below are ordered so the cheapest questions get asked first and the vendor gets one complete list of questions, not a slow drip.

## Before You Start

- **Sandbox first.** Run the full load in a sandbox that has the same objects, fields, and automation as production. A load is a large burst of inserts and updates, and every trigger, flow, and downstream sync that listens on those objects will treat it as real activity.
- **Confirm there is no data freeze or ETL window** in progress for the target org. A load during a freeze invalidates whatever everyone else is validating.
- **Get the vendor's files into a controlled location** your team owns, and keep the originals untouched. Every cleaned or split file is a new file with a new name.

## Step 1: Decide Upsert or Insert

1. **Ask whether the file is a full export or a delta.** Vendor history files are almost always full exports, which means they include records the live integration has already written.
2. **If any rows could already exist, the load is an upsert, never an insert.** An insert of a full file duplicates every record the live integration created, and duplicates in a child object are painful to unwind because other records may already point at them.
3. **Identify the external ID field the upsert will key on.** In Salesforce, it must be a field marked as an External ID on the target object, and ideally marked Unique as well. If two existing records share a value, the upsert errors on that row rather than guessing, which is the behavior you want.
4. **Confirm the field is actually deployed on every target object** in the target org. Add-on fields shipped separately from a core package are the usual gap: present in one org, missing in another, and the upsert has nothing to key on without them.
5. **Note any object that has no external ID at all.** A load into that object can't be safely re-run, because a second attempt inserts everything again. That file has to be right the first time, so it gets the most scrutiny, not the least.

## Step 2: Check Field Names Against the Org, Not the Documentation

1. **Pull the real API names** from the target org (Object Manager, or a describe call) and build the file headers from that list.
2. **Watch for mixed namespaces.** When a managed package's objects carry a namespace prefix, fields added to those objects by a separate, unmanaged add-on or by the org itself do not. A header such as `pkg__Course_Enrollment__c.pkg__Vendor_Id__c` fails when the real field is `Vendor_Id__c`, and it fails on every row.
3. **Use relationship notation for parent lookups** when the parent also carries the vendor's ID. In Salesforce, a column headed `Course__r.Vendor_Id__c` lets the load resolve the parent by its external ID, so nobody has to translate vendor IDs into record IDs by hand.

## Step 3: Reconcile Parent Keys Before Loading Children

This is the check that catches the largest failures.

1. **List every distinct parent key the file references.** A child file with thousands of rows often references only a handful of parents.
2. **Query those parents in the target org:**

   ```sql
   SELECT Id, Name, Vendor_Id__c
   FROM Course__c
   WHERE Vendor_Id__c IN ('1001', '1001:1001', '1002:1005')
   ```

3. **Compare the format, not just the presence.** Vendors change their ID formats over time: a plain number becomes a versioned pair, a prefix gets added, leading zeros appear. Parent records written before the change keep the old format, and a fresh export sends the new one for everything. The two disagree, and every child row pointing at those parents fails to attach.
4. **Weigh each mismatch by row count.** One parent can carry most of the file. A mismatch on that parent isn't an edge case to load around; it's the blocker.
5. **Ask the vendor what their live integration writes today,** not what the export contains. If the live integration still sends the old format, map the file back to it. If it has moved to the new format, the existing parent records need updating first, and that update gets sequenced deliberately, in sandbox, before any child history loads.

6. **Confirm every referenced key resolves to exactly one parent.** Presence and format aren't enough. An external ID field that isn't marked Unique can hold the same value on two records, most often after a vendor resends or re-creates parent records in an org that already had them. When that happens, every child row pointing at the duplicated value fails on the upsert. Count instead of eyeballing:

   ```sql
   SELECT Vendor_Id__c, COUNT(Id)
   FROM Course__c
   WHERE Vendor_Id__c IN ('1001', '1001:1001', '1002:1005')
   GROUP BY Vendor_Id__c
   ```

   You want one row back per value you referenced, each with a count of 1. A value that doesn't come back is a missing parent. A count above 1 is a duplicate to resolve before any child loads.
7. **Look in the right direction.** A blank external ID on a parent record the file never references is harmless. The question that matters is whether every value the file does reference resolves cleanly, so scope the query to the file's distinct keys rather than scanning the whole object for blanks.

Apply the same check one object over. If a second child file references a different parent object, run the same query against that parent before releasing it.

## Step 4: Check Dates and Time Zones

1. **Ask the vendor whether timestamps are UTC or local,** and get the answer in writing.
2. **Check the target field types.** Loading a timestamp into a date-only field truncates the time portion. Records that fall in the overnight window where UTC and your local time zone sit on different calendar days will land one day off if the assumption is wrong.
3. **Count the affected rows** so the size of the risk is visible: "a few rows" and "a third of the file" call for different levels of caution.
4. **Confirm how your load tool interprets dates.** Some tools apply a configured time zone to date and datetime values; check that setting in sandbox with a few known rows before trusting a full run.

## Step 5: Check Identifier Types Row by Row

1. **Validate that every person reference is the type you expect.** In Salesforce, Contact record IDs start with `003` and are 15 or 18 characters. A short numeric value in that column is usually the vendor's own internal ID leaking through.
2. **Set those rows aside rather than guessing a match.** Matching them by name or email is exactly the silent failure described in [Identity Keys vs. Record Keys](09-identity-vs-record-keys.md). Ask the vendor for the correct record IDs.
3. **If the vendor can't supply them, document the decision to drop those rows** and what it means: those people will have no history in the CRM, and that is the explanation when someone later asks why their record looks empty.

## Step 6: Check Required Fields and Picklist Values

1. **Compare every required field on the target object against the file.** Rows missing a required value fail outright.
2. **Look at the pattern before asking.** Rows that are blank in one batch suggest an export problem; rows blank across every group and date range suggest a real state (for example, enrollments created but never started). Describe the pattern to the vendor so their answer can be specific.
3. **Map status values to existing picklist entries** and propose a mapping for the ambiguous ones, rather than adding new picklist values to fit the file.

## Step 7: Split, Label, and Reconcile

Separate the file into clearly named sets so nobody loads the wrong one later:

| Label | Meaning | Who acts next |
|---|---|---|
| `READY` | Passed every check; cleared to load | Whoever runs the load |
| `HELD` | Waiting on a specific answer from the vendor | The vendor, then you |
| `DROPPED` | Will never load; the reason is documented | Nobody; kept for the record |

Then reconcile: `READY` plus `HELD` plus `DROPPED` must equal the vendor's original row count exactly. A count that doesn't reconcile means a row was lost in cleaning, and you want to find that before the load, not after.

## Step 8: Send One Consolidated Question List

Send the vendor every open question in one message, numbered, with the evidence for each: the distinct values found, the row counts affected, and what you assumed while waiting. Say which questions block the load and which only affect the held rows. A complete list answered once is faster than five separate threads, and it shows the vendor exactly why each question matters.

> Thanks for sending the history files. I've reviewed both in detail and mapped them to our fields. We're close, and I need [N] things confirmed before the load runs. I'd rather ask now than have a few thousand rows land wrong and need unwinding.
>
> 1. **[Question].** [What you found, with the distinct values or row counts.] [Why it matters.] [What you assumed in the meantime.]
> 2. ...
>
> Items [1 and 2] block the main load. Items [3 and 4] only affect rows I've set aside, so they aren't holding anything up. The counts reconcile exactly to what you sent, and nothing has been dropped.

## Step 9: Load, Then Verify

1. **Load in sandbox, then verify** a sample of rows from each file against the source, including at least one row from each parent and one row from the overnight time window.
2. **Re-run the same checks against production** before the production load. Production can differ from sandbox, especially for add-on fields and parent records the live integration created recently.
3. **Load production in reasonable batches** and keep an eye on the org's daily API usage; see [Responding to an API Limit Alert](07-api-limit-alert-response.md).
4. **Keep the success and error files** from every run. They're the audit trail for what landed and what didn't.

## Common Failure Modes

| What went wrong | What you see | How the checks above catch it |
|---|---|---|
| Insert instead of upsert on a full export | Duplicates next to live-integration records | Step 1 |
| External ID field missing in the target org | Every row fails | Step 1 |
| Namespace prefix on an unpackaged field | Every row fails on header mapping | Step 2 |
| Vendor ID format changed over time | Child rows fail to find their parent, often most of the file | Step 3 |
| Duplicate external ID on a parent after a vendor resend | Every child row pointing at that parent fails on the upsert | Step 3 |
| UTC vs. local assumption wrong | Dates silently one day off | Step 4 |
| Vendor's internal ID in a record ID column | Rows fail, or worse, get hand-matched to the wrong person | Step 5 |
| Missing required values | Rows rejected | Step 6 |
| Rows lost in cleaning | Counts don't reconcile | Step 7 |

Related: [Runbook: Records Missing from an Incremental Sync](10-runbook-records-missing-from-sync.md), for history that loads correctly but never reaches a downstream system.
