---
description: "Always-available reference card for Claude at Clickatell — what it does, how to turn it off, who to ask"
---

# /clickatell:help

The trust backstop. Always-available reference card.

## Process

Render the following markdown verbatim, with current plugin version and named human contact substituted in:

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
- /clickatell:foundation — show Clickatell's foundation
- /stratafy:status — what's synced and how fresh
```

## Rules

1. KEEP this output static-ish — when the named contact, version, or connected items change, push an update via the marketplace
2. NEVER add features, marketing language, or persona — this is a reference card, not a sales pitch
3. ALWAYS surface the named human contact — this command exists primarily so they know who to ask
