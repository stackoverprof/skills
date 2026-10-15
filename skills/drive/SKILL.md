---
name: drive
description: Run the app and check it like a human tester, then iterate on what you see. Starts the app, then drives it (Claude in Chrome for web, Maestro MCP for an iOS Simulator or Android emulator, curl for an API) in one loop of screenshot, read the screen, act, read logs, edit, reload, check again. Use whenever asked to run, start or serve the app, when a task needs to see or click through a UI to build, debug or verify it, or when the user says "test it on the emulator", "click through it", "reproduce this", or asks for screenshots.
---

# Drive the app

Work like a human tester with the code open: look at the screen, act on it, read the logs, change the code, look again. **Never call a UI change done from the diff alone.**

## 1. Start the app

Find how the project starts (README, `package.json` scripts, `Makefile`, `docker-compose.yml`). Start it in the background and keep its output file, so you can read the logs later.

Tip: once you know how each app in your repo starts, write it down in `apps/<name>.md` next to this file (start command, env it needs, test login, how to prove it talks to the backend) and link them here. Then no session has to rediscover it.

## 2. Pick the driver

| Task is about | Driver | Device |
| --- | --- | --- |
| A web page or URL | Claude in Chrome (`mcp__claude-in-chrome__*`) | Chrome tab |
| A mobile app on iOS | Maestro MCP (`mcp__maestro__*`) | iOS Simulator |
| A mobile app on Android | Maestro MCP | Android emulator |
| Mobile, platform not stated | Maestro MCP | Loop on one, confirm on the other before done |
| An API, no screen | `curl` + the server log | none |

Maestro MCP setup, once: `brew install mobile-dev-inc/tap/maestro`, then `claude mcp add maestro -s user -- "$(which maestro)" mcp`, and restart the session.

## 3. The loop

1. **Look.** Web: screenshot or `read_page`. Mobile: `list_devices`, then `inspect_screen` before every action (text, ids, bounds), and `take_screenshot` when layout or color matters.
2. **Act.** Click, type, navigate. On mobile, `run` with inline YAML; one call can hold a whole path:
   ```yaml
   appId: com.example.app
   ---
   - tapOn: 'Order now'
   - tapOn: { id: 'store-card', index: 0 }
   - inputText: '5555555'
   - scrollUntilVisible: { element: 'Cart', direction: DOWN }
   ```
3. **Read logs** after anything unexpected: browser console and network, the dev server output, `adb logcat`, or `xcrun simctl spawn booted log show`.
4. **Edit the code.** Hot reload, then look again.
5. **Done means:** the screen shows the intended result on every platform the task names, and the logs have no new errors.

When a path is worth keeping, save it as a test (a Maestro flow in `.maestro/`, a Playwright spec).

## 4. Screenshots as evidence

Capture at three moments only:

1. **The problem reproducing**, before any fix. It usually cannot be retaken later, so if you take only one, take this one.
2. **The same screen fixed.** Same page, viewport and data, so the pair only differs where the fix does.
3. **Anything surprising** found on the way.

Save them to a file (not just the in-chat preview), at full resolution, cropped to the thing being shown. Name the device on anything you share ("Android 16 emulator, Pixel 8"); never imply an emulator is a real device. `/walkthrough` turns these into slides.

## Don't

- Don't claim a UI change works without a screenshot taken after the change.
- Don't tap by coordinates when text or an id exists.
- Don't run two dev servers on the same port.
- Don't fake data in a way the evidence would hide. If anything is stubbed, say so.
