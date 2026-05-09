# Clickatell

Branded onboarding for Clickatell employees. Welcome script, help reference, named human contact.

## Commands

- `/clickatell:welcome` — First-run induction (~5 min). Re-runnable.
- `/clickatell:help` — Quick reference card. Always available.

## Voice

This plugin speaks in CLICKATELL'S voice, not Stratafy's. The welcome script and help reference are reviewed and signed off by Clickatell comms (Pieter or designated reviewer).

## Sibling

This plugin works alongside `stratafy-core`. The welcome flow recommends `/stratafy:foundation` and `/stratafy:status` (commands provided by stratafy-core).

## Local Files Written

- `~/.clickatell/welcomed.json` — first-run state
- `~/.clickatell/welcome-questions.log` — unanswered questions (with consent)

## What This Plugin Does NOT Do (v1)

- No identity capture
- No role/lens preferences
- No Clickatell-specific workflows (account-prep, pipeline-review, QBR, deal-review, compliance-check)
- No internal Clickatell connectors (Salesforce, data warehouse, wiki)

Each of these is a phase 2+ deferral, deliberately out of scope for v1.
