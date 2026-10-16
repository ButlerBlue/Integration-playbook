# Defect Escalation Brief (Template)

*A fill-in brief for reporting an integration defect to the product owner or engineering team so it can be prioritized on the first read. A defect that arrives as "the sync is broken for a client" waits in a queue. One that arrives with a traced case, the mechanism, the exposure, and specific questions gets a decision.*

**How to use it:** fill this in once you have traced at least one concrete case end to end. Paste it into the defect ticket as the description, and keep it current as new cases are confirmed. Every section should be short. If a section can't be filled in yet, say so rather than leaving it blank, because "unknown" is information.

## 1. Summary

| | |
|---|---|
| **One-line statement** | [What the system does, under what condition, e.g., "The outbound sync skips records whose status changed since the last run if they no longer meet the sending criteria."] |
| **Type** | [Defect / Gap in capability / Regression after a change] |
| **Severity, in business terms** | [What the client can't do, or what data is wrong, and for how many] |
| **Workaround in place?** | [None / One-time cleanup / Interim bridge; see [Governing Interim Workarounds](../docs/14-interim-workaround-governance.md)] |
| **Ask** | [The decision you need, e.g., "Prioritize ahead of the current roadmap date," "Confirm intended behavior"] |

## 2. Traced Case

Walk one real case from the symptom to the cause. Use record IDs, not names.

1. **Reported symptom:** [What the client saw, and when]
2. **Source record state:** [Relevant field values and timestamps in the source system]
3. **What the integration did:** [Which run, which step, what it sent or skipped]
4. **Where it diverges from expected:** [The exact step where the behavior is wrong]
5. **Mechanism:** [One paragraph a non-engineer can follow, e.g., "A full load only re-sends records that changed on the sending side, so an edit made on the receiving side is never corrected."]

```sql
-- The query that shows the source state, so the reader can reproduce it
SELECT Id, LastModifiedDate, SystemModstamp, [Relevant_Field__c]
FROM [Object]
WHERE Id IN ('[Record Id]')
```

**Confirmed vs. inferred:**

| Statement | Status |
|---|---|
| [e.g., The record qualified at the time of the run] | Confirmed by [query / log / screenshot] |
| [e.g., The same logic applies to every client on this version] | Inferred, not yet verified |

## 3. Confirmed Cases

Add a row each time a new case is confirmed. A second confirmed case with evidence is often what moves a defect from "under consideration" to scheduled.

| # | Client or environment | Record(s) | Date confirmed | Same root cause? | Evidence |
|---|---|---|---|---|---|
| 1 | [ ] | [Record Id] | [ ] | Yes | [Link to query output or log] |
| 2 | [ ] | [Record Id] | [ ] | [Yes / Suspected] | [ ] |

## 4. Exposure

| | |
|---|---|
| **Who is affected today** | [Clients, environments, or record sets, marked as confirmed or "as I understand it"] |
| **Who could be affected next** | [e.g., any new client on this version, any environment refreshed after a date] |
| **Trigger events** | [What would turn latent exposure into a live failure: a re-authorization, a sandbox refresh, a platform enforcement date, a busy season] |
| **Detection** | [Does it fail loudly, or does data quietly go missing? How would anyone know?] |

## 5. Open Questions for the Product Owner

Questions only the product owner or engineering can answer. Number them so replies can reference them.

1. [e.g., When two exclusion rules conflict, which takes precedence?]
2. [e.g., The documentation gives two different values for the same window. Which is correct?]
3. [e.g., Is this intended behavior? If so, where should it be documented?]

## 6. What We've Already Done

- [Workaround applied, with its type and owners]
- [Related tickets linked]
- [Who has been told, and what they were told]

## 7. Proposed Next Steps

| Action | Owner | Due |
|---|---|---|
| [Confirm intended behavior] | [Product owner] | [ ] |
| [Test fix in sandbox] | [Engineering] | [ ] |
| [Update client-facing staff] | [Integration lead] | [After fix is confirmed] |

## Why This Format Works

- **The traced case does the persuading.** A reader who can follow one record from symptom to cause doesn't need to be convinced the defect is real.
- **Confirmed vs. inferred protects everyone.** It keeps a plausible theory from being repeated as fact, and it shows the reader exactly what still needs checking.
- **Exposure and trigger events turn a defect into a risk decision.** Leadership can weigh a known gap against a known date; they can't weigh "it's broken for some clients."
- **Numbered questions get numbered answers.** Open-ended threads drift. A list of three specific questions usually comes back with three specific answers.

See also: [Governing Interim Workarounds](../docs/14-interim-workaround-governance.md) and [Runbook: Records Missing from an Incremental Sync](../docs/10-runbook-records-missing-from-sync.md).
