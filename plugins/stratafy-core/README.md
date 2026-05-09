# stratafy-core

Universal Stratafy connector. In the Clickatell marketplace, this plugin provides the platform-side surfaces (connection state, sync transparency, telemetry visibility). The customer-branded surfaces (welcome, help, foundation) live in the sibling `clickatell` plugin.

## What it does

- `/stratafy:status` — Sync state and telemetry transparency

## Provenance

On every mutation tool call (`create_*`, `update_*`, `archive_*`, `link_*`, `record_*`), this plugin always includes:

- `_source_plugin`: "stratafy-core"
- `_source_command`: the command being run
- `_change_reasoning`: 1–2 sentences explaining WHY this change is being made

The system handles approval automatically based on the user's workspace role.

## Privacy

This plugin reports plugin lifecycle events (install, command-run counts, errors, uninstall) to Stratafy. It does NOT report conversation content or per-employee activity. The `/stratafy:status` command shows exactly what's transmitted.

## Workspace Pin

The Clickatell workspace ID (`f06499c2-a2a8-4e7d-ad02-c66d6fd46873`) is hardcoded in the skills. Every command call pins this workspace before any read — the user's prior workspace selection is ignored.
