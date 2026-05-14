---
description: "End-of-week rhythm — close the week against Monday's commitments. What got done, what didn't, what surfaced, what worked and didn't work with Claude. Generates a weekly wrap you can paste into your team channel."
---

# /clickatell:call-it-a-week

The Friday command. Closes the week. The natural complement to `/clickatell:lets-go`.

Walks through what got done against Monday's commitments, captures what got stuck (with the *why*), surfaces insights/risks/decisions that emerged during the week, and gets two pieces of plugin feedback (one thing that worked, one thing that didn't). Generates a 1-paragraph wrap the user can share with their team.

Optional: at the end, offer to log material insights/risks/assumptions to Stratafy via the MCP (consent-based, not automatic).

## Process

### Step 0: Pin Clickatell + Load User Context (always first)

Call `get_user_context` with:

- `workspace_id`: `"f06499c2-a2a8-4e7d-ad02-c66d6fd46873"`
- `command_name`: `"call-it-a-week"`
- `plugin_name`: `"clickatell"`
- `_llm_model`, `_intent: "user_request"`, `_reason`, `_source_plugin: "clickatell"`, `_source_command: "call-it-a-week"`

### Step 1: Read Monday's commitments

```bash
PROJECT_ROOT="${CLAUDE_PROJECT_DIR:-$(pwd)}"
ISO_WEEK=$(date +%G-W%V)
WEEK_FILE="$PROJECT_ROOT/.clickatell/week-${ISO_WEEK}.json"
[ -f "$WEEK_FILE" ] && cat "$WEEK_FILE"
```

- If found → use the commitments array to drive the conversation
- If missing → ask: *"I don't have your Monday commitments — did you skip /clickatell:lets-go this week? We can still do the wrap, I'll just ask you to recall what you set out to do."*

### Step 2: Walk through each commitment

For each commitment from Monday (or each one the user recalls):

1. Repeat their words back: *"On Monday you said: '{{commitment text}}', anchored to {{anchor}}. How'd it go?"*
2. **If got done** → brief acknowledgement. Capture as "done."
3. **If didn't get done** → ask *why* — capture the answer. This becomes a candidate for an Assumption (we thought X, but Y) or a Risk (this isn't moving and here's why).
4. **If partial** → capture status, ask what's still in flight.

Stay conversational. Don't let it become a checklist.

### Step 3: What surfaced

Ask:

> *"Anything come up this week that should be logged — a decision you've been making, an assumption you've been working from, a risk you've been seeing, an insight that's worth capturing?"*

If yes, capture in their own words. Tag whether it's an `insight`, `assumption`, `risk`, or `decision`.

### Step 4: Plugin feedback (two questions)

> *"Two quick questions about Claude itself this week."*
>
> *"One thing Claude actually helped with?"*
>
> *"One thing you wished Claude could do but couldn't?"*

Append both with consent to `<project-root>/.clickatell/plugin-feedback.log`:

```bash
PROJECT_ROOT="${CLAUDE_PROJECT_DIR:-$(pwd)}"
echo "$(date -u +%Y-%m-%dT%H:%M:%SZ) | helped: {{response 1}}" >> "$PROJECT_ROOT/.clickatell/plugin-feedback.log"
echo "$(date -u +%Y-%m-%dT%H:%M:%SZ) | gap: {{response 2}}" >> "$PROJECT_ROOT/.clickatell/plugin-feedback.log"
```

(Or write via the Write tool with absolute path — append-only behaviour requires the Read-then-Write or Bash append pattern.)

### Step 5: Offer to log material items to Stratafy

If anything from Step 2 (why something didn't get done) or Step 3 (what surfaced) feels material, offer to log it to the workspace:

> *"The thing about {{X}} — should I log that to Stratafy as a {{insight/risk/assumption/decision}}? Takes 5 seconds, your team will see it. Or we keep it local."*

If yes → call the appropriate MCP tool (`create_insight`, `create_risk`, `create_assumption`, or `create_decision`) with provenance:

- `_source_plugin: "clickatell"`
- `_source_command: "call-it-a-week"`
- `_change_reasoning: "Captured during weekly wrap with {{user}}"`

If no → keep local. No pressure.

### Step 6: Generate the weekly wrap

Render a 1-paragraph wrap the user could paste into their team channel:

```markdown
**Week {{ISO_WEEK}} — {{user}}**

This week I focused on {{commitment summaries with what landed}}.
{{One sentence on what didn't and why, if relevant}}.
{{One sentence on what surfaced or what I learned}}.
Anchored to {{active values}}.
```

Keep it short, plain, factual. Not corporate.

### Step 7: Write the wrap file + close

Resolve and write:

```bash
PROJECT_ROOT="${CLAUDE_PROJECT_DIR:-$(pwd)}"
ISO_WEEK=$(date +%G-W%V)
mkdir -p "$PROJECT_ROOT/.clickatell"
echo "$PROJECT_ROOT/.clickatell/wrap-${ISO_WEEK}.json"
```

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
    {
      "commitment": "{{from Monday}}",
      "anchor": "{{from Monday}}",
      "status": "done | partial | stuck",
      "why_if_stuck": "{{user's words}}"
    }
  ],
  "surfaced": [
    {"type": "insight | assumption | risk | decision", "content": "{{user's words}}", "logged_to_stratafy": true | false}
  ],
  "plugin_feedback": {
    "what_helped": "{{user's words}}",
    "what_was_missing": "{{user's words}}"
  },
  "weekly_wrap_text": "{{the 1-paragraph summary}}"
}
```

Close with:

> *"That's the week, {{user}}. Wrap saved. {{If anything was logged to Stratafy: 'Logged {{X}} to the workspace — your team will see it.'}} Have a good weekend."*

## Voice

Reflective, not exhaustive. The conversation should feel like a check-in with a trusted colleague, not a status report.

## Provenance Context

- `_source_plugin`: `"clickatell"`
- `_source_command`: `"call-it-a-week"`
- `_change_reasoning`: e.g., `"Weekly wrap for {{user}} — week {{ISO_WEEK}}"`

## Rules

1. NEVER write to `~/.clickatell/` — always absolute project-rooted path
2. NEVER guilt the user if commitments slipped — capture the *why* instead, it's more useful
3. NEVER auto-log to Stratafy without explicit consent — every workspace mutation needs the user to say yes
4. Capture the user's own words. Don't summarise their summary.
5. Plugin feedback questions are non-skippable — they're what makes the next plugin version better
6. The weekly wrap text should fit in a team-channel post — short, plain, honest
