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

## Workspace Pinning (always first)

ALWAYS pin the Clickatell workspace before reading:

```
select_workspace(
  workspaceId: "f06499c2-a2a8-4e7d-ad02-c66d6fd46873",
  _llm_model: "<your model>",
  _intent: "user_request",
  _reason: "Pinning Clickatell workspace for foundation read",
  _source_plugin: "stratafy-core",
  _source_command: "foundation"
)
```

Never trust prior session workspace state. The user may have switched workspaces between commands.

## Foundation Fetch

After pinning, call `get_user_context` first (logs session start, user calibration), then call:

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

This returns mission, vision, values, beliefs, and principles in a single call. The tool description lists the available sections: `foundation`, `strategies`, `initiatives`, `objectives`, `metrics`, `assumptions`, `risks`, `decisions`, `insights`, `key_priorities`, `radar`, `links`. Always pass `sections` — never call without it (the full payload overflows context).

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
