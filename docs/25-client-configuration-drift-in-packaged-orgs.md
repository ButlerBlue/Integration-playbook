# Protecting a Packaged Integration from Client Configuration Drift

*What to do when a client's own admin change quietly breaks a package or integration that depends on it, and the standard that keeps the next one from reaching production unnoticed.*

## Why This Needs a Standard

A package installed in a client's org makes assumptions about that org. It sets a picklist value by its exact text. It refers to its own fields by name. It expects a required field, a record type, or a status to exist the way it shipped. The client's admins, meanwhile, own their org and are free to rename, add, and tidy as they see fit. Most of those changes are harmless. A few land exactly on something the package depends on.

When that happens, three things make it expensive:

1. **The error appears far from the change.** The admin renamed a value weeks ago; the failure shows up today, in a sign-up flow or a partner's API call, with a message that does not mention the rename.
2. **Everyone looks at the package first.** The vendor gets a defect report, the vendor investigates its own code, and days pass before anyone checks what changed in the org.
3. **Some failures happen after an irreversible step.** If the process charges a card, sends an email, or pushes a record to another system before it fails, every retry repeats that step. Users who try again because they saw an error may have been charged each time.

This page covers the response to one occurrence and the standard that turns it into a fleet-wide safeguard.

## The Two Shapes of Drift

Most drift that breaks a package takes one of two shapes. The error text usually tells you which.

| Shape | What happened | What the error looks like | Why it breaks |
|---|---|---|---|
| **A value the package sets was changed** | An admin renamed a status or picklist value, even by one letter of capitalization | The package's exact string, rejected. For example, a lead conversion returning `invalid convertedStatus: [Value]` with `FIELD_CUSTOM_VALIDATION_EXCEPTION` | The package sends the value it shipped with, and the org no longer has a value that matches it |
| **A local field collides with a packaged field** | An admin created a custom field on a packaged object with the same name as a packaged field, minus the namespace prefix | A message such as `duplicate field selected: [ns]__Field__c` | Inside the package's code, the local field and the packaged field can resolve to the same reference, so the query or save fails |

Other changes belong on the same watch list, because they fail in the same way: a new validation rule or newly required field on an object the package writes to, a deactivated value the package still sets, and a record type or business process the package's records can no longer use.

## Step 1: Read the Error for the Value It Names

The error almost always names the exact string or field that failed. Start there, not with the vendor.

1. **Copy the value from the error exactly,** including capitalization and spacing.
2. **Compare it to the org's current configuration** using a describe call or the setup page for that field. Check every org in the path: the sandbox where the error was reported and production.
3. **If the error names a field, list every field on that object with a similar name,** packaged and local. A local `Field__c` sitting next to a packaged `[ns]__Field__c` is the collision.

If the configuration matches what the package expects, it is not drift. Route it as a package defect with the [Defect Escalation Brief](../templates/defect-escalation-brief.md).

## Step 2: Find When It Changed

The date of the change sets the size of the problem.

1. **Check the org's Setup Audit Trail** for the rename or the field creation and note the date. It keeps a limited window of history, so check early; if the change is older than the window, use the first failed transaction as the start date instead.
2. **Treat everything since that date as in scope.** Every transaction that ran through the affected process since then either failed or behaved differently.
3. **Ask the client admin what the change was for,** without judgment. Sometimes the answer is "a cleanup," sometimes it is a real need that will have to be met another way.

## Step 3: Size the Damage Before You Fix Anything

When the failure happens after an irreversible step, find out what actually went out the door before anyone retries or refunds.

1. **Use the external reference on the record as the tell.** Most processes that call an outside system store that system's reference ID on the record. A populated reference means the outside step happened. A blank one means nothing was sent.
2. **Separate completed, in-flight, and never-sent attempts** in one list per affected person.
3. **Confirm with the outside system's owner whether in-flight transactions can be cancelled.** If they cannot, the clean path is to let them complete and then reverse the duplicates. Reversing before they settle can leave records in a state that is harder to reconcile.
4. **Hand the list to the client with a recommended action per row,** and let the client's finance or operations owner make the call on refunds.

## Step 4: Fix at the Smallest Point, Sandbox First

1. **For a renamed value, rename it back in place.** Do not add a second value and do not use a replace operation. Salesforce documents that renaming a picklist value changes its label, while replacing a value is a global change that rewrites existing records. Confirm in your org whether the package matches on the label or the API name before you choose.
2. **Update every place the value lives,** for example the value set and any business process or record type that lists it.
3. **For a colliding field, confirm with the client admin that it is unused and its data is not needed,** then have the client remove it. If they need what the field was for, give it a name that cannot collide.
4. **Do it in sandbox, verify, then production.** Verify with a describe call in each org and by running the real process once.
5. **Tell the vendor they can stand down** as soon as the cause is confirmed as configuration. Ask, without urgency, whether a later release guards against this shape of drift. Do not hold the fix waiting for the answer.

## Step 5: Sweep the Fleet

One occurrence is a signal about every org running the same package.

1. **Check every other org and sandbox for the same value or field,** including developer sandboxes that a team member may be testing in today.
2. **Fix the sandboxes too.** A sandbox that still has the drifted value will reproduce the error the next time a partner tests against it, and someone will open a new defect for it.
3. **Record the check** so the next person can see which orgs were confirmed clean and when.

## Step 6: Publish a Protected Configuration Register

The lasting fix is making the package's assumptions visible to the people who change the org. Keep a short register per package, shared with client admins at go-live and linked from your admin documentation.

| Protected item | Why the package depends on it | Who may change it | How to check it |
|---|---|---|---|
| [Status or picklist value] | Set by exact text during [process] | Implementation team only, after a package review | Describe the field; compare to the register |
| [Packaged field name] | Referenced by the package; a local field with the same name collides | Nobody adds a local field with this name | List fields on the object; look for a local match |
| [Required field or validation rule on a packaged object] | Package writes records here without that field | Client admin, after confirming with the implementation team | Run the package process once in sandbox |
| [Record type or business process] | Package creates records with it | Implementation team only | Describe record types and processes |

Ownership is what makes the register work:

| Role | Owns | Does not own |
|---|---|---|
| Client admin | Their org's configuration, and checking the register before changing a listed item | Deciding whether a listed item is safe to change |
| Implementation team | The register, the fleet sweep, and the fix when drift is found | Refund or reversal decisions for the client |
| Package publisher | Confirming what the package depends on, and any product guard against collisions | Changing a client's configuration |

## Step 7: Tell the Client What Changed, Without Blame

The client's admin made a reasonable change in an org they own. The message should describe the system, not the person.

> Hi [Client Contact],
>
> We found the cause of the [process] errors. A [status value / field] in your org was [renamed / added] on [date], and our package depends on it matching exactly. We've [restored the original value / confirmed it can be removed] in your sandbox and production, and [process] is working again.
>
> Because the failures happened after [the outside step], some people may have been [charged / sent] more than once. The attached list shows each attempt and our recommendation for each one. [Name] on your side is the right person to approve those.
>
> To keep this from happening again, here is the short list of items in your org that our package depends on: [link to register]. If you need to change any of them, let us know first and we'll find a way that works for both.

## Common Failure Modes

| What went wrong | Consequence | Prevented by |
|---|---|---|
| Defect sent to the vendor before checking the org | Days lost while the vendor investigates code that is working | Step 1 |
| Fix applied before the damage was sized | Duplicate charges or sends discovered later, by the people affected | Step 3 |
| In-flight transactions reversed before they settled | Records harder to reconcile than the original duplicates | Step 3 |
| New value added instead of renaming back | Two near-identical values, and reports split across them | Step 4 |
| Only the reported org fixed | The same error resurfaces in a sandbox or another client's org | Step 5 |
| Dependencies known only to the implementation team | The next admin cleanup breaks the package again | Step 6 |
| Message framed around who made the change | A defensive client, and a slower answer on refunds | Step 7 |

Related: [Vendor Change Requests in a Client Org](19-vendor-change-requests-in-client-orgs.md) for changes in the other direction, [Status Discipline for Vendor Fixes](21-vendor-fix-status-discipline.md), [Packaging a Partner Integration into a Reusable Salesforce Package](18-packaging-integration-for-reuse.md), and the [Vendor Change Authorization](../templates/vendor-change-authorization.md) template for recording a configuration change.
