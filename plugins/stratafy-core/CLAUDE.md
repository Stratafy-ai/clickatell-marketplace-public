# Stratafy Core (Clickatell)

Universal Stratafy connector for Clickatell. Provides the platform-side surfaces that aren't customer-branded: connection state, sync transparency, telemetry visibility.

The customer-branded surfaces (welcome, help, foundation) live in the sibling `clickatell` plugin, which calls the Stratafy MCP through this connection.

This plugin pins itself to the Clickatell workspace (`f06499c2-a2a8-4e7d-ad02-c66d6fd46873`) on every command call. The user's prior workspace selection is irrelevant — every command call always pins Clickatell first.

## Commands

- `/stratafy:status` — Show what's synced, when, and what telemetry the plugin reports

## Provenance

Every MCP mutation includes:

- `_source_plugin`: "stratafy-core"
- `_source_command`: the command being run
- `_change_reasoning`: brief explanation of the change

## Privacy

This plugin reports plugin lifecycle events (install, command-run counts, errors, uninstall) to Stratafy. It does NOT report conversation content or per-employee activity. Run `/stratafy:status` to see exactly what's transmitted.

## Workspace Pin

Hardcoded for Clickatell. Future versions may read this from a workspace config file, but for v1 the simplest reliable approach is to pin on every command.
