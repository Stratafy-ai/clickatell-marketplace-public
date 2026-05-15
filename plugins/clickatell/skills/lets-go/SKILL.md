---
name: "lets-go"
description: "Monday-morning rhythm command for a Clickatell employee. Reads role+strategy+values from onboarded.json, pulls the week's relevant objectives/decisions/insights/signals/risks from the workspace, helps the user commit to 1-3 things anchored to their active values. Use when the user runs /clickatell:lets-go or when a Monday morning starts and they have welcomed.json + onboarded.json but no week-{ISO}.json for the current week."
---

# Let's Go — Monday Morning Rhythm

The Monday command. Sets the week. Pulls the most relevant from the workspace, helps the user commit to 1–3 things, anchored to their active values.

Paired with `/clickatell:call-it-a-week` (Friday). Together they form the weekly rhythm — the load-bearing structural habit that turns AI-installed into AI-integrated.

## Voice

Pragmatic, warm, direct. Not corporate huddle. Not therapy. A 5-minute conversation that orients the week, not a checklist exercise.

## Workspace Pin + User Context (always first)

```
get_user_context(
  workspace_id: "f06499c2-a2a8-4e7d-ad02-c66d6fd46873",
  command_name: "lets-go",
  plugin_name: "clickatell",
  _llm_model: "<your model>",
  _intent: "user_request",
  _reason: "Monday-morning rhythm — pinning Clickatell workspace and loading user context",
  _source_plugin: "clickatell",
  _source_command: "lets-go"
)
```

Capture `personal`, `role`, `lens` from the response.

## Read onboarded state

Read `<project-root>/.clickatell/onboarded.json` to get role + primary strategy + active values:

```bash
PROJECT_ROOT="${CLAUDE_PROJECT_DIR:-$(pwd)}"
ONBOARDED="$PROJECT_ROOT/.clickatell/onboarded.json"
[ -f "$ONBOARDED" ] && cat "$ONBOARDED"
```

If missing, offer onboarding but proceed without it if user declines.

## Pull this week's relevant context

Parallel MCP calls (slim filters; aim for 5-10 entities total):

- `list_key_priorities` — active status — the company-wide cross-strategy lenses focused on right now
- `list_objectives` — active status, target_date within next 4 weeks, scoped to primary strategy if known
- `list_risks` — high impact + open status, scoped to primary strategy if known

Three pulls. Plus `get_user_context` for the identity line.

**Why deliberately spare:** decisions, insights, and signals surface naturally during the week when relevant. The Monday briefing is for orientation, not completeness. Less noise, more useful. If the user wants the full intelligence-layer dump, they can ask Claude directly mid-week.

**Why Key Priorities are included even for users anchored to a primary strategy:** KPs are cross-strategy lenses. A CHRO whose primary strategy is People & Culture still needs the week's company-wide focus surfaced. The KP section deliberately ignores the primary-strategy filter — it shows what the whole company is focused on this quarter.

If any pull returns empty for the user's scope, say so honestly. Don't pad with workspace-wide data unless the user asks.

## Conversation arc

### 1. Briefing

Render a structured Monday briefing — one screen, no scroll. Four sections:

- User's context line (role, primary strategy, active values)
- Company key priorities right now (active KPs — typically 2-4)
- Objectives near target dates (3-5 max)
- Open high-impact risks (2-3 max)

### 2. Commitments

Ask: *"What are 1–3 concrete things you're aiming to land this week?"*

Capture each in the user's own words. For each, ask: *"Which value or strategy is this anchored in?"* Capture.

If they have 1, that's fine. If they have 5, push back: *"Which two would you cut?"*

### 3. Close

> *"Three things this week, anchored to {{active values}}. I'll see you Friday — run `/clickatell:call-it-a-week`. Have a good week."*

## Write the commitments file

Compute the ISO week and write:

```bash
PROJECT_ROOT="${CLAUDE_PROJECT_DIR:-$(pwd)}"
ISO_WEEK=$(date +%G-W%V)
mkdir -p "$PROJECT_ROOT/.clickatell"
```

Absolute path: `$PROJECT_ROOT/.clickatell/week-${ISO_WEEK}.json`

JSON shape:

```json
{
  "version": "1.0",
  "iso_week": "{{ISO_WEEK}}",
  "set_at": "{{ISO8601 timestamp}}",
  "role": "{{role from onboarded.json}}",
  "primary_strategy_id": "{{strategy id}}",
  "primary_strategy_name": "{{strategy name}}",
  "active_values": ["{{value1}}", "{{value2}}"],
  "commitments": [
    {
      "commitment": "{{user's actual words}}",
      "anchor": "{{value or strategy name they chose}}"
    }
  ]
}
```

## Local Files

- `<project-root>/.clickatell/week-{iso-week}.json` — one per ISO week, set Monday, read Friday by call-it-a-week

The file is local-only. Does NOT sync to Stratafy. Commitments are between Claude and the user.

## Provenance

- `_source_plugin`: `"clickatell"`
- `_source_command`: `"lets-go"`
- `_change_reasoning`: e.g., `"Monday-morning rhythm for week 2026-W20"`

## Rules

1. NEVER fabricate workspace data — empty pulls render as empty
2. NEVER write to `~/.clickatell/` — always absolute project-rooted path
3. NEVER pressure the user to commit to 3 things — 1 is fine
4. Briefing is one screen — slim pulls, no data dumps
5. Commitments captured in user's own words, not paraphrased
6. If onboarded.json missing, offer onboarding; let user proceed without it
