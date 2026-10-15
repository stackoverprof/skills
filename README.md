# skills

Claude Code skills for taking a task from "someone asked for something" to "shipped, verified, and reported back", with you mostly just approving.

```
see this <link to the request>
make a GitHub issue                        /ticket
ask me until everything is clear enough    /grill-me
use the browser/emulator to check it works /drive
review the PR, merge, smoke test on prod   /lgtm
reply back to whoever asked                /draft
attach screenshots as evidence             /walkthrough
close if nothing is left                   /close
```

| Skill | What it does |
| --- | --- |
| `/ticket` | Files one GitHub issue: short plain-English title, Problem / Sketch / Acceptance body, and a log of where the request came from so a later session knows where to report back. |
| `/grill-me` | Interviews you one question at a time until the plan has no open branches. From [mattpocock/skills](https://github.com/mattpocock/skills). |
| `/drive` | Runs the app and checks it like a human tester: screenshot, act, read logs, edit, check again. Chrome for web, Maestro for mobile, curl for APIs. |
| `/lgtm` | **L**et's **G**et **T**his **M**erged. Rates the PR's risk (T0 to T3), then reviews, settles findings, gets CI green, smoke tests the preview, merges, smoke tests prod, and reverts on a regression. |
| `/draft` | Writes a short Slack reply in your voice and saves it as a Slack **draft**. Never sends. |
| `/walkthrough` | Turns the screenshots from `/drive` into square evidence slides (PNG) for Slack or the PR. |
| `/close` | Checks the issue's Acceptance against what shipped, then closes it with a one-line comment, or lists what is still open. |

## Install

**As a plugin** (inside Claude Code):

```
/plugin marketplace add stackoverprof/skills
/plugin install workflow-skills@stackoverprof
```

Skills are then available as `/workflow-skills:ticket` and so on (Claude also picks them up automatically when your request matches).

**Or copy them in** for short names like `/ticket`:

```bash
git clone https://github.com/stackoverprof/skills
cp -r skills/skills/* ~/.claude/skills/        # for all your projects
# or: cp -r skills/skills/* <repo>/.claude/skills/   for one repo, shared with the team
```

## What they need

- [`gh`](https://cli.github.com) CLI, logged in: `/ticket`, `/lgtm`, `/close`
- [Claude in Chrome](https://claude.com/chrome) for web, [Maestro MCP](https://docs.maestro.dev) for mobile: `/drive`, `/lgtm`
- Slack MCP connector: `/draft`
- Google Chrome (headless render): `/walkthrough`
- Optional: Sentry MCP, for `/lgtm` step 8 and `/close`

## Make them yours

These are starting points. The parts worth editing first:

- **`/ticket`:** replace the example scope list with your codebase's surfaces.
- **`/draft`:** paste 3 to 5 real messages you sent into "The user's voice". That section does most of the work.
- **`/lgtm`:** list your repo's high-risk paths (migrations, payments, auth, CI) so tiering is mechanical, and add your deploy check.
- **`/drive`:** add an `apps/<name>.md` per app with how it starts, the env it needs, and a test login.

The best way to improve a skill: when Claude gets something wrong, tell it to fix the skill so it never happens again, with the reason.

## Conventions they share

- A PR links its issue with `Part of #123`, never `Closes #123`. One issue often takes several PRs, and `/close` closes it by hand once everything shipped.
- An issue's `## Situation Development` section logs the conversation behind it (who asked, where, every decision, with permalinks). `/ticket` starts it; the others append.

## License

MIT
