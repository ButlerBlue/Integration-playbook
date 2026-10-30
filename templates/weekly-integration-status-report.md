# Weekly Integration Status Report (Template)

*A fill-in weekly update for an integration or implementation lead to send upward. It is built to be read in two minutes, forwarded without edits, and acted on: what moved, what was decided, and what is still open against which date.*

**How to use it:** send it at the same time every week, to the same audience, in the same order. Write every line so it would still make sense if your manager forwarded it to their manager word for word, because sometimes they will. If a section has nothing in it this week, write "None this week" rather than dropping it, so readers learn to trust that an empty section means empty.

## The Report

**Weekly status, [Name], week of [Start date] to [End date]**

**Key accomplishments**

[Two to five short paragraphs. Lead each one with the outcome, then the one sentence of how. Name the system and the effect on clients, not the effort.]

- [Outcome first, e.g., "Traced a vendor's repeated bulk load failures to an org storage ceiling rather than the daily API limit, which redirected their fix. The vendor is adding test coverage for both limit types."]
- [e.g., "Diagnosed and closed a sync error backlog. Root cause was a deactivated reference value tripping a validation rule; walked the support team through clearing the queued records."]
- [e.g., "Reset intake for a partner so new integration requests come to the integration team directly instead of through the client."]

**Decisions made**

[Each decision on one line, with who made it if it wasn't you. This is the section leadership reads most closely, because it is where direction changed.]

- [e.g., "Client Credentials is the direction for the internal sync's authentication change, rather than JWT Bearer, because it fits the existing per-client credential model with minimal rework."]
- [e.g., "Manual imports for two affected clients stay a one-time cleanup owned by the diagnosing team, not a repeatable support procedure, until the platform fix ships."]

**Still open or in progress**

| Item | Status | Owner | Measured against | Next step |
|---|---|---|---|---|
| [Short name, plus ticket reference internally] | [e.g., In requirements] | [Name or team] | [The external or committed date it must beat, e.g., "Platform enforcement in production, [date]"] | [The single next action and when] |
| [ ] | [ ] | [ ] | [ ] | [ ] |

**Needs a decision** *(include only when something does)*

1. [The decision, the options, your recommendation, and the date by which it is needed.]

**Development** *(optional)*

[One line on training or certification progress, if your team reports it.]

## Writing Rules

1. **Outcome before effort.** "Spent most of the week on X" tells the reader where time went. "X is now fixed for three clients" tells them what changed. Lead with the second and keep the first to a clause.
2. **Every open item names the date it is measured against.** A status of "in requirements" means nothing on its own. "In requirements, original release target has slipped, platform enforcement is [date]" is a risk leadership can act on.
3. **Say when a target slipped, the first week it slips.** A slipped date reported early is a planning input. Reported late, it is a surprise.
4. **Separate confirmed from inferred.** Use "as I understand it" for anything not yet verified. A plausible theory repeated upward as fact is hard to walk back.
5. **Describe the system, not the people.** "The request was routed through the client and landed with the wrong team" fixes the process. Naming who routed it fixes nothing and travels badly.
6. **Bring a path with every problem.** If an item needs leadership, the report says what you propose, not only what is wrong.
7. **Give credit by name.** A line thanking a colleague who unblocked something costs nothing and is often the line that gets remembered.
8. **Keep it short.** If an item needs more than three sentences, link to the ticket or brief and summarize here. The [Defect Escalation Brief](defect-escalation-brief.md) is a good home for the detail.

## Why This Format Works

- **The same order every week makes change visible.** Readers learn where to look, so a new item under "Needs a decision" stands out without bold text or exclamation points.
- **Decisions get their own section** because they are the part most often lost. A decision buried in an accomplishments paragraph gets re-litigated a month later.
- **The "measured against" column turns status into risk.** Leadership can weigh a known gap against a known date. They can't weigh "still in progress."
- **Written to be forwarded, it builds trust upward.** A report that survives being forwarded unedited saves your manager time and shows the altitude you work at.

See also: [Leading an Integration Fleet Through a Platform OAuth Enforcement Change](../docs/13-platform-oauth-enforcement-fleet-migration.md) for program-level status states, and [Governing Interim Workarounds](../docs/14-interim-workaround-governance.md) for reporting a workaround with its cost visible.
