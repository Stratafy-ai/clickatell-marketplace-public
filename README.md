# Clickatell Marketplace

Cowork plugins distributed to Clickatell employees. Two plugins:

- **stratafy-core** — Stratafy platform connector. Provides `/stratafy:status` (connection state, sync freshness, telemetry transparency).
- **clickatell** — Customer-branded employee surface. Provides setup (`/clickatell:welcome`, `/clickatell:onboard-me`), weekly rhythm (`/clickatell:lets-go`, `/clickatell:call-it-a-week`), and always-available (`/clickatell:help`, `/clickatell:foundation`).

## Branches

- `main` — Production, syncs to Clickatell's live marketplace
- `staging` — Pilot group testing before promotion

## Update Workflow

| Change | Process |
|---|---|
| Routine fixes (typos, MCP config, status output) | PR into `main`, auto-syncs |
| Welcome script changes | PR into `main`, requires Clickatell admin or designated comms reviewer approval |
| Pilot testing | Push to `staging`, pilot group's environment connected to staging marketplace |

## Local Files Written

All inside the user's project folder so they're visible to file tools in both Claude Code CLI and Claude Desktop / Cowork. Resolve the project root via `${CLAUDE_PROJECT_DIR:-$(pwd)}` in Bash.

- `<project-root>/.clickatell/foundation.md` — Clickatell foundation cache (7-day TTL)
- `<project-root>/.clickatell/welcomed.json` — welcome first-run timestamp + script version
- `<project-root>/.clickatell/onboarded.json` — role-aware onboarding state (role, primary strategy, active values, weekly focus)
- `<project-root>/.clickatell/week-{iso-week}.json` — Monday's commitments per ISO week
- `<project-root>/.clickatell/wrap-{iso-week}.json` — Friday's wrap per ISO week
- `<project-root>/.clickatell/welcome-questions.log` — unanswered welcome questions (with consent)
- `<project-root>/.clickatell/plugin-feedback.log` — weekly plugin feedback captured by call-it-a-week (with consent)

Earlier plugin versions cached to `~/.stratafy/foundation.md` and `~/.clickatell/welcomed.json` under `$HOME`. Cowork only exposes the selected project folder to file tools, so those legacy locations are invisible and cache writes silently failed to round-trip. v0.3.0+ scopes everything inside the project folder.

## Provenance

On every mutation tool call, plugins always include:

- `_source_plugin`: the calling plugin name
- `_source_command`: the command being run
- `_change_reasoning`: 1–2 sentences explaining WHY this change is being made

The system handles approval automatically based on the user's workspace role.

## Privacy

These plugins report plugin lifecycle events (install, command-run counts, errors, uninstall) to Stratafy. They do NOT report conversation content or per-employee activity. Run `/stratafy:status` to see exactly what's transmitted.
