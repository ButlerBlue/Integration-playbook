# Page Templates

Two destinations, two jobs. Pick the template that matches Step 3 of the skill.

<br>

## Internal wiki page

**Audience:** teammates who already have context and are in the middle of a task.

**Job:** get them to the right action quickly, and make sure they know what breaks if they skip a step. Assume they know the systems; don't assume they've seen this specific issue.

````markdown
## [Page Title]

One sentence on what this page covers. One sentence on when to use it.

<br>

### Before You Start

1. Access or permissions needed
2. Information to have in hand (ticket number, customer, record IDs)
3. Anything that must be true first, such as "confirm no data freeze is in progress"

<br>

### Steps

1. First action. What to expect when it works.
2. Second action.
   - If you see `[specific error]`, stop and go to Troubleshooting.
3. Third action.

> One sentence on the step that does real damage if skipped, and what the damage is.
{.is-warning}

<br>

### Troubleshooting

| Symptom | Likely cause | What to do |
|---|---|---|
| | | |

<br>

### Who Owns What

| Item | Our team | Other team / vendor |
|---|---|---|
| | | |

<br>

### Related Pages

- [Page *What it covers*](/path)
{.links-list}

Source: [ticket key], [thread link]
````

<br>

## External developer / vendor page

**Audience:** vendors, integrators, and customer IT staff.

**Job:** replace a conversation. This is the page someone gets pointed to instead of receiving a one-off email, so pointing at it has to be enough. If it leaves a gap, they come back with a question, and the hand-holding it was meant to prevent happens anyway.

What an external page has to do:

1. **Scope itself in the first two sentences,** including what it does not cover and where that lives.
2. **Answer the follow-up question before it's asked.**
3. **State the line between what you provide and what the vendor builds.**
4. **Give real, copyable examples,** not descriptions of examples.
5. **Take a position.** Name the standard approach, the exception path, and the reason. A page that presents three options evenhandedly gets the vendor choosing whichever one they already have code for, and every downstream failure looks like a platform problem. Vendors follow reasoning; they route around unexplained rules.

````markdown
## [Page Title]

What this page covers, in one sentence. What it doesn't cover, in the next, with a link to the page that does.

<br>

### Requirements

1. What the vendor needs before starting
2. What they need to send you (and how, if it's sensitive)

<br>

### Standard Approach

Name the standard and give the reason in two or three sentences.

1. First step, with a copyable example:

   ```json
   { "example": "payload" }
   ```

2. Second step.

<br>

### Identifiers

Which identifier does which job. Vendors get this wrong more than anything else, so it gets its own section instead of a mention inside a step.

| Identifier | Used for | Never used for |
|---|---|---|
| | | |

<br>

### What We Provide and What You Build

| Item | We provide | You build and maintain |
|---|---|---|
| | | |

<br>

### Platform Best Practices

API limits, bulk operations, efficient queries, error handling. Frame these as requirements of building on the platform, not as preferences of your team.

<br>

### Related Pages

- [Page *What it covers*](/path)
{.links-list}
````
