---
name: walkthrough
description: "Turn what was just verified in this session into square slide images (2184x2184 PNGs) to attach to Slack or a PR: one slide per case, each with the screenshot, the findings, and a provenance footer. Use whenever the user asks for slides, evidence, proof, screenshots-with-captions, or a walkthrough of a check that was just run. Three slides by default. It packages evidence; it never re-runs the verification."
argument-hint: "[n slides, default 3]"
---

# /walkthrough: verification evidence, as square slides

Take the screenshots and findings **already gathered in this session** and lay them out as square slide images the user can drag straight into Slack.

**The job is packaging, not proving.** Never re-run the verification, never go looking for new evidence, never invent a case that was not actually checked. Nothing verified yet? Say so and stop.

Why square: they read well as Slack and PR thumbnails, and one slide per case keeps each image self-explaining when dragged in on its own.

## Capture while working, not after

The broken state is the one that disappears first: someone fixes the data, and the "before" can never be photographed again. So any session verifying something should save shots at the moment (see `/drive`, "Screenshots as evidence") into `~/walkthrough/.work/captures/<YYYY-MM-DD>/`, with a `manifest.md` row per shot:

```markdown
| file | issue | shows | provenance |
| --- | --- | --- | --- |
| 271-empty-extras-before.png | 271 | Add to Cart greyed out, cart empty | PREVIEW |
| 271-empty-extras-after.png | 271 | same product, same cart, button live | PREVIEW |
```

`shows` states what is **visible in the image**, not what the code does. This skill starts by reading that manifest. If it is missing or thin, use what the session holds and say plainly that the evidence is partial.

## Default shape

- **Three slides**, one per case. More is a report. Exceed it only when the user names a number.
- **2184x2184 PNG** (a 1092 layout rendered at 2x so it stays sharp), written to `~/walkthrough/<issue>/<slug>-N-<case>.png`. No issue: `~/walkthrough/unfiled/<slug>/`.
- **Each slide:** mono eyebrow (`EVIDENCE 1 OF 3` + the issue ref), a short headline, the screenshot, two or three findings, a provenance footer.
- **No posting.** If a message is wanted, that is `/draft`.

## Themes

`template.html` carries five. Set one per `<section class="sq" data-theme="...">`; a deck often mixes them.

| Theme | Use for |
| --- | --- |
| `product` | a customer-facing site or app. Pass its real brand colors (`--brand`, `--paper`, `--ink`) read from its CSS, never invented |
| `shared` | a shared library. Brand-neutral on purpose: it means every consumer |
| `embed` | an embeddable widget, drawn on a host page |
| `admin` | an internal dashboard |
| `terminal` | APIs, CLIs, workers. Logs, SQL, test output |

The screenshot's own surface picks the theme, not the issue's scope.

## Phone shots

- **Phone left, findings right** (`.phone` in the template). A portrait phone across the full width wastes two thirds of the square.
- **Full screen gets a device frame** (`.device`, plus `android` for Android corners). Set `--ar` to the shot's width / height.
- **A crop gets no frame.** A bezel around a fragment reads as a broken phone.
- **Before and after on one slide** when the pair is the point: `.pair`. Same device, screen and data.

## Provenance must be true

The footer says where the screenshot came from:

- `LIVE PRODUCTION`: the real deployed system.
- `PREVIEW`: a branch preview or local dev server.
- Simulator/emulator shots name the device: `PREVIEW · iOS SIMULATOR, iPHONE 17 PRO`. Never let one pass as a real device.
- `SIMULATED RESPONSE` (add class `sim` to `.live`): any data was injected or stubbed. **Never dress a stubbed response as a real one.**

## Procedure

1. **Collect the cases:** screenshot path, headline, two or three findings (what the reader can see), provenance.
2. **Crop each screenshot** to the evidence, ending on a clean row boundary. Full-browser shots waste the square and are unreadable as thumbnails.
3. **Write one HTML per slide** from `template.html`, screenshots inlined as base64 `data:` URIs. Strip the template's comments first (they contain a literal `<section class="sq"`, which trips a naive regex). Add `:root{--side:1092px !important}`.
4. **Render in headless Chrome at 2x:**
   ```bash
   C="/Applications/Google Chrome.app/Contents/MacOS/Google Chrome"   # or google-chrome / chromium
   "$C" --headless=new --disable-gpu --hide-scrollbars --force-device-scale-factor=2 \
     --window-size=1092,1092 --virtual-time-budget=4000 \
     --screenshot="$OUT/<slug>-N-<case>.png" "file://$DIR/slide-N.html"
   ```
5. **Check before handing over:** every file is 2184x2184 (`sips -g pixelWidth -g pixelHeight` on macOS, `identify` elsewhere), the hashes differ (identical slides mean a template or crop went wrong), and open one to read the small text. Never upscale a smaller image to hit the size; it ships blurry.
6. List the paths in the reply.

## Traps

- **Every size in the template is written at 760px and scaled by `--u`.** A new rule uses `calc(<n> * var(--u))`, never a bare px, or it renders ~30% small at 1092.
- **The browser extension's zoom/screenshot capture is never the deliverable.** It is bound to the tab's viewport and can be downscaled. Use the headless render.
- **Let the shot take the leftover height.** If a crop overflows, the crop is too tall for its findings: check `.sq-inner` `scrollHeight > clientHeight` in a preview.
