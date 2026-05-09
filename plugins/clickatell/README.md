# Clickatell Plugin

Branded surface for Clickatell employees on the Cowork marketplace.

## What it does

- `/clickatell:welcome` — First-run induction (~5 min). Explains what just got installed, why, the boundaries, and answers questions
- `/clickatell:help` — Always-available reference card with named human contact
- `/clickatell:foundation` — Display Clickatell's foundation (mission, vision, values, beliefs, principles), pulled live from Stratafy and cached locally for 7 days

## Sibling

Pairs with `stratafy-core`, which provides `/stratafy:status` (connection / telemetry transparency). The customer-facing employee surfaces all live here.

## Local Files

- `~/.stratafy/foundation.md` — cached foundation (refreshed weekly)
- `~/.clickatell/welcomed.json` — first-run timestamp + version
- `~/.clickatell/welcome-questions.log` — unanswered questions, append-only with consent

## Provenance

Every MCP mutation includes `_source_plugin: "clickatell"`, the command name, and a short change-reasoning string.
