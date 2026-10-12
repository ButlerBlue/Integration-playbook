---
name: data-load-preflight
description: "Review a vendor's data export (CSV or spreadsheet files) before it is loaded into a CRM such as Salesforce, and return a load verdict, the files split into ready, held, and dropped sets, and one consolidated list of questions for the vendor. Use when asked to 'check this file before we load it', 'prep this history file', 'review this export', 'can we upsert this', or whenever a vendor sends historical or bulk data for import."
---

# Data Load Preflight

Review a vendor's export before anyone loads it, so the load runs once and never has to be unwound. The standard behind this skill is [Runbook: Pre-Load Checks for a Historical Data Load from a Vendor](../../docs/11-runbook-historical-data-load.md). Read it if it's available.

Most failures in a bulk load are silent: a row attached to the wrong parent, a date one day off, a duplicate beside a record the live integration already wrote. This skill exists to find those before they happen, and to turn every open question into one clear message for the vendor.

## Step 1: Collect the inputs

Ask for anything missing. Don't guess at any of these:

- The files, and which target object each one loads into
- The target org's real field API names for each object (a describe or field list, not the documentation)
- Which field is the external ID on each target object, and whether it is marked External ID and Unique
- Whether each file is a full export or a delta
- The field types for every date and datetime column
- Required fields and picklist values on each target object
- The parent records already in the org, with their external ID values, for every parent the file references

If the user can't provide the org details, say which checks you can't run and mark them as open, rather than assuming they pass.

## Step 2: Run the checks

Work through every file. Use a script for counts and pattern checks rather than eyeballing them, and report exact numbers.

| # | Check | What to report |
|---|---|---|
| 1 | Upsert or insert | Full export with possible existing rows means upsert; flag any plan to insert |
| 2 | External ID present | Missing or undeployed key field means the file can't load |
| 3 | Re-runnable | Any target object with no external ID can't be safely re-run; say so prominently |
| 4 | Header names | Each header against the org's API names, including namespace prefixes on add-on fields |
| 5 | Parent keys | Every distinct parent key, its row count, and whether it matches an org record exactly, format included |
| 6 | Time zone | Whether timestamps are UTC or local, how many rows land on a different calendar day in each reading, and which target fields are date-only |
| 7 | ID types | Rows where a record ID column holds something that isn't a valid record ID (in Salesforce, Contact IDs start with `003` and are 15 or 18 characters) |
| 8 | Required fields | Rows missing a required value, and whether the gaps cluster or spread across the file |
| 9 | Picklists | Values that don't match an existing picklist entry, with a proposed mapping |
| 10 | Reconciliation | Ready plus held plus dropped equals the original row count exactly |

When a parent key doesn't match, look for a pattern before reporting it: a format change over time (plain numbers becoming versioned pairs, added prefixes, leading zeros) usually explains a whole group of mismatches at once.

## Step 3: Split the files

Produce separate files and never modify the originals:

- `READY`: rows that pass every check
- `HELD`: rows waiting on a specific vendor answer, with the question number in a column
- `DROPPED`: rows that will never load, with the documented reason

Name the files so the status is obvious from the file name alone.

## Step 4: Return the review

Lead with the verdict, then the reasoning.

```
Verdict: [Clear to load / Clear to load READY files only / Not ready]

Load plan:
- [file] into [object], upsert on [field], [N] rows
- Run in sandbox first, then re-check against production before the production load

Blocking (must be answered before any load):
1. [Check #] [What was found, with counts] [Why it matters]

Held for a second pass (not blocking):
1. [Check #] [What was found] [Rows affected]

Dropped:
- [Rows] [Reason] [What that means for the people or records involved]

Reconciliation: [ready] + [held] + [dropped] = [original] (matches / does not match)

Assumptions made while waiting:
- [Assumption] [Rows it affects]
```

Then draft the message to the vendor:

- One message, every question numbered, with the evidence for each: the distinct values, the row counts, and what you assumed in the meantime.
- Say which questions block the main load and which only affect held rows.
- Ask what their live integration writes today, not only what the export contains.
- Close by confirming the counts reconcile and nothing has been lost.

## Don't

- Don't match people by name or email to fill a missing record ID. Ask for the ID, or drop the row with a documented reason.
- Don't recommend a production load before a sandbox run.
- Don't recommend loading during a data freeze or ETL window, even a small file.
- Don't round counts or say "a few" when you have the exact number.
- Don't describe the vendor's export as careless. Formats change over time and the material is complex; describe what was found.
