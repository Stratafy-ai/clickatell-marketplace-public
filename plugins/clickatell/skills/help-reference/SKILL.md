---
name: "help-reference"
description: "Quick reference card for Claude at Clickatell. Use when the user runs /clickatell:help, asks how to turn the plugin off, asks what the plugin does, asks who to contact for help, or expresses confusion about what Claude knows."
---

# Help Reference

Static-ish reference card. The trust backstop.

## Trigger

- `/clickatell:help` invoked
- User asks how to turn the plugin off
- User asks what the plugin does
- User asks who to contact
- User expresses confusion about what's connected

## Output

Render this verbatim with substitutions for `{{named-human-contact}}`, `{{contact-email}}`, plugin version:

```markdown
# Claude at Clickatell — Help

## What this plugin does
Brings Clickatell's strategy and methodology into Claude.
Run /clickatell:welcome for the full intro.

## What's connected right now
- Clickatell foundation (refreshed weekly)

Run /stratafy:status to see sync state.

## What stays on your machine
Your conversations with Claude. The plugin reports plugin lifecycle
events and command-run counts, but never conversation content.
Run /stratafy:status for full transparency on what's transmitted.

## Who to ask
For anything Claude can't help with, or for questions about how the
plugin is managed, contact {{named-human-contact}} at {{contact-email}}.

## Useful commands
- /clickatell:welcome — re-run the introduction
- /clickatell:onboard-me — connect your role + active values to Claude (~10 min, re-runnable)
- /clickatell:foundation — show Clickatell's foundation
- /stratafy:status — what's synced and how fresh
```

## Voice

Quiet, factual, calm. This is a help card, not a sales pitch.

## Update Cadence

When named contact, version, or connected items change → push an update via the marketplace. Don't customize at runtime.
