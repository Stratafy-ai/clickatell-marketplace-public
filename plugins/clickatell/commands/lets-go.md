---
description: "Monday morning rhythm — pull the week's most relevant objectives, decisions, and signals from your strategies; capture 1–3 commitments anchored to your active values"
---

# /clickatell:lets-go

The Monday-morning command. Sets the week.

Pulls what's most relevant to your work from Stratafy — active objectives near their target dates, pending decisions in your strategies, recent insights and signals. Then helps you commit to 1–3 concrete things you're aiming to land this week, anchored to your active values from `/clickatell:onboard-me`.

Run on Monday. Re-runnable mid-week if priorities shift. Paired with `/clickatell:call-it-a-week` on Friday.

## Process

### Step 0: Pin Clickatell + Load User Context (always first)

Call `get_user_context` with:

- `workspace_id`: `"f06499c2-a2a8-4e7d-ad02-c66d6fd46873"` (Clickatell — pins the session as a side effect)
- `command_name`: `"lets-go"`
- `plugin_name`: `"clickatell"`
- `_llm_model`, `_intent: "user_request"`, `_reason`, `_source_plugin: "clickatell"`, `_source_command: "lets-go"`

### Step 1: Resolve project root + read onboarded state

The user's role, primary strategy, and active values were captured by `/clickatell:onboard-me` to a local file. Read them so the week-setting is role-aware:

```bash
PROJECT_ROOT="${CLAUDE_PROJECT_DIR:-$(pwd)}"
ONBOARDED="$PROJECT_ROOT/.clickatell/onboarded.json"
[ -f "$ONBOARDED" ] && cat "$ONBOARDED"
```

- If found → use `role`, `primary_strategy_name`, `active_values` from the JSON
- If missing → offer to run `/clickatell:onboard-me` first (5 sec ask); user can decline and continue with generic context

### Step 2: Pull the workspace's "this week" context

Call in parallel:

- `list_objectives` filtered to active status with target dates within the next 4 weeks, scoped to the user's primary strategy if known
- `list_risks` filtered to high impact / open status, scoped to the user's primary strategy if known

Aim for a slim pull — 5–10 entities total across both categories. The point is signal, not noise. The briefing is two sections (Objectives near target + High-impact risks) plus the user's context line — deliberately spare.

Why no decisions / insights / signals: those surface naturally during the week when relevant. The Monday briefing is for orientation, not completeness. Less noise, more useful.

### Step 3: Render the briefing

Present a structured Monday-morning briefing:

```markdown
# Let's go — {{ISO week}} ({{date}})

Good morning, {{first_name}}. Here's what the workspace has for you this week.

## Your context
- Role: {{role}}
- Primary strategy: {{primary_strategy_name}}
- Active values: {{value1}} · {{value2}}

## Objectives near their target dates
{{For each: title — target_date — current_value vs target_value}}

## High-impact open risks
{{For each: name — likelihood × impact — one-line description}}
```

If either section is empty, say so cleanly: *"No high-impact risks logged against your primary strategy."* Don't fabricate.

### Step 4: Commit to 1–3 things

After the briefing, ask:

> *"What are 1–3 concrete things you're aiming to land this week?"*

Stay in the conversation until the user gives you 1–3 commitments. For each one:

1. Capture in their actual words
2. Ask: *"Which value or strategy is this anchored in?"*
3. Capture which they pick

If they have trouble naming 3, one is fine. Don't pressure-pad.

### Step 5: Write commitments to local state

Resolve the ISO week and write the commitments file:

```bash
PROJECT_ROOT="${CLAUDE_PROJECT_DIR:-$(pwd)}"
ISO_WEEK=$(date +%G-W%V)   # e.g. 2026-W20
mkdir -p "$PROJECT_ROOT/.clickatell"
echo "$PROJECT_ROOT/.clickatell/week-${ISO_WEEK}.json"   # absolute path for Write
```

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

NEVER pass `~/.clickatell/...` to Write. Always the absolute project-rooted path.

### Step 6: Close

Close with:

> *"Got it. Three things this week, anchored to {{active values}}. I'll see you Friday — run `/clickatell:call-it-a-week` and we'll look at what landed, what didn't, and what surfaced. Have a good week."*

## Voice

Clickatell voice — pragmatic, warm, direct. Not corporate huddle. Not therapy. The point is a 5-minute conversation that orients the week.

## Provenance Context

- `_source_plugin`: `"clickatell"`
- `_source_command`: `"lets-go"`
- `_change_reasoning`: brief description (e.g., "Setting up Monday-morning briefing for {{user}}")

## Rules

1. NEVER fabricate workspace data — if a section returns empty, say so
2. NEVER write to `~/.clickatell/` — always the absolute project-rooted path
3. NEVER pressure the user to name 3 commitments if they only have 1
4. The briefing should fit on one screen — slim pulls, not data dumps
5. If `onboarded.json` is missing, offer onboarding but let them continue without it
6. Capture commitments in the user's own words, don't paraphrase
