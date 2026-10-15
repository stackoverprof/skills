---
name: draft
description: Write a short Slack message in the user's own voice and save it as a real Slack draft (slack_send_message_draft) in the right DM, channel or thread, or tighten one that is too long or sounds like an AI wrote it. Use when the user types /draft, asks for a Slack message, reply, update or announcement, or says a draft is "too long" or "sounds like AI". Draft only; never sends.
argument-hint: "[what to say, who it is for, and where] or [the draft to fix]"
---

# /draft

Write a Slack message in the user's voice and save it **as a Slack draft** for them to review and send. The usual complaint about AI drafts is "too long, and sounds like AI". This skill exists to get it right the first time.

Needs the Slack MCP connector. **Never send.** Only `slack_send_message_draft`, never `slack_send_message`.

## Where it goes

1. **A reply:** in that thread (`thread_ts` of the parent, copied verbatim from a read, never built from a displayed time).
2. **A DM:** find the user id with `slack_search_users`, draft to that id.
3. **A channel:** the one the conversation or the issue's `## Situation Development` names. Never guess a channel; if none is named, ask.

## Read the conversation first, every time

Before writing a word, read the latest messages where the draft will land (`slack_read_channel` or `slack_read_thread`). What the session "knows" is old; the other person may have written since.

- **They asked something not answered yet:** the draft answers it first, in one clause, then makes the new point.
- **Their message changes the point** (already done, different question): don't save a draft. Tell the user what they said and ask how to go on.
- **Nothing new:** draft as planned.

## Length (the rule that matters most)

- **Hard cap: 3 sentences** for a question or reply, aim for 2. An update may take two short paragraphs, never more.
- **One fact, not every fact.** The single detail that lets the reader answer. They can ask for the rest.
- **Lead with the situation**, not a framing question. "The old POS app looks abandoned." beats "Hey, what's the plan for the POS app?".
- **First person** ("I just moved it", "should I take it down?").
- **Questions carry their options inline** ("merge now, or wait till Monday?"). Several questions get numbered.
- **Offer your next move** when there is one ("I'll flip the stores one at a time").

Target shape:

> The old POS app (PR #1348) looks abandoned. I moved it to the new framework with everything else, but nobody could test it. Keep it going, or should I take it down?

## Spacing

One beat per paragraph (what happened, the problem, the ask), separated by a blank line, so it reads in chunks instead of a wall. Spacing splits the same 2 or 3 sentences; it never adds more. A one-liner stays one line.

- **Every PR or issue number is a link**, never a bare `#1234`.
- At most one `inline code` chip per message. Say the rest in words.

## The user's voice

Fill this in with 3 to 5 real messages you sent and liked. Real examples beat any description. Note your habits: casing, contractions, emoji, how you open and close, the language per person or channel.

```
> <paste a real message of yours here>
> <another one>
```

## No AI-speak

Cut on sight:

- Warm-up openers: "Quick update:", "Quick question,", "Just a heads up", "Hope this finds you well", "Just wanted to", "I wanted to reach out".
- Closers: "Happy to help", "Let me know if you have any questions".
- Words: "It's worth noting", "essentially", "comprehensive", "robust", "seamless", "leverage", "ensure", "streamline", "delve".
- Em dashes. Use commas or periods.
- Headings, bold labels, or bullet lists in a two-line message.
- Repeating what the reader just said. Over-thanking, over-apologising.

## Evidence slides

If the draft is about work that has `/walkthrough` slides, write it to read right with them attached ("slides below"), and end with a marker line after a blank line: `[attach walkthrough <issue>]`. Slack's API cannot attach files to a draft, so the user drags them in and deletes the marker before sending.

## Output

After saving, one line: where it is, with the link. Don't repeat the text unless Slack failed; if it did, say so and give the text to paste. If something needs the user's call, put a placeholder inside the draft (`[refund amount?]`).

## Check before saving

1. Count sentences. Over the cap? Cut.
2. Read it as the user saying it out loud to a teammate. Anything they'd never say, rewrite.
3. Scan for every phrase in the No AI-speak list and every em dash.
4. Every `#<number>` is a link.
