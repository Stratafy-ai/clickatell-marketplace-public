---
name: "status-check"
description: "Reports on the freshness of Stratafy-synced context, the plugin's connection state, and what telemetry is reported to the workspace admin. Use when the user runs /stratafy:status or asks about sync state, connection, what the plugin currently knows, or what data is transmitted."
---

# Status Check

Reports plugin sync state and telemetry transparency.

## Trigger

Use when:

- User runs `/stratafy:status`
- User asks about sync state, connection state, what data is transmitted
- User expresses concern about privacy or what the plugin "sees"

## Workspace Pinning

Pin Clickatell workspace before any MCP read:

```
select_workspace(workspaceId: "f06499c2-a2a8-4e7d-ad02-c66d6fd46873", ...)
```

Provenance: `_source_plugin: "stratafy-core"`, `_source_command: "status"`.

## Status Block

Render this format:

```
Stratafy plugin status:

✓ Connected to Stratafy as {{user.email}}
✓ Workspace: Clickatell
✓ Foundation synced {{n}} days ago
  Next auto-refresh: in {{7-n}} days
  Force refresh: /stratafy:foundation

Plugin version: stratafy-core {{version}}

What this plugin reports to your workspace admin:
- Plugin install / activation / uninstall events
- Command-run counts (e.g. "/stratafy:foundation run 4 times this week")
- Error counts when commands fail

What this plugin does NOT report:
- Your conversation content
- Per-employee activity
- Files you reference

For help: /clickatell:help
```

## Telemetry Transparency

The "what this plugin reports / does NOT report" section is non-negotiable. It must always be present. The trust pitch in `/clickatell:welcome` depends on this surface being honest and visible.

## Failure Modes

If `select_workspace` fails (auth, network) → render:

```
✗ Could not connect to Stratafy
  Error: {{error message}}

  Try: /stratafy:status (to retry)
  Or contact {{named-human-contact}} for help.
```

Do NOT pretend everything is fine.

## Provenance

- `_source_plugin`: `"stratafy-core"`
- `_source_command`: `"status"`
