---
description: "Display Clickatell's strategic foundation — mission, vision, values, beliefs, principles"
---

# /stratafy:foundation

Display Clickatell's foundation in clean markdown. Refreshed from Stratafy weekly.

## Process

### Step 1: Check Local Cache

Check if `~/.stratafy/foundation.md` exists and was modified less than 7 days ago.

- If yes → display its contents and skip to Step 4.
- If no → continue to Step 2.

### Step 2: Pin Clickatell Workspace

Call `select_workspace` with:

- `workspaceId`: `"f06499c2-a2a8-4e7d-ad02-c66d6fd46873"` (Clickatell)
- `_llm_model`: your model ID
- `_intent`: `"user_request"`
- `_reason`: `"Pinning Clickatell workspace for foundation read"`
- `_source_plugin`: `"stratafy-core"`
- `_source_command`: `"foundation"`

### Step 3: Fetch Foundation

Call `get_user_context` first (logs session start, user calibration).

Then call `get_workspace_snapshot` with:

- `sections`: `["foundation"]` — REQUIRED, never call without `sections` (full payload overflows context)
- `_llm_model`, `_intent: "user_request"`, `_reason`, `_source_plugin: "stratafy-core"`, `_source_command: "foundation"`

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

- `_source_plugin`: `"stratafy-core"`
- `_source_command`: `"foundation"`
- `_change_reasoning`: brief description (e.g., "Initial foundation sync after 7-day cache expiry")

## Rules

1. ALWAYS pin the Clickatell workspace before reading — never trust prior session state
2. ALWAYS show the sync timestamp so the user knows freshness
3. NEVER fabricate foundation content if the workspace returns empty — surface honestly
4. Cache is 7 days — force refresh by deleting `~/.stratafy/foundation.md`
