# Clickatell Plugin

Branded surface for Clickatell employees on the Cowork marketplace.

## What it does

**Setup (run-once-ish):**
- `/clickatell:welcome` — First-run induction (~5 min). Explains what just got installed, why, the boundaries, and answers questions
- `/clickatell:onboard-me` — Role-aware onboarding after welcome (~10 min). Captures role, primary strategy, active values, and three concrete role-specific patterns. Re-runnable when role shifts.

**Weekly rhythm:**
- `/clickatell:lets-go` — Monday morning: pull the week's most relevant objectives/decisions/insights from the workspace; capture 1–3 commitments anchored to active values
- `/clickatell:call-it-a-week` — Friday afternoon: walk through Monday's commitments, capture what surfaced, give plugin feedback, optionally log material items to Stratafy, generate a team-channel-ready weekly wrap

**Always available:**
- `/clickatell:help` — Reference card with named human contact
- `/clickatell:foundation` — Show Clickatell's foundation, pulled live from Stratafy and cached locally for 7 days

## Sibling

Pairs with `stratafy-core`, which provides `/stratafy:status` (connection / telemetry transparency). The customer-facing employee surfaces all live here.

## Local Files

All files live inside the user's project folder so they're visible to Claude's file tools in both Claude Code CLI and Claude Desktop / Cowork. Resolve the project root via Bash: `PROJECT_ROOT="${CLAUDE_PROJECT_DIR:-$(pwd)}"`.

- `<project-root>/.clickatell/foundation.md` — cached foundation (7-day TTL)
- `<project-root>/.clickatell/welcomed.json` — welcome first-run timestamp + version
- `<project-root>/.clickatell/onboarded.json` — role-aware onboarding state (role, primary strategy, active values, weekly focus)
- `<project-root>/.clickatell/week-{iso-week}.json` — Monday's commitments, written by `/clickatell:lets-go`
- `<project-root>/.clickatell/wrap-{iso-week}.json` — Friday's wrap, written by `/clickatell:call-it-a-week`
- `<project-root>/.clickatell/welcome-questions.log` — unanswered welcome questions, append-only with consent
- `<project-root>/.clickatell/plugin-feedback.log` — plugin feedback captured by `/clickatell:call-it-a-week`, append-only with consent

> Earlier plugin versions wrote these to `~/.stratafy/foundation.md` and `~/.clickatell/welcomed.json`. Those locations are invisible to file tools in Claude Desktop / Cowork. Re-running `/clickatell:foundation` and `/clickatell:welcome` migrates state into the new project-scoped location.

## Provenance

Every MCP mutation includes `_source_plugin: "clickatell"`, the command name, and a short change-reasoning string.
