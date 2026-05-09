---
description: "Show plugin sync state and telemetry transparency"
---

# /stratafy:status

Show the current state of the Stratafy connection, what's synced and how fresh, and what plugin telemetry is reported to the workspace admin.

## Process

### Step 1: Pin Clickatell Workspace

Call `select_workspace` with `workspaceId`: `"f06499c2-a2a8-4e7d-ad02-c66d6fd46873"`.

Provenance: `_source_plugin: "stratafy-core"`, `_source_command: "status"`, `_intent: "user_request"`, `_reason: "Pinning Clickatell workspace for status read"`.

### Step 2: Get User Context

Call `get_user_context` with `command_name: "status"`, `plugin_name: "stratafy-core"`.

### Step 3: Read Local Cache State

Check:

- `~/.stratafy/foundation.md` — exists? last modified time?
- `~/.clickatell/welcomed.json` — exists?

### Step 4: Render Status Block

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
- Command-run counts per plugin (e.g. "/stratafy:foundation run 4 times this week")
- Error counts when commands fail

What this plugin does NOT report:
- Your conversation content
- Per-employee activity
- Files you reference

For help: /clickatell:help
```

### Step 5: Surface Connection Issues

If `select_workspace` fails (auth, network, missing MCP config) → render:

```
✗ Could not connect to Stratafy
  Error: {{error message}}

  Try: /stratafy:status (to retry)
  Or contact {{named-human-contact}} for help.
```

Do NOT pretend everything is fine.

## Provenance Context

- `_source_plugin`: `"stratafy-core"`
- `_source_command`: `"status"`

## Rules

1. ALWAYS show telemetry transparency in the status output — non-negotiable for the trust pitch
2. ALWAYS surface auth/connection issues with concrete recovery steps
3. The "what this plugin reports / does NOT report" section is non-negotiable — must always be present
