---
name: close
description: Close a GitHub issue once its work is done, with a short closing comment, and settle any Sentry report it came from. Checks the issue's Acceptance against what actually shipped first; if nothing is left it closes straight away, and if something is unmet it lists exactly what and lets the user decide to close anyway or cancel. Use when the user types /close or says "close the issue".
argument-hint: "[issue number or URL; defaults to the issue this session worked on]"
---

# /close

Close the issue **only if nothing is left to work on**. PRs say `Part of #n`, never `Closes`, so this is the one place closing happens. Be fast: no warm-up, no recap, no "shall I?" when the answer is clear.

## 1. Find the issue

- Argument given: use it.
- No argument: the issue this session worked on (branch name suffix, the `Part of #n` PRs). Several candidates: ask which, in one line.

## 2. Check what is left, quietly, in one pass

```bash
gh issue view <n> --json title,state,body,comments
gh pr list --search "<n> in:body" --state all --json number,title,state,isDraft
```

Something is **left** when:

- An `## Acceptance` item is not met by what merged and deployed.
- A `Part of #<n>` PR is still open or draft.
- A merge has not deployed yet, or needs a ship step (app release, manual deploy) that has not happened.
- A rollout step is pending: a flag not on, a config not set, a migration not run.
- The latest `## Situation Development` entry is a question or wait on someone, with no answer yet.
- An `## Open questions` item still changes the work.
- A linked error report is still firing after the fix deployed.

Already closed: say so in one line and stop.

## 3a. Nothing left: close it now

No confirmation. Settle linked error reports (below), then:

```bash
gh issue close <n> --comment "<closing comment>"
```

Reply with one line: `Closed #<n> (<title>): <comment's first sentence>.`

## 3b. Something left: list it and wait

```
#<n> still has:
1. <what is unmet, one concrete line: the acceptance item, PR, flag, or person>
2. ...
Close anyway, or cancel?
```

Close only on "close" (note the open items in the comment). Anything else: do nothing.

## The closing comment

Short, plain English, no headings. First sentence: `Shipped in [#123](<url>): <what it does>.` If closed with items open, one more sentence naming them so the next reader knows it was closed knowingly.

## Error reports (Sentry or similar)

If the issue links an error report, settle it as part of closing:

- **The work fixed it:** resolve. If it fired again after the fix deployed, the fix did not hold: that is something left.
- **Not a bug, or deliberately not fixed:** archive / ignore.
- **Reduced but can still happen at a normal rate:** archive until it escalates.

Name what was done in the closing comment. If you cannot reach the tracker, that is something left, not a silent close.

## After closing

Append a line to the issue's `## Situation Development` if this session posted or decided something not logged yet.
