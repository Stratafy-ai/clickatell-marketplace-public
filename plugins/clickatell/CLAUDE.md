# Clickatell

Branded onboarding for Clickatell employees. Welcome script, help reference, named human contact.

## Commands

- `/clickatell:welcome` — First-run induction (~5 min). Re-runnable.
- `/clickatell:help` — Quick reference card. Always available.
- `/clickatell:foundation` — Display Clickatell's foundation (mission, vision, values, beliefs, principles). Cached 7 days at `~/.stratafy/foundation.md`.

## Voice

This plugin speaks in CLICKATELL'S voice, not Stratafy's. The welcome script and help reference are reviewed and signed off by Clickatell comms (Pieter or designated reviewer).

## Workspace Pin

Pinned to Clickatell (`f06499c2-a2a8-4e7d-ad02-c66d6fd46873`). Every command that calls the MCP starts with `select_workspace` for that ID — including `/clickatell:welcome`, so any organic conversation after the welcome is anchored on Clickatell context regardless of what workspace the user had selected before.

## Sibling

This plugin works alongside `stratafy-core`, which provides the platform-side `/stratafy:status` for connection / telemetry transparency. The customer-facing commands (welcome, help, foundation) all live here.

## Local Files Written

- `~/.clickatell/welcomed.json` — first-run state
- `~/.clickatell/welcome-questions.log` — unanswered questions (with consent)

## What This Plugin Does NOT Do (v1)

- No identity capture
- No role/lens preferences
- No Clickatell-specific workflows (account-prep, pipeline-review, QBR, deal-review, compliance-check)
- No internal Clickatell connectors (Salesforce, data warehouse, wiki)

Each of these is a phase 2+ deferral, deliberately out of scope for v1.
