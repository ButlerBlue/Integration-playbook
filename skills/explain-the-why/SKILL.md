---
name: explain-the-why
description: "Explain a technical fix or root cause to a colleague or internal partner so they can handle it themselves next time. Use when asked to 'help me explain this', 'write this up for [person or team]', or 'respond to this thread', and whenever drafting a reply to a teammate's technical question, including chat replies and emails that explain a cause or a fix, even if the word 'explain' never appears."
---

# Explain the Why

Help the user answer a colleague so the fix is clear **and** the colleague walks away understanding the why well enough to handle it next time. The why is the point: the goal is to stop being the only person who can resolve this kind of issue. But the colleague should build that understanding from one clean explanation, not have it quizzed out of them or buried in an essay.

## What good looks like

Lead with the answer. Then the why, as a short, plain mechanism. Then the action. Then what to expect. In that order. Someone skimming gets the fix from the first line and the reasoning from the next two.

Use this shape:

1. **Answer / fix:** stated plainly, first.
2. **Why:** the mechanism and cause in one or two sentences. Plain language, with a field, setting, or object name in `inline code` when it anchors the point.
3. **What to do:** the concrete action.
4. **What to expect:** timing, what to watch for, what's still open.

## Hard rules

- **Never quiz them into the answer.** No "what do you think happens?", no "you're closer than you think", no making someone guess the thing they came for. State it, then explain it. People under time pressure need the fix, not a Socratic detour, and the guessing game reads as withholding.
- **No filler praise.** Cut "great point" and "thanks so much, super helpful." One specific, genuine credit is fine when someone actually moved the problem forward ("that date range was the key piece, thanks"). Praise on every line reads as padding and undercuts the author.
- **Link, don't dump.** Point to the documentation page for the deep mechanism and the full query. Pasting long code blocks into a chat reply as a teaching device buries the why. The durable version of the why belongs in a doc people can come back to, not in a message they scroll past.
- **Calibrate to the person and the urgency.** A peer who clearly gets the gist needs the fix and one line of why. Someone who's lost needs more. When unsure, give the fix first and offer the deeper why as an optional follow-up.
- **Stay grounded.** If the root cause isn't confirmed, say what's still unverified. Don't write a tidy explanation that papers over a gap just to sound authoritative.
- **Never frame the person as inexperienced,** even if it's factually true. If something is hard, the material is complex. Skip "since you're new to this."

## How the why becomes theirs

The colleague owns the why when they get a clean causal chain once, in plain terms, plus a pointer to the doc for depth: mechanism, cause, fix, link. That's what they reread, reuse, and repeat to the next person. Quizzing makes them depend on you to run the session. Essays make them tune out. A crisp explanation plus a reference makes the knowledge portable, which is the whole goal.

## Before drafting, confirm

Ask only what the conversation hasn't already answered:

1. Who is this going to, and how technical are they?
2. How urgent is it? Do they need just the fix now, or the full why?
3. Is the root cause confirmed, or still being validated?

## Channel

- **Chat (Slack, Teams, etc.):** conversational and short. Never document-style.
- **Email:** short paragraphs, numbered action items at the end if it runs long, and a one-paragraph summary at the bottom of anything lengthy.

## Example

**Before** (quizzing, essay, praise):

> You're closer than you think! Pull the last-modified dates for those records and note the range. Now re-read the sync filter. In plain English, what does a record have to do before it gets considered? And if it hasn't been touched since before the sync was turned on, what do you think happens? Tell me what you're thinking and I think you'll have your answer.
>
> *[followed by two 30-line queries and four paragraphs of background]*

**After** (answer, why, action, expectation):

> The sync is incremental: it only picks up a record if `LastModifiedDate` is newer than the last sync run. The ~500 missing members haven't been edited since before the sync was turned on for their group, so it never had a reason to look at them. That's also why the error report is empty. It never tried, so nothing failed.
>
> Fix: run an update on those records so `LastModifiedDate` moves to now. No field values need to change. Leave out the ones being corrected by hand.
>
> Give it about two hours, then they should show up downstream. Anything still missing after that, send my way and we'll dig in.
>
> Full mechanism and the query: [doc link]
