---
description: "Display Clickatell's strategic foundation — mission, vision, values, beliefs, principles"
---

# /clickatell:foundation

Display Clickatell's foundation in clean markdown. Refreshed from Stratafy weekly.

## Process

### Step 1: Check Local Cache

Check if `~/.stratafy/foundation.md` exists and was modified less than 7 days ago. Use Bash with `"$HOME/.stratafy/foundation.md"` (quoted) — never rely on `~` literal expansion in tool args.

- If yes → display its contents and skip to Step 4.
- If no → continue to Step 2.

**Path discipline:** the cache lives at the absolute path `$HOME/.stratafy/foundation.md`. The `Write` tool does NOT expand `~` — passing `~/.stratafy/foundation.md` to Write creates a literal `~` folder in the current working directory. Always:

1. Resolve `$HOME` via Bash (`echo "$HOME"`) before Write
2. Pass the absolute path (e.g. `/Users/X/.stratafy/foundation.md`) to Write
3. `mkdir -p "$HOME/.stratafy"` first to ensure the directory exists

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

Assemble the response into the canonical document format (Step 4 template). Write to `~/.stratafy/foundation.md` with a sync timestamp footer.

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
4. Cache is 7 days — force refresh by deleting `~/.stratafy/foundation.md`
