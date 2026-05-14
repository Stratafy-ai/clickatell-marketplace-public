# Clickatell Plugin

Branded surface for Clickatell employees on the Cowork marketplace.

## What it does

- `/clickatell:welcome` — First-run induction (~5 min). Explains what just got installed, why, the boundaries, and answers questions
- `/clickatell:onboard-me` — Role-aware onboarding after welcome (~10 min). Captures the user's role, primary strategy, active values, and three concrete role-specific patterns. Re-runnable when role shifts.
- `/clickatell:help` — Always-available reference card with named human contact
- `/clickatell:foundation` — Display Clickatell's foundation (mission, vision, values, beliefs, principles), pulled live from Stratafy and cached locally for 7 days

## Sibling

Pairs with `stratafy-core`, which provides `/stratafy:status` (connection / telemetry transparency). The customer-facing employee surfaces all live here.

## Local Files

All files live inside the user's project folder so they're visible to Claude's file tools in both Claude Code CLI and Claude Desktop / Cowork. Resolve the project root via Bash: `PROJECT_ROOT="${CLAUDE_PROJECT_DIR:-$(pwd)}"`.

- `<project-root>/.clickatell/foundation.md` — cached foundation (7-day TTL)
- `<project-root>/.clickatell/welcomed.json` — welcome first-run timestamp + version
- `<project-root>/.clickatell/onboarded.json` — role-aware onboarding state (role, primary strategy, active values, weekly focus)
- `<project-root>/.clickatell/welcome-questions.log` — unanswered questions, append-only with consent

> Earlier plugin versions wrote these to `~/.stratafy/foundation.md` and `~/.clickatell/welcomed.json`. Those locations are invisible to file tools in Claude Desktop / Cowork. Re-running `/clickatell:foundation` and `/clickatell:welcome` migrates state into the new project-scoped location.

## Provenance

Every MCP mutation includes `_source_plugin: "clickatell"`, the command name, and a short change-reasoning string.
