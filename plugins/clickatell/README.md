# Clickatell Plugin

Branded onboarding for Clickatell employees on the Cowork marketplace.

## What it does

- `/clickatell:welcome` — First-run induction (~5 min). Explains what just got installed, why, the boundaries, and answers questions
- `/clickatell:help` — Always-available reference card with named human contact

## Sibling

Pairs with `stratafy-core`, which provides `/stratafy:foundation` and `/stratafy:status`. The welcome flow recommends both.

## Local Files

- `~/.clickatell/welcomed.json` — first-run timestamp + version
- `~/.clickatell/welcome-questions.log` — unanswered questions, append-only with consent

## Provenance

This plugin does not write to the Stratafy MCP directly — it relies on stratafy-core for that.
