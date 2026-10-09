---
name: excellent-pr
description: "Write or update a PR body for any change: ticket, background, user impact, evidence, reviewer-question notes. UI changes get before/after screenshots or video. Defects spotted on a touched surface go into a stacked PR."
---

# Excellent PR

Excellence principles, applied to every change:

- Scope small enough to do it well.
- Push back when clarity, craft, performance, or trust is at risk. Say so in chat with a concrete reason and an alternative.
- Leave every surface better than you found it.

A PR body is short because a long one is a _slop grenade_: the reviewer loses the change inside the description.

## Body

```markdown
[PROJ-123](<ticket url>)

## Background

<why this change exists, one to three sentences>

## User impact

<what gets better for the user, e.g. long messages are easier to read>

## Evidence

<how the change was verified, e.g. tested in Chromium>

<screenshots, video, or test output>

## Implementation notes

- <only what makes a reviewer ask a question>
```

- Implementation notes hold the unconventional hack, the workaround for a limitation, the surprising choice. Omit the section when nothing qualifies.
- Evidence for a UI or style change is before and after captured in a real browser, both shown, motion as video. If the project documents how to run and capture the app, read that first. For other changes it is the failing and passing test run.
- Follow the repository's own PR conventions for title, ticket references, and attribution.
- Run a humanizing pass over the prose so it reads like a person wrote it.

Done when the reviewer grasps what changed and why in under a minute, each sentence of User impact is visible in the evidence (after looking at every image yourself and confirming "before" shows the old code), and every implementation note answers a question they would otherwise ask.

## Defects found on the way

Fix them in a stacked PR: base is the current PR's branch, same ticket unless told otherwise, its own evidence, its own short body. Keep the original PR focused and tell the author in chat.

Done when the stacked PR is open and the original body mentions nothing of it.
