---
name: wikijs-page-draft
description: "Draft and revise documentation pages in Wiki.js markdown for an internal team wiki or an external developer/vendor documentation site, gathering source material from tickets and chat threads when the context isn't already in the conversation. Use when asked for a wiki page, KB article, runbook, SOP, setup guide, or reference page, or when asked to 'document this', 'write this up for the wiki', or 'turn this thread into a page', even if the word 'wiki' never appears."
---

# Wiki.js Page Draft

Produce publishable Wiki.js markdown. The output is a file the user pastes directly into the Wiki.js editor, so it has to render correctly on the first paste and contain nothing that isn't meant to be published.

## Step 1: Gather the source material

If the page documents a resolved issue, an integration pattern, or a troubleshooting fix, and the full context isn't already in the conversation, go get it instead of asking the user to retype what's already written down.

- **Ticket tracker** (Jira or similar): search by ticket key if one was given, otherwise by keyword, customer, or date range. Read the description, **all resolution comments**, and linked issues. The comments are usually where the actual fix lives.
- **Chat** (Slack, Teams, or similar): search for the ticket number or feature name, then read the **whole thread**, not just the matching message. The confirmed answer is usually several messages after the first proposal. Use the last confirmed answer, not the first idea.
- **When tickets and chat disagree:** the ticket resolution is authoritative for what actually shipped. Chat explains the why and surfaces caveats that never made it into the ticket. Note any real conflict in Step 8 instead of quietly picking one.
- **Customer boundaries:** pull only content that belongs to the customer or project this page covers. When people or threads span multiple customers, confirm which one a thread belongs to before using it. A page that blends two customers' details is worse than a page with an acknowledged gap.
- **Synthesize, don't paste.** Write in the page's own voice. Ticket numbers and thread links go in a short Source line near the end, not scattered through the prose.

## Step 2: New page or update?

Work out whether a page on this topic already exists. If there's no direct access to the wiki, ask for the current page content or link in one line, and keep going with what you know.

- **Updating:** a new ticket about a known issue usually means one section needs a new step, a corrected value, or a new caveat. Keep everything the new material doesn't contradict. Before the file, summarize in chat what changed and why, so the user can confirm the change is scoped correctly.
- **New:** continue to Step 3.

## Step 3: Audience and destination

Internal and external pages have different jobs. Identify which one this is before writing. See `references/page-templates.md` for the template for each.

| Destination | Audience | Job |
|---|---|---|
| Internal wiki | Teammates who already have context | Assist someone mid-task |
| External developer docs | Vendors, integrators, customer IT | **Replace a conversation**; pointing someone at this page has to be enough |

You need two things before drafting: the audience/destination and the one-sentence purpose of the page. Take both from the conversation and the Step 1 research when you can, state the assumption in one line, and start. Only ask when the answer really isn't available.

## Step 4: Read the syntax reference

Read `references/wikijs-syntax.md` before every draft.

Wiki.js class annotations attach to the deepest element before them, so a callout containing a bullet list silently renders as an unstyled quote around a colored list. A plain markdown preview won't show this. It shows up only after publishing, on the page people are already reading.

## Step 5: Structure the page

Order of operations throughout: prerequisites before steps, dependencies stated explicitly, numbered steps for anything someone will execute.

- `H2` for the page title, `H3` for major sections, `H4` for subsections
- **No `---` horizontal rules.** A line of hyphens directly under text is a heading marker, so the paragraph above it silently becomes a header. Separate sections with a blank line, `<br>`, and another blank line.
- Field names, object names, API names, permission sets, paths, and literal values in `inline code`. That's what makes a page usable as a reference.
- **Callouts sparingly.** Default to prose. Save callouts for the one or two things that do real damage if missed. A page with six callouts effectively has none.
- **Link rather than restate.** If content lives on another page, link it or add a `{.links-list}` card. Two copies always drift, and the stale one is the one someone finds.

## Step 6: Explain the why, not just the what

Every procedure states the consequence of getting it wrong alongside the mechanics: what breaks, who sees it, and whether it fails loudly or silently. Label silent failures as silent, because that's exactly why nobody caught them before the page existed.

Any page about work shared between teams, or between your team and a vendor or customer, states what each side owns. Ambiguity on that line generates the tickets the page was written to prevent.

Never describe the reader as new or inexperienced. Frame difficulty as a property of the material.

## Step 7: House facts

Documentation gets cited back months later, so errors outlive the draft. If your organization has facts that must always be stated the same way (canonical identifiers, systems that are off-limits to certain teams, retired approaches that must never be recommended, escalation thresholds, naming conventions), keep them in a short `references/house-facts.md` and read it before drafting.

Never restate a house fact from memory in a different form. If a draft touches one, quote it as written.

## Step 8: Deliver the file, then flag what isn't verified

Write the page to a `.md` file named in kebab case after the page title. The file contains the page and nothing else: no preamble, no "here's your draft," no explanation of choices. Notes go in chat.

In chat, alongside the file:

1. Suggest a wiki path for the page.
2. Suggest the `{.links-list}` card to add to the parent landing page, so the page is reachable. A page nobody can navigate to is a page nobody reads.
3. List anything the draft couldn't confirm: inferred values, steps never run against a real environment, claims from a single thread, ticket-versus-chat conflicts, and whether the new-versus-update check was ever confirmed against the live wiki.

A wiki page carries institutional authority the moment it's published. Unverified content has to be visibly unverified while it's still a draft.

## Reference files

- `references/wikijs-syntax.md`: Wiki.js markdown extensions, callout mechanics, and the class-targeting gotcha. Read before every draft.
- `references/page-templates.md`: Templates for internal and external pages.
- `references/house-facts.md` *(optional, you create this)*: your organization's must-never-drift facts.
