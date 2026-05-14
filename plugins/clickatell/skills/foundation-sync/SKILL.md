---
name: "foundation-sync"
description: "Manages the local cache of Clickatell's foundation (mission, vision, values, beliefs, principles). Use when the user runs /clickatell:foundation, when foundation context is relevant to their request, or when <project-root>/.clickatell/foundation.md is missing or older than 7 days. Pins Clickatell workspace, calls get_workspace_snapshot with sections: [\"foundation\"], writes the result to disk in the user's project folder so file tools can read it, displays formatted output."
---

# Foundation Sync

This skill manages the local cache of Clickatell's foundation. The cache lives **inside the user's project folder** (not the user's home directory) so file tools in Claude Desktop / Cowork can read it — Cowork only exposes the selected project folder to the model.

## Trigger

Use when:

- The user runs `/clickatell:foundation`
- Foundation context is relevant to a request (the user asks "what does Clickatell value?", "what's our mission?", etc.)
- `<project-root>/.clickatell/foundation.md` is missing or older than 7 days

## Workspace Pinning + User Context (single call)

Combine workspace pin and user-context load in ONE call by passing `workspace_id` to `get_user_context`. This sets the session workspace AND logs session start AND returns user calibration data — saving a redundant `select_workspace` round-trip:

```
get_user_context(
  workspace_id: "f06499c2-a2a8-4e7d-ad02-c66d6fd46873",
  command_name: "foundation",
  plugin_name: "clickatell",
  _llm_model: "<your model>",
  _intent: "user_request",
  _reason: "Loading user context and pinning Clickatell workspace for foundation read",
  _source_plugin: "clickatell",
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
  _source_plugin: "clickatell",
  _source_command: "foundation"
)
```

This returns mission, vision, values, beliefs, and principles in a single call. Available sections: `foundation`, `strategies`, `initiatives`, `objectives`, `metrics`, `assumptions`, `risks`, `decisions`, `insights`, `key_priorities`, `radar`, `links`. Always pass `sections` — never call without it (the full payload overflows context).

Assemble the foundation portion of the response into the canonical document format. Write to `<project-root>/.clickatell/foundation.md`, where `<project-root>` is the user's project directory (see *Resolving the project root* below).

## Cache Logic

- File exists AND mtime < 7 days → display from cache
- File missing OR mtime ≥ 7 days → re-fetch, write, display

User can force refresh by deleting the cache file.

### Migration from the previous location

Earlier versions of this plugin cached to `~/.stratafy/foundation.md` (the user's home directory). That file is invisible to the Write/Read tools in Claude Desktop / Cowork — Cowork only exposes the selected project folder. If the old file exists, treat it as a cache miss and re-fetch into the new project-relative location. Optionally delete the old file with the user's permission.

## Honest Empty State

If foundation is empty or sparse, surface that honestly:

> Clickatell's foundation in Stratafy is being populated — some sections are still in draft. For the complete picture, contact {{named-human-contact}}.

NEVER fabricate content to fill gaps.

## Sync Timestamp

Always include the sync timestamp in the displayed output:

> *Synced from Stratafy on {{timestamp}}. Refreshed weekly.*

## Provenance

- `_source_plugin`: `"clickatell"`
- `_source_command`: `"foundation"`
- `_change_reasoning`: e.g., `"Foundation cache miss after 7-day expiry"`

## Local Files

Reads / writes (all paths relative to the resolved project root):

- `<project-root>/.clickatell/foundation.md` — cached foundation document, scoped to the user's project folder so the Read/Write tools can access it directly in both Claude Code CLI and Claude Desktop / Cowork.

### Resolving the project root

The plugin runs in two environments. The cache location must be resolvable in both.

| Environment | What "project root" is |
|---|---|
| Claude Code CLI | The current working directory (`pwd`) — the directory the user invoked `claude` from |
| Claude Desktop / Cowork | The user's selected project folder. Its absolute path is exposed in the system prompt and accessible via the Read/Write/Bash tools. `pwd` in this mode points at a session scratchpad, not the project. |

Resolve `<project-root>` in this order via Bash:

1. **`$CLAUDE_PROJECT_DIR` environment variable** if set (Claude Code CLI exposes it; Cowork may too)
2. Otherwise, the **selected project folder** as exposed in the Cowork system prompt — its absolute path
3. As a last resort, the **current working directory** (`pwd`)

Bash snippet for path resolution + write-prep:

```bash
PROJECT_ROOT="${CLAUDE_PROJECT_DIR:-$(pwd)}"
mkdir -p "$PROJECT_ROOT/.clickatell"
echo "$PROJECT_ROOT/.clickatell/foundation.md"   # absolute path to use with Write
```

The Write tool does NOT expand `~` or resolve relative paths. Always pass the **absolute path** returned by the Bash resolution above.

### Why the project folder (not `$HOME`)

In Claude Desktop / Cowork, the model only has Read/Write access to the user's selected project folder. Files in `$HOME` (`~/.stratafy/`, `~/.clickatell/`) are invisible — every cache write would silently land somewhere unreadable on the next run. Scoping local files to the project folder fixes this and aligns with how Cowork actually models user data.

A side effect worth noting: foundation cache is per-project. A user who opens two different projects in Cowork will see two separate caches. That's correct semantics — if they want shared state across projects, they can run the Stratafy MCP fetch live each time.
