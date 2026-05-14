---
description: "Display Clickatell's strategic foundation — mission, vision, values, beliefs, principles"
---

# /clickatell:foundation

Display Clickatell's foundation in clean markdown. Refreshed from Stratafy weekly.

## Process

### Step 1: Resolve the Project Root + Check Local Cache

The foundation cache lives in the user's project folder (not `$HOME`) so it's readable in Claude Desktop / Cowork — Cowork only exposes the selected project folder to file tools.

Resolve the project root via Bash, then check the cache freshness:

```bash
PROJECT_ROOT="${CLAUDE_PROJECT_DIR:-$(pwd)}"
CACHE_FILE="$PROJECT_ROOT/.clickatell/foundation.md"
if [ -f "$CACHE_FILE" ] && [ "$(find "$CACHE_FILE" -mtime -7 | wc -l)" -gt 0 ]; then
  echo "FRESH ($CACHE_FILE)"
  cat "$CACHE_FILE"
else
  echo "MISS ($CACHE_FILE)"
fi
```

- FRESH → display its contents and skip to Step 4.
- MISS → continue to Step 2.

In Claude Code CLI, `$CLAUDE_PROJECT_DIR` (or `pwd`) gives the project. In Cowork, the selected project folder's absolute path is what the model sees in the system prompt — use that. The Write tool does NOT expand `~` and does NOT resolve relative paths; **always pass the absolute path** built from `$PROJECT_ROOT`.

**Migration note:** if `~/.stratafy/foundation.md` exists from an older plugin version, treat it as a cache miss and re-fetch into the new project-relative location. The old home-directory location is invisible in Cowork.

### Step 2: Pin Clickatell + Load User Context (single call)

Call `get_user_context` with:

- `workspace_id`: `"f06499c2-a2a8-4e7d-ad02-c66d6fd46873"` (Clickatell — pins the session as a side effect)
- `command_name`: `"foundation"`
- `plugin_name`: `"clickatell"`
- `_llm_model`, `_intent: "user_request"`, `_reason`, `_source_plugin: "clickatell"`, `_source_command: "foundation"`

This sets the workspace, logs the session, and returns user calibration data in one round-trip. Do NOT call `select_workspace` separately — it's redundant when `get_user_context` carries `workspace_id`.

### Step 3: Fetch Foundation

Call `get_workspace_snapshot` with:

- `sections`: `["foundation"]` — REQUIRED, never call without `sections` (full payload overflows context)
- `_llm_model`, `_intent: "user_request"`, `_reason`, `_source_plugin: "clickatell"`, `_source_command: "foundation"`

This returns mission, vision, values, beliefs, and principles in a single call.

Assemble the response into the canonical document format (Step 4 template). Write to `$PROJECT_ROOT/.clickatell/foundation.md` (resolved in Step 1) with a sync timestamp footer. Ensure the directory exists first: `mkdir -p "$PROJECT_ROOT/.clickatell"`.

### Step 4: Display

```markdown
# Clickatell — Foundation

## Mission
[from Stratafy]

## Vision
[from Stratafy]

## Values
[from Stratafy]

## Beliefs
[from Stratafy]

## Principles
[from Stratafy]

---
*Synced from Stratafy on {{timestamp}}. Refreshed weekly.*
*Stratafy is the source of truth — to update, edit in Stratafy at https://app.stratafy.ai*
```

If foundation is empty or sparse, surface that honestly:

> *Clickatell's foundation in Stratafy is being populated — some sections are still in draft. For the complete picture, contact {{named-human-contact}}.*

NEVER fabricate content to fill gaps.

### Step 5: Offer Follow-up

End with: "Run `/stratafy:status` to see what else is synced, or `/clickatell:help` for support."

## Provenance Context

Every call includes:

- `_source_plugin`: `"clickatell"`
- `_source_command`: `"foundation"`
- `_change_reasoning`: brief description (e.g., "Initial foundation sync after 7-day cache expiry")

## Rules

1. ALWAYS pin the Clickatell workspace before reading — never trust prior session state
2. ALWAYS show the sync timestamp so the user knows freshness
3. NEVER fabricate foundation content if the workspace returns empty — surface honestly
4. Cache is 7 days — force refresh by deleting `<project-root>/.clickatell/foundation.md`
