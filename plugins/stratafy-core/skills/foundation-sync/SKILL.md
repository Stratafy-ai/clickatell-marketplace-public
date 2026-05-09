---
name: "foundation-sync"
description: "Manages the local cache of Clickatell's foundation (mission, vision, values, beliefs, principles). Use when the user runs /stratafy:foundation, when foundation context is relevant to their request, or when ~/.stratafy/foundation.md is missing or older than 7 days. Pins Clickatell workspace, calls get_workspace_snapshot with sections: [\"foundation\"], writes the result to disk, displays formatted output."
---

# Foundation Sync

This skill manages the local cache of Clickatell's foundation.

## Trigger

Use when:

- The user runs `/stratafy:foundation`
- Foundation context is relevant to a request (the user asks "what does Clickatell value?", "what's our mission?", etc.)
- `~/.stratafy/foundation.md` is missing or older than 7 days

## Workspace Pinning + User Context (single call)

Combine workspace pin and user-context load in ONE call by passing `workspace_id` to `get_user_context`. This sets the session workspace AND logs session start AND returns user calibration data — saving a redundant `select_workspace` round-trip:

```
get_user_context(
  workspace_id: "f06499c2-a2a8-4e7d-ad02-c66d6fd46873",
  command_name: "foundation",
  plugin_name: "stratafy-core",
  _llm_model: "<your model>",
  _intent: "user_request",
  _reason: "Loading user context and pinning Clickatell workspace for foundation read",
  _source_plugin: "stratafy-core",
  _source_command: "foundation"
)
```

The `workspace_id` parameter on `get_user_context` validates access and sets the session workspace as a side effect. Tool source: `layers/mcp/server/tools/personal-intelligence-tools.ts`. Always pass `workspace_id` — never trust prior session state, because the user may have switched workspaces between commands.

Do NOT call `select_workspace` separately. It is redundant when `get_user_context` carries `workspace_id`.

## Foundation Fetch

After the pin-and-context call above, fetch the foundation:

```
get_workspace_snapshot(
  sections: ["foundation"],
  _llm_model: "<your model>",
  _intent: "user_request",
  _reason: "Refreshing Clickatell foundation cache",
  _source_plugin: "stratafy-core",
  _source_command: "foundation"
)
```

This returns mission, vision, values, beliefs, and principles in a single call. Available sections: `foundation`, `strategies`, `initiatives`, `objectives`, `metrics`, `assumptions`, `risks`, `decisions`, `insights`, `key_priorities`, `radar`, `links`. Always pass `sections` — never call without it (the full payload overflows context).

Assemble the foundation portion of the response into the canonical document format. Write to `~/.stratafy/foundation.md`.

## Cache Logic

- File exists AND mtime < 7 days → display from cache
- File missing OR mtime ≥ 7 days → re-fetch, write, display

User can force refresh by deleting the cache file.

## Honest Empty State

If foundation is empty or sparse, surface that honestly:

> Clickatell's foundation in Stratafy is being populated — some sections are still in draft. For the complete picture, contact {{named-human-contact}}.

NEVER fabricate content to fill gaps.

## Sync Timestamp

Always include the sync timestamp in the displayed output:

> *Synced from Stratafy on {{timestamp}}. Refreshed weekly.*

## Provenance

- `_source_plugin`: `"stratafy-core"`
- `_source_command`: `"foundation"`
- `_change_reasoning`: e.g., `"Foundation cache miss after 7-day expiry"`

## Local Files

Reads / writes:

- `~/.stratafy/foundation.md` — cached foundation document

**Path expansion is critical.** Always expand `~` to `$HOME` before writing. The `Write` tool does NOT auto-expand `~` — passing `~/.stratafy/foundation.md` to Write creates a literal `~` directory in cwd. Either:

- Resolve via Bash first: `mkdir -p "$HOME/.stratafy"` then `Write` to the absolute path returned by `echo "$HOME/.stratafy/foundation.md"`, OR
- Always use the absolute path string (e.g. `/Users/leonardcremer/.stratafy/foundation.md`) — which means the skill must read `$HOME` at runtime.

Recommended: shell out to Bash to ensure the directory exists, capture the absolute path, then Write to that absolute path. The Bash freshness check should also use `"$HOME/.stratafy/foundation.md"` (quoted) for consistency.
