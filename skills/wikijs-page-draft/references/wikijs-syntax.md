# Wiki.js Markdown Syntax

Wiki.js supports CommonMark plus GitHub Flavored Markdown, then adds its own class-annotation extensions. The extensions are what break when written from memory, so they're covered in detail here. Standard markdown behaves as expected and isn't repeated.

## Contents

1. Headers
2. Callout blocks
3. The class-targeting gotcha
4. Navigation cards
5. Tabsets
6. Code and field names
7. Section breaks
8. Tables
9. Links and anchors
10. What not to use

<br>

## 1. Headers

`H2` is the page title. `H3` is a major section. `H4` is a subsection. Don't use `H1`: Wiki.js renders the page title from page properties, so an `H1` in the body duplicates it.

Headers generate their own anchors automatically.

<br>

## 2. Callout blocks

A callout is a blockquote with a class on the line immediately after it. Blank line before the blockquote; class on the line directly after the last quoted line.

```markdown
> Production changes require a sandbox test first. No exceptions.
{.is-danger}
```

Four classes, each with a job:

| Class | Use for |
|---|---|
| `{.is-info}` | Context the reader needs, but that does no damage if skimmed |
| `{.is-success}` | What a correct result looks like |
| `{.is-warning}` | Something that will cost time or create a support ticket |
| `{.is-danger}` | Something customer-visible, destructive, or silently wrong |

Two or three callouts on a page is already a lot. At six, readers learn to skip them, including the one that mattered.

<br>

## 3. The class-targeting gotcha

The `{.class}` annotation attaches to the deepest element before it. When a blockquote contains a list, the class lands on the `ul`, not the `blockquote`, and you get an unstyled quote wrapped around a colored list.

A plain markdown preview won't show this. It shows up after publishing.

Broken:

```markdown
> Confirm before acting:
> - Who owns the integration
> - Who owns the data
{.is-warning}
```

Correct, using the decorate syntax to name the target element:

```markdown
> Confirm before acting:
> - Who owns the integration
> - Who owns the data
<!-- {blockquote:.is-warning} -->
```

The pattern generalizes to `<!-- {tag-name:.class-name} -->` wherever the preceding content is ambiguous. The simpler fix is to keep callouts to plain prose, which also keeps them short, which is the point of a callout.

<br>

## 4. Navigation cards

`{.links-list}` renders a list of links as navigation cards. The italic part of each link becomes the card subtitle.

```markdown
- [SSO Overview *OAuth 2.0 flow and token lifetimes*](/integrations/sso/overview)
- [Sync Field Mappings *What syncs and when*](/integrations/sync/mappings)
- [Escalation Paths *When a delay becomes an incident*](/support/escalation)
{.links-list}
```

Use these on landing pages and under a "Related pages" `H3` at the bottom of a page. This is what makes a new page reachable, so a new page almost always comes with a card to add to its parent.

`{.grid-list}` renders plain list items as a grid, no links needed. Occasionally useful for a short set of parallel items.

```markdown
- Sandbox validation
- Permission set assignment
- Production deployment
{.grid-list}
```

<br>

## 5. Tabsets

A header marked `{.tabset}` turns the headers one level below it into tabs.

```markdown
### Permission Set Packaging {.tabset}

#### Full Platform License

Content for the full-license variant.

#### API Only

Content for the API-only variant.
```

A good fit for content that branches by variant, license type, or environment. A bad fit for sequential steps, because a reader following a procedure can't see step 3 while working on step 2.

<br>

## 6. Code and field names

Inline code for every field name, object name, API name, permission set, path, and literal value: `External_Identity_Id__c`, `Contact`, `/services/data/v62.0/`.

Fenced blocks with a language tag for anything multi-line. The tag drives syntax highlighting.

````markdown
```sql
SELECT Id, Name, LastModifiedDate
FROM Contact
WHERE LastModifiedDate = LAST_N_DAYS:7
```
````

Wiki.js has no SOQL grammar, so use `sql` for SOQL; it highlights close enough to be useful. Also worth using: `json`, `yaml`, `bash`, `apex`, `javascript`, `html`.

<br>

## 7. Section breaks

Don't use `---` or `***` horizontal rules. A line of hyphens directly beneath text is a setext heading marker, so the paragraph above silently becomes a header and the break disappears. It only works with an empty line above it, which makes it fragile on any page someone will edit later.

Use this instead, between the end of one section and the next header:

```markdown
Last line of the previous section.

<br>

### Next Section
```

<br>

## 8. Tables

Standard pipe tables: header row, separator, body.

```markdown
| Field | Object | Purpose |
|---|---|---|
| `External_Identity_Id__c` | `Contact` | SSO identity key |
| `Id` | `Contact` | Record match key |
```

Tables suit field references, mappings, and ownership matrices. They don't suit procedures, which need numbered steps.

<br>

## 9. Links and anchors

Internal links are root-relative paths without a file extension.

```markdown
See [SSO Overview](/integrations/sso/overview) for token lifetimes.
```

External links use the full URL. External-facing pages link to the canonical documentation rather than reproducing it.

For a specific section, append the auto-generated anchor: the header text, lowercased, with hyphens for spaces.

```markdown
See [escalation thresholds](/support/escalation#sync-delay-thresholds).
```

<br>

## 10. What not to use

| Avoid | Reason |
|---|---|
| `H1` in the body | Duplicates the page title from page properties |
| `---` and `***` rules | Turns the paragraph above into a header |
| Raw HTML beyond `<br>` | Renders inconsistently and doesn't survive editor round-trips |
| Callouts containing lists | The class attaches to the list, not the quote, unless decorated |
| Restating content from another page | The copies drift, and the stale one gets found |
