---
description: "Show plugin sync state and telemetry transparency"
---

# /stratafy:status

Show the current state of the Stratafy connection, what's synced and how fresh, and what plugin telemetry is reported to the workspace admin.

## Process

### Step 1: Pin Clickatell + Load User Context (single call)

Call `get_user_context` with:

- `workspace_id`: `"f06499c2-a2a8-4e7d-ad02-c66d6fd46873"` (Clickatell — pins the session as a side effect)
- `command_name`: `"status"`
- `plugin_name`: `"stratafy-core"`
- `_llm_model`, `_intent: "user_request"`, `_reason`, `_source_plugin: "stratafy-core"`, `_source_command: "status"`

Do NOT call `select_workspace` separately — `get_user_context` with `workspace_id` does both jobs.

### Step 3: Read Local Cache State

The clickatell plugin caches state inside the user's project folder (NOT `$HOME`) so files are visible in Claude Desktop / Cowork. Resolve via Bash:

```bash
PROJECT_ROOT="${CLAUDE_PROJECT_DIR:-$(pwd)}"
```

Then check:

- `$PROJECT_ROOT/.clickatell/foundation.md` — exists? last modified time?
- `$PROJECT_ROOT/.clickatell/welcomed.json` — exists?

If `~/.stratafy/foundation.md` or `~/.clickatell/welcomed.json` exist from older plugin versions, flag them as legacy locations and recommend re-running `/clickatell:foundation` and `/clickatell:welcome` to migrate.

### Step 4: Render Status Block

```
Stratafy plugin status:

✓ Connected to Stratafy as {{user.email}}
✓ Workspace: Clickatell
✓ Foundation synced {{n}} days ago
  Next auto-refresh: in {{7-n}} days
  Force refresh: /clickatell:foundation

Plugin version: stratafy-core {{version}}

What this plugin reports to your workspace admin:
- Plugin install / activation / uninstall events
- Command-run counts per plugin (e.g. "/clickatell:foundation run 4 times this week")
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
