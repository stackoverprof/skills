---
name: ticket
description: File one new GitHub issue with a short `Scope: Title` headline and a body of Problem / Sketch / Open questions / Acceptance / Situation Development / References, logging where the request came from so a later session knows who to report back to. Use whenever the user asks to file, open, create, raise, or write up an issue or a ticket. CREATION ONLY, never edits or closes existing issues.
argument-hint: "[title or description of the problem] [owner/repo]"
---

# /ticket

File **one** new GitHub issue, well-titled and well-bodied, in the repo it belongs to. That is the whole job. No labels, no project boards, no branches, no PRs. If asked for those, say it is out of scope and do the ticket only.

```bash
gh issue create -R <owner>/<repo> --title "<title>" --body "<body>" --assignee @me
```

- **Always `--assignee @me`.** An unassigned issue is invisible to every "my open issues" sweep.
- Pick the repo from the issue's subject, not from where the session happens to run. Ask only if it is genuinely ambiguous.

## Title: `Scope: Short title`

**At most 40 characters, prefix included.** The first letter after `: ` is always a capital (`Api: Retire old routes`, never `Api: retire...`). Check that one character before running the command.

The title says **what happened or what changes, in words a manager understands on first read.**

- Name the effect, not the mechanism: `Api: App could erase payment records`, not `Api: Append-only bypass not admin-only`.
- No jargon, code words, or field/function names. If the only accurate word is jargon, the title is describing the fix instead of the problem.
- No vague containers (`follow-ups`, `review X`, `X support`). Name what the work delivers.
- Drop filler words, use `+` for "and". Push detail into the body.
- Say it out loud as if presenting it in a meeting. If you would have to explain a word, rewrite it.

| was | better |
| --- | --- |
| Api: Append-only bypass not admin-only | Api: App could erase payment records |
| Web: Adjuster discount swallows extras | Web: Discounts must not cut paid extras |
| Web: Harden double-charge safety nets | Web: Double charge on slow payment |
| Dash: Status badge reads as error | Dash: Closed store badge misleading |

### Scopes

Keep a small, fixed list of scopes for your codebase (5 to 8 is plenty) and put it here. **One scope per ticket, never combined.** Example list, replace with yours:

| Scope | Surface |
| --- | --- |
| `Web:` | the customer-facing site or app |
| `Api:` | backend, services, workers |
| `Dash:` | the internal admin dashboard |
| `Lib:` | shared packages used by several apps |
| `Misc:` | no product surface: CI, deps, docs, tooling (last resort) |

Rules that usually decide it:

1. **Scope by subject, not activity.** A CI fix that exists to serve the website is `Web:`, not `Misc:`.
2. **`Misc:` is the last resort.** Only when no product surface fits.
3. **Two fit? Pick where the problem is observed**, not where you guess the fix lands. Name the suspected root cause in the body.

### Review requests

When the ticket exists only because a teammate asked you to review their PR, title it `Scope: Review <what the PR does>`. The opening section names who asked and which PR. Acceptance is about the review (checked, tested, verdict posted, teammate told), never about owning the fix.

## Body

```
## Problem         (or ## Idea / ## Goal, whichever fits)
<what + why. If it came from someone else: "Proposed by <name> (<where>, <date>)".>

## Sketch          (optional: file paths, the small change, a snippet)
## Open questions  (optional: decisions to settle first)
## Acceptance      (verifiable done-criteria)
## Situation Development   (where this came from and where updates go)
## References      (optional: files, sibling issues, docs)
```

- The opening section is never optional.
- **Acceptance is the most valuable section.** Verifiable criteria let a teammate, or a fresh Claude session, ship the ticket unattended.
- **Sketch** is worth 30 seconds whenever you already know the file paths.
- A small idea may be one line. Never pad a small ticket into a fake spec.

## Situation Development: the ticket's return address

A session that picks the issue up later has no memory of the chat that started it, so it cannot answer the one question it will certainly reach: *where does the "this is done" message go?* Log it here.

```
## Situation Development

**Channel:** [#team-frontend](<channel link>), where updates on this ticket get posted.

**Thu, 17 Sep**
- [Alex](<message permalink>): asked for scheduled orders, pickup only.
- [Me](<message permalink>): agreed 7 days ahead, in the channel.
```

1. **Channel on its own first line**, even when the request came by DM. If none is agreed yet: `**Channel:** not agreed yet. Ask before posting any update.`
2. **Append, never rewrite.** It is a log.
3. **Every related message**, not just the first: scope changes, decisions, rejections.
4. **Permalink or it did not happen.** The person's name is the link.
5. **One sentence per line.** An index to the conversation, not a transcript.

Omit the section only when there was no conversation behind the ticket at all. Other skills that post about the issue (`/draft`, `/close`) append their own lines.

## After filing

If someone else proposed it, offer to tell them with the issue link. Draft by default; send only on an explicit yes.

## Output

The final title, the repo, and the issue URL. Nothing else. If the scope could have gone another way, one line saying which two you weighed.
