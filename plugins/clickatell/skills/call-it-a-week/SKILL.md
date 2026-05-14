---
name: "call-it-a-week"
description: "Friday-afternoon rhythm command for a Clickatell employee. Reads Monday's commitments from week-{iso-week}.json, walks through what got done / didn't / surfaced, captures plugin feedback, offers to log material items to Stratafy with consent, generates a 1-paragraph team-channel-ready weekly wrap. Use when the user runs /clickatell:call-it-a-week or on a Friday afternoon when week-{iso}.json exists but wrap-{iso}.json doesn't."
---

# Call It a Week — End-of-Week Rhythm

The Friday command. Closes the week against Monday's commitments. Captures what got done, what didn't (with the *why*), what surfaced, and what worked/didn't with Claude. Generates a team-shareable wrap.

Paired with `/clickatell:lets-go` (Monday). Together they form the weekly rhythm.

## Voice

Reflective, warm, honest. Like a Friday check-in with a trusted colleague — not a status report, not a performance review. The point is to honestly close the week.

## Workspace Pin + User Context (always first)

```
get_user_context(
  workspace_id: "f06499c2-a2a8-4e7d-ad02-c66d6fd46873",
  command_name: "call-it-a-week",
  plugin_name: "clickatell",
  _llm_model: "<your model>",
  _intent: "user_request",
  _reason: "End-of-week wrap — pinning Clickatell workspace and loading user context",
  _source_plugin: "clickatell",
  _source_command: "call-it-a-week"
)
```

## Read Monday's commitments

```bash
PROJECT_ROOT="${CLAUDE_PROJECT_DIR:-$(pwd)}"
ISO_WEEK=$(date +%G-W%V)
WEEK_FILE="$PROJECT_ROOT/.clickatell/week-${ISO_WEEK}.json"
[ -f "$WEEK_FILE" ] && cat "$WEEK_FILE"
```

- Found → use the commitments array to drive Step 1
- Missing → tell user honestly, offer to do the wrap from memory anyway

## Conversation arc

### 1. Commitment review

For each commitment from Monday (or each one the user recalls):

- Repeat their words back: *"On Monday you said: '{{commitment}}', anchored to {{anchor}}. How'd it go?"*
- Capture status: done / partial / stuck
- If stuck or partial → ask *why* in 1 sentence. The *why* is more valuable than the status.

### 2. What surfaced

> *"Anything come up this week that should be logged — a decision you've been making, an assumption you've been working from, a risk you've been seeing, an insight worth capturing?"*

Capture each as one of: `insight`, `assumption`, `risk`, `decision`. User's own words.

### 3. Plugin feedback (non-skippable)

Ask both:

- *"One thing Claude actually helped with this week?"*
- *"One thing you wished Claude could do but couldn't?"*

Append to `<project-root>/.clickatell/plugin-feedback.log` with consent. Format per line:

```
{{ISO8601 timestamp}} | helped: {{response}}
{{ISO8601 timestamp}} | gap: {{response}}
```

### 4. Optional: log material items to Stratafy

If anything from Step 1 (the why) or Step 2 (what surfaced) feels material, offer to log it:

> *"The thing about {{X}} — should I log that to Stratafy as a {{insight | risk | assumption | decision}}? Takes 5 seconds, your team will see it."*

If yes, call the right MCP tool with provenance (`_source_plugin: "clickatell"`, `_source_command: "call-it-a-week"`). Use:

- `create_insight` for patterns / learnings
- `create_assumption` for "we thought X, but Y" findings  
- `create_risk` for things that aren't moving + why
- `create_decision` for choices the user is making / has made

If no, keep local. No pressure.

### 5. Generate the weekly wrap

Render a 1-paragraph plain-language wrap the user could paste into their team channel:

```markdown
**Week {{ISO_WEEK}} — {{user}}**

This week I focused on {{commitments with status}}. {{One line on what didn't and why, if relevant}}. {{One line on what surfaced or what was learned}}. Anchored to {{active values}}.
```

Show it. Let the user edit if they want.

### 6. Write wrap state file

Compute ISO week, write JSON to absolute project-rooted path:

```bash
PROJECT_ROOT="${CLAUDE_PROJECT_DIR:-$(pwd)}"
ISO_WEEK=$(date +%G-W%V)
mkdir -p "$PROJECT_ROOT/.clickatell"
```

Path: `$PROJECT_ROOT/.clickatell/wrap-${ISO_WEEK}.json`

JSON shape:

```json
{
  "version": "1.0",
  "iso_week": "{{ISO_WEEK}}",
  "wrapped_at": "{{ISO8601 timestamp}}",
  "role": "{{from onboarded.json}}",
  "primary_strategy_name": "{{primary strategy}}",
  "active_values": ["{{value1}}", "{{value2}}"],
  "commitments_review": [
    {"commitment": "...", "anchor": "...", "status": "done | partial | stuck", "why_if_stuck": "..."}
  ],
  "surfaced": [
    {"type": "insight | assumption | risk | decision", "content": "...", "logged_to_stratafy": true|false}
  ],
  "plugin_feedback": {
    "what_helped": "...",
    "what_was_missing": "..."
  },
  "weekly_wrap_text": "..."
}
```

### 7. Close

> *"That's the week, {{user}}. Wrap saved. {{If anything logged: 'Logged {{N items}} to the workspace.'}} Have a good weekend."*

## Local Files

- `<project-root>/.clickatell/wrap-{iso-week}.json` — one per ISO week, written Friday
- `<project-root>/.clickatell/plugin-feedback.log` — append-only, all weeks; with consent

Both are local-only. The plugin-feedback log is the structured surface for "what would make this better" — reviewed monthly by the named human contact and CHRO per the Welcome Q&A document's process.

## Provenance

- `_source_plugin`: `"clickatell"`
- `_source_command`: `"call-it-a-week"`
- `_change_reasoning`: e.g., `"End-of-week wrap for {{user}}, week {{ISO_WEEK}}"`

## Rules

1. NEVER guilt the user if commitments slipped — capture the *why*, it's the useful signal
2. NEVER auto-log to Stratafy — every workspace mutation needs explicit consent for THIS specific item
3. NEVER write to `~/.clickatell/` — always absolute project-rooted path
4. Plugin feedback questions are non-skippable — they're how the next version gets better
5. Capture in the user's own words. No paraphrasing of their summary.
6. The weekly wrap text fits a team-channel post — short, plain, honest. Not corporate.
