# Packaging a Partner Integration into a Reusable Salesforce Package

*Turning a one-off, hand-configured vendor integration into an installable package, so the fifth client takes an afternoon instead of the two months the first one took.*

## Why Package This at All

The first time you provision a third-party integrator, the work is necessarily manual: a scoping conversation, a permission set built field by field, a round of "why can't I see this record" debugging, a few iterations once the integrator's code is actually running against real data. That process produces real knowledge, exact objects, exact fields, exact edge cases, but by default it lives nowhere except a permission set sitting in one client's org and an email thread.

The fix is to turn that permission set, those custom objects, and the dependencies between them into a Salesforce **Unlocked Package**: a versioned, installable artifact that reproduces the same access model in a new org in minutes. The integrator (or you, if you own the client-side objects) builds it once. Every client after the first is an install plus a short checklist, not a repeat of the original multi-week conversation.

This doc walks through what breaks when you try to package something that was built by hand, why each fix is the right one and not just a workaround, and the packaging mechanics themselves. The running example is a fictional grant-management platform, **GiveAGrant**, syncing applications and reviewer data into a nonprofit's CRM. None of the specifics are real; the failure modes are.

## Before You Package Anything: Prune to What's Actually Used

A permission set built by hand accumulates grants defensively. Someone asks "will I need this later," the answer is "maybe," and the field or object gets added just in case. That habit is harmless in a single hand-configured org. It becomes a hard build error the moment you try to package it.

Package builders validate that every component a permission set references actually exists in the build context. If `GiveAGrant_Platform_Access` has field-level security on a field that belongs to a different vendor's managed package, say a `Membership__c` object that ships with the host CRM's own installed package, the build fails outright: that object doesn't exist in a clean build org, and it won't exist in every future client's org either, since not every client necessarily has that other package installed at all.

The fix is not a workaround, it's the audit you should have done regardless: does the integration's actual code ever read or write this field? If GiveAGrant's export logic never queries `Membership__c`, because it already knows someone is an applicant or a reviewer from its own records, then the grant was defensive, not functional, and removing it is correct independent of the build error. A package that only declares what it actually uses is more portable, not less complete; a new client's org without that other vendor's package installed will work with it immediately instead of failing for a reason nobody can explain from the error message alone.

Two things to check with equal care, but that often turn out differently:

- **Record type visibility** on an object you don't own is almost always safe to drop. It only controls which record type a user can *select* when creating or editing a record, something an API integration never does and a read-only admin view doesn't need either.
- **Field permissions on a field your integration actually reads at runtime** are not safe to drop, even if removing them also clears a build error. If the field carries the identifier your integration uses to resolve a person (see the next section), deleting the permission makes the package build but silently breaks the integration the moment it installs somewhere that isn't already compensating for the gap some other way.

Don't apply the same fix to both. The build error looks identical either way; the correct response does not.

## Split the Permission Set by Who's Using It

Most integrations need two kinds of access, and conflating them into one permission set hides gaps until they surface in production:

| Permission set | Who holds it | Needs |
|---|---|---|
| **API / integration user** | The automated process making callouts | Object and field CRUD on exactly what it reads and writes. No tab visibility, no record type visibility, no UI-facing grants at all, this identity never opens the Salesforce UI. |
| **Admin / platform access** | A human administrator browsing the synced data | Tab visibility so the custom objects appear in the app, record type visibility if they'll create or edit records by hand, plus whatever fields they need to see. |

The trap: if your test user has both permission sets assigned at once, which is the easy way to test, everything appears to work regardless of which permission set actually carries which grant. The gap only shows up once a real deployment assigns the integration user *only* the API-facing permission set, the way it's meant to be scoped, and the automated process suddenly can't read a field it's always been able to read in testing.

Audit each permission set against what actually uses it, not against what a stacked test user can see. If the integration code resolves a person by an identifier field, the API permission set needs read access to that field directly, independent of whatever the admin-facing permission set separately grants.

## Matching People Across Systems, and What Happens When You Can't

This playbook already has a full treatment of identity matching in [Identity Keys vs. Record Keys](09-identity-vs-record-keys.md): use a durable, platform-issued identifier as the primary match key, fall back to email only for the remainder, and never fall back to name. That guidance holds unchanged when the matching happens inside a packaged integration rather than custom code you wrote yourself.

One addition worth calling out on its own, because it's easy to miss until a client asks about it directly: **don't invent a placeholder record for unmatched people.** It's tempting, every applicant needs *something* in the lookup field, and a single "Unmatched" record feels like a clean way to satisfy a required relationship. It isn't. If ten different unmatched applicants all point at the same placeholder, any report or rollup built on that record now silently represents ten unrelated people as one. Leave the lookup null instead, store the external system's own identifier and email as plain reference fields alongside it, and expose a boolean or formula field flagging "no match found" so a client can find and reconcile the exceptions themselves. A null relationship is honest about what the integration doesn't know. A placeholder record hides it behind something that looks like data.

## Referencing a Parent Record by External ID, Not Internal ID

If your integration creates child records that need to point at a parent, say an application that belongs to a specific grant cycle, you have two ways to set that lookup over the API:

1. Resolve the parent's Salesforce record ID yourself first, then write it directly into the lookup field.
2. Let Salesforce resolve it for you, by nesting the parent's **External ID** field inside the relationship name.

The second option is almost always the better one, because it means your integration never has to know or store a Salesforce record ID for the parent at all. It stays addressable purely by the identifier your own system already uses.

The mechanism depends on a distinction that trips people up the first time: a custom lookup field's **API name** always ends in `__c`, permanently, there is no way to rename it to end in anything else. Alongside that stored field, Salesforce automatically derives a separate **relationship name** ending in `__r`, which exists only for use in code, SOQL traversal and this nested-create syntax, never as something you configure in Setup. The two are two views of the same field, not two different fields.

Given a lookup field `GiveAGrant_Cycle__c.Program_Lookup__c` pointing at `GiveAGrant_Program__c`, and assuming `GiveAGrant_Program__c` has an External ID field `External_Program_Id__c`, a record creation call looks like this:

```json
{
  "attributes": { "type": "GiveAGrant_Cycle__c" },
  "External_Cycle_Id__c": "9001",
  "Program_Lookup__r": {
    "External_Program_Id__c": "4821"
  }
}
```

That tells Salesforce: find the `GiveAGrant_Program__c` record where `External_Program_Id__c` equals `4821`, and point `Program_Lookup__c` at it, in the same call that creates the Cycle. No separate query, no stored Salesforce ID anywhere in the integrator's own database.

A same-named field on the child object does **not** create this relationship. If `GiveAGrant_Cycle__c` happens to also have its own field called `External_Program_Id__c` for some unrelated reason, writing a matching value into it does nothing but store a number; only the actual lookup field, referenced by its relationship name, establishes the relationship. Two fields that happen to share a name across two objects are coincidence, not connection, a mistake worth watching for specifically because the error message it produces, a generic "required field missing," doesn't hint at the actual cause.

## The Recycle Bin Will Outlast Your Delete Button

External ID fields are frequently marked **Unique**, which is exactly what makes the pattern above safe: no two parent records can silently collide on the same external identifier. It has a side effect worth knowing before it costs you an afternoon of confusion.

A standard delete in Salesforce is a soft delete. The record moves to the Recycle Bin and is fully recoverable for up to fifteen days (or until the bin is emptied). While it sits there, **unique field constraints still apply against it**. If your test cycle deletes a Program and a Cycle to reset state, then recreates them reusing the same external identifier values, the create collides with your own "deleted" records sitting in the bin, not with anything visibly present in a list view.

The symptom looks like a disappearing record: a list view correctly shows zero live rows, because the row really is gone from the live data, while a create against the same external ID either fails outright or, worse, silently resurrects and updates the old recycled record instead of making a fresh one. Either way, the fix is the same: **permanently delete, not just delete**, as part of any test cycle that reuses external identifier values. A normal delete clears the uniqueness block in fifteen days; emptying the Recycle Bin for those specific records clears it immediately.

## Two Ceilings, Not One: API Limits and Storage Limits

Salesforce orgs govern two completely independent resources, and conflating them during testing leads to the wrong fix:

| Limit | Governs | Error code | Driven by |
|---|---|---|---|
| Daily API requests | How many API calls you make in a rolling 24-hour window | `REQUEST_LIMIT_EXCEEDED` | Call count, not what each call does |
| Data storage | How many records and files the org holds in total | `STORAGE_LIMIT_EXCEEDED` | Record volume, independent of how many calls created it |

If a large-volume test run throws `STORAGE_LIMIT_EXCEEDED`, that is not evidence you're anywhere near the request limit, it's a completely separate governor, and sandboxes in particular often carry a small, fixed storage allocation that a production org of the same client will never share. Don't chase a full-volume dataset in a sandbox to "prove" the integration scales; prove the chunking logic works at a few thousand records, and treat the real ceiling as a production-only question once you know the target org's actual allocation.

Testing the request-limit side doesn't require large data volume either, and that's worth being deliberate about, since the two are easy to conflate precisely because a test that creates a lot of records *also* makes a lot of calls. Two better options, in order of preference:

1. **Mock the error response in your own test suite.** If your integration's error handling checks for a status code containing something like `LIMIT` and `EXCEEDED`, verify that logic against a fabricated `REQUEST_LIMIT_EXCEEDED` response. No Salesforce interaction required, no storage risk, and it tests the thing you actually care about: does your handling code catch it correctly.
2. **Poll the platform's limits endpoint during a real run** (`/services/data/vXX.X/limits` in Salesforce) to watch your actual call count against the ceiling in real time, without needing to force a failure at all. This is the only approach that validates real numbers against a real org, and it costs nothing destructive to try.

Driving the call count up with cheap, non-data-creating requests (simple queries, repeated calls to the limits endpoint itself) is a legitimate way to exercise the real failure path if you need to see it happen, but it's rarely necessary once you've confirmed the handling logic with a mock.

## Building the Package

With the permission sets cleaned up, the mechanics are a standard Salesforce DX flow:

```bash
# Authenticate to the org that has the working configuration, and to a Dev Hub
sf org login web --instance-url https://test.salesforce.com --alias SourceOrg
sf org login web --alias DevHub

# Scaffold a project
sf project generate --name MyIntegrationPackage

# Pull the configuration out of the source org
sf project retrieve start --target-org SourceOrg --metadata \
  CustomObject:GiveAGrant_Program__c \
  CustomObject:GiveAGrant_Cycle__c \
  CustomObject:GiveAGrant_Application__c \
  PermissionSet:GiveAGrant_Platform_Access \
  PermissionSet:GiveAGrant_API_Access \
  Profile:"GiveAGrant Integration User"

# Create the package, then a version
sf package create --name "MyIntegration" --package-type Unlocked --path force-app --target-dev-hub DevHub
sf package version create --package "MyIntegration" --target-dev-hub DevHub --wait 10
```

One retrieval gap is easy to miss and produces a confusing error. If any permission set references `tabSettings` for a custom object, and it almost certainly does if a human is meant to browse that object's records, those tab definitions are a **separate metadata type**, `CustomTab`, stored in its own `tabs` folder. Retrieving `CustomObject` and `PermissionSet` does not pull them along automatically. The package version create step then fails with something like:

```
no CustomTab named GiveAGrant_Application__c found
```

That error does not mean the tab needs to be created, it almost certainly already exists in the source org, which is exactly what let a human browse that object there in the first place. It means the local project is missing the component the permission set is referencing. The fix is to retrieve it explicitly:

```bash
sf project retrieve start --target-org SourceOrg --metadata \
  CustomTab:GiveAGrant_Program__c \
  CustomTab:GiveAGrant_Cycle__c \
  CustomTab:GiveAGrant_Application__c
```

Confirm what actually exists before assuming anything is missing: Setup, Quick Find "Tabs", the Custom Tabs section lists every tab in the org. If an object genuinely has no tab there, that's the one case where you'd need to create one before it can be retrieved or packaged, rather than simply widening the retrieve command.

## Depending on a Package You Don't Own

A real integration frequently needs a field or object that belongs to the host platform's own managed package rather than anything the integrator built, most commonly an identifier field used for the SSO-based matching described earlier. Deleting that reference is the wrong move here, unlike the unused-grant case earlier in this doc: the field is load-bearing, and stripping it to clear a build error would make the package install cleanly while silently breaking the matching logic the moment it runs somewhere that doesn't already have broader access configured some other way.

The correct fix is declaring the other package as an explicit **dependency**, so the build environment has it available and the reference resolves instead of failing. This requires the exact **Subscriber Package Version ID** (an `04t...` value) of the package you depend on; there's no way to declare this kind of cross-publisher dependency by namespace and a version label alone, the platform needs the specific ID.

You don't need the package's owner to hand you that ID. If you already have API access to any org that has the dependency installed, which you will, since that's the org your integration already targets, you can query it yourself. Two things trip this up on the first try. `InstalledSubscriberPackage` lives only in the Tooling API, not the standard API the CLI queries by default, so the query needs the tooling flag or it comes back as an unsupported sObject. And `SubscriberPackage.NamespacePrefix` is selectable but not filterable, a property of the field itself, not a syntax issue, so it can appear in the field list but not in a `WHERE` clause. The simplest path around both is to pull back every installed package and read the right one off the list, there are usually only a handful in any org:

```bash
sf data query --use-tooling-api --target-org SourceOrg --query \
  "SELECT SubscriberPackageVersionId, SubscriberPackage.Name, SubscriberPackage.NamespacePrefix \
   FROM InstalledSubscriberPackage"
```

Find the row where `NamespacePrefix` matches the package you depend on, and take its `SubscriberPackageVersionId`.

With that ID, add it to `sfdx-project.json` as a named alias, then reference the alias from your package directory's dependency list:

```json
{
  "packageDirectories": [
    {
      "path": "force-app",
      "default": true,
      "package": "MyIntegration",
      "versionName": "ver 0.1",
      "versionNumber": "0.1.0.NEXT",
      "dependencies": [
        { "package": "HostPlatformPackage" }
      ]
    }
  ],
  "packageAliases": {
    "MyIntegration": "0Ho000000000000AAA",
    "HostPlatformPackage": "04t000000000000AAA"
  }
}
```

Pin this to whatever minimum version actually carries the field you depend on, not to the exact build installed in the one org you tested against. Salesforce's own install-time check enforces "the target org must be at this version or newer," so a reasonable floor covers every client that's current or ahead, and a client genuinely behind that floor gets a clear, correct failure telling them to update the other package first, rather than a confusing error from your integration's own code.

## What This Buys You

None of the fixes above are expensive individually. What they add up to is the difference between an integration that one engineer can reason about in one org, and one that installs, predictably, into the next client's org without a repeat of the original conversation. The permission-set split means a client's security review has a clean, minimal artifact to look at instead of a grant list built up over months of "just in case." The external ID pattern means onboarding a new client never involves hand-mapping internal record IDs. The dependency declaration means the package tells you, at build time, exactly what the target org needs before it, instead of failing at runtime in a way that looks like your own bug.

The first client is always the expensive one. The point of packaging it is to make sure the second one isn't.

## Pre-Ship Checklist

- Every field and object permission in the package is something the integration's code actually reads or writes; anything defensive or "just in case" has been removed
- The API-facing permission set and the admin-facing permission set are separate, and each has been tested in isolation, not as a stacked test user
- No unmatched external record is ever linked to a shared placeholder; the lookup is left null with a visible flag field instead
- Parent-child relationships are created through the relationship name and the parent's External ID, not a stored Salesforce record ID
- Test cycles that reuse External ID values permanently purge the Recycle Bin, not just delete
- `STORAGE_LIMIT_EXCEEDED` and `REQUEST_LIMIT_EXCEEDED` are never treated as the same problem
- `CustomTab` metadata is retrieved explicitly for every object a human needs to browse
- Any dependency on another package's field is declared in `sfdx-project.json` against a Subscriber Package Version ID, pinned to a sensible minimum, never deleted to make a build pass

Pairs with [Identity Keys vs. Record Keys](09-identity-vs-record-keys.md) and the [Third-Party Integrator Security Playbook](03-integrator-security-playbook.md).
