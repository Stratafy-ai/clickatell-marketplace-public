---
name: "onboard-me"
description: "Role-aware onboarding for a Clickatell employee — captures their day-to-day work, the strategy their work ladders into, and the two values most active for them. Produces three concrete role-specific patterns for using Claude. Re-runnable when role shifts. Use when the user runs /clickatell:onboard-me or when <project-root>/.clickatell/onboarded.json doesn't exist and the user has already completed welcome."
---

# Onboard Me — Clickatell role-specific deepening

The role-aware onboarding that follows `/clickatell:welcome`. Welcome establishes trust and what the plugin is. Onboard-me makes it personal: who you are, what you do at Clickatell, the strategy your work touches, and three concrete ways Claude is useful in YOUR role.

~10 minutes, conversational. Re-runnable when role or focus shifts.

## Voice

Speak in CLICKATELL'S voice — pragmatic, warm, direct. Not corporate HR. Not generic AI. The whole point of this command is making Claude stop being generic and start being useful for the user's specific work.

## Workspace Pin + User Context (always first)

Before anything else, call `get_user_context` with `workspace_id` set. This pins Clickatell, logs the session, and returns user calibration data in one round-trip:

```
get_user_context(
  workspace_id: "f06499c2-a2a8-4e7d-ad02-c66d6fd46873",
  command_name: "onboard-me",
  plugin_name: "clickatell",
  _llm_model: "<your model>",
  _intent: "user_request",
  _reason: "Loading user context and pinning Clickatell workspace for role-aware onboarding",
  _source_plugin: "clickatell",
  _source_command: "onboard-me"
)
```

Capture from the response: `personal` (values, forward_anchor, current_chapter), `role` (mandate, job_title, primary_lens), `_meta.needs_onboarding`. This determines whether the session is filling new context or extending what exists.

## Trigger

Use when:

- The user runs `/clickatell:onboard-me`
- `<project-root>/.clickatell/onboarded.json` is missing AND the user has already completed welcome (welcomed.json present)
- The user signals their role has changed and asks to "redo onboarding" or similar

## Re-Run Behaviour

Resolve the project root via Bash, then check the flag:

```bash
PROJECT_ROOT="${CLAUDE_PROJECT_DIR:-$(pwd)}"
FLAG="$PROJECT_ROOT/.clickatell/onboarded.json"
```

- File missing → full version
- File present → ask: "Want to refresh — anything changed in your role or focus?"
  - Yes → full version, overwriting the file
  - No → skip to wrap (Step 7), no rewrite needed

## Conversation Arc

### 1. Open with what we already know

Don't open with a blank slate. If `get_user_context` returned a job title and mandate, reflect them back:

> *"Stratafy has you down as {{job_title}} with {{mandate}} as your mandate. I want to spend the next 10 minutes making sure Claude actually understands what you do day-to-day — so it stops being generic and becomes useful for YOUR work. Sound good?"*

If we don't know their role yet (`role` is null in user context), lead with:

> *"Let me ask a few quick things so Claude stops being generic and starts being useful for your specific work."*

### 2. Day-to-day check

Ask 1-2 grounded questions and capture the user's actual words. Don't paraphrase — their vocabulary is what makes Claude useful later.

- *"In a typical week, what are the two or three things that take most of your time?"*
- *"What's the work that, when it goes well, you feel like the day went well?"*

### 3. Strategy connection

Pull the active strategies via `get_workspace_snapshot(sections: ["strategies"])` if not already in context. Based on their role/answers, surface 2-3 strategies most likely to touch their work, with one-line taglines.

> *"Three of Clickatell's active strategies that most directly touch your work:*
> *- {{Strategy A}} — {{tagline}}*
> *- {{Strategy B}} — {{tagline}}*
> *- {{Strategy C}} — {{tagline}}*
>
> *Which one feels most alive in your work right now?"*

Capture which they pick. This becomes their primary strategy lens.

### 4. Values activation

Use `get_workspace_snapshot(sections: ["foundation"])` if not cached. Show all 5 values briefly:

> *"Clickatell has five values. Most days, two of them do the actual work in how you make decisions. Which two are most active for you?*
>
> *- CHAMPIONS — Do the right thing*
> *- CURIOUS — Ask why*
> *- COLLABORATIVE — Succeed as a team*
> *- COURAGEOUS — True grit*
> *- CREATIVE — Find a way"*

Capture their two. These become the values Claude leans on when drafting against the foundation.

### 5. Three concrete ways Claude helps you

Propose THREE concrete patterns based on (role + active strategy + two active values). DON'T generic-ize. Each pattern names a specific strategy or value.

Templates by role family — adapt to the specific user:

**Sales / Customer-facing:** customer-email-against-value / prospect-message-strategy-check / call-prep-with-context
**Engineering / Product:** technical-decision-against-strategy / one-pager-grounded-in-value / strategy-tree-impact-walk-through
**Operations / Internal:** short-process-doc-in-voice / rollout-risk-review / meeting-summary-with-decision-routing
**Leadership / Functional heads:** initiative-measurability-review / team-update-grounded-in-strategy / people-picture-scan

For each one, give a concrete sentence the user could literally paste back into Claude tomorrow. Examples:

> *"Try saying: 'Draft a customer email about the WhatsApp API rate limit change that respects CHAMPIONS — keep it under 150 words.'"*

### 6. Wrap and write state

Close with:

> *"From now on when you ask Claude something, it knows you're {{role}} working on {{Strategy A}} with {{value1}} and {{value2}} as your active values. Try one of the three patterns above, or just ask Claude about your work — see what changes when it has context. Run `/clickatell:onboard-me` again any time your role shifts."*

Then write the onboarded-state file with project-root path discipline:

```bash
PROJECT_ROOT="${CLAUDE_PROJECT_DIR:-$(pwd)}"
mkdir -p "$PROJECT_ROOT/.clickatell"
```

Then Write to the absolute path `$PROJECT_ROOT/.clickatell/onboarded.json`. JSON shape:

```json
{
  "version": "1.0",
  "onboarded_at": "{{ISO8601 timestamp}}",
  "role": "{{user-provided or get_user_context value}}",
  "primary_strategy_id": "{{strategy id from Step 3}}",
  "primary_strategy_name": "{{strategy name from Step 3}}",
  "active_values": ["{{value1}}", "{{value2}}"],
  "weekly_focus": "{{user's answer from Step 2, first 200 chars}}",
  "onboarding_script_version": "1.0"
}
```

NEVER pass `~/.clickatell/onboarded.json` to Write directly — the Write tool doesn't expand `~` or resolve relative paths.

## Local Files

All inside the user's project folder so they're visible to file tools in Claude Desktop / Cowork.

- `<project-root>/.clickatell/onboarded.json` — role + strategy + values + focus, written at end of full run

The onboarded state is local-only. It does NOT sync back to Stratafy automatically. If the user wants their Stratafy `role_context` (mandate, lens, etc.) updated based on this conversation, they update it via the Stratafy UI separately.

## What this skill does NOT do (v1)

- Does NOT auto-update `role_context` or `personal_context` in Stratafy
- Does NOT change any workspace entities
- Does NOT report the captured role/values to the workspace admin
- Does NOT require manager or HR review

The captured state is between Claude and the user, in the project folder.

## Trust framing

This skill captures personal information (role, weekly focus, active values). Be explicit about that during the wrap:

> *"What we just captured — your role, what takes your time, the values you lean on — lives only in this project folder. It doesn't go to Stratafy, your manager, or IT. Run `/stratafy:status` if you want to see exactly what the plugin reports back."*

The local-only design is intentional. It's the same trust posture as the welcome flow: aggregate signals (command-run counts, install events) go to the workspace admin; per-employee context never does.

## Provenance

- `_source_plugin`: `"clickatell"`
- `_source_command`: `"onboard-me"`
- `_change_reasoning`: e.g., `"Role-aware onboarding after welcome completion"`

## Rules

1. NEVER paraphrase the user's answers — use their actual words when reflecting back role, focus, values
2. NEVER generic-ize the patterns in Step 5 — every concrete pattern must reference a specific strategy or value
3. NEVER write to `~/.clickatell/` — always the absolute project-rooted path
4. ALWAYS pin Clickatell workspace first via `get_user_context(workspace_id)`
5. The point is making Claude useful for THIS user — not collecting data. Skip questions freely if the user signals they want to.
6. Re-runs are encouraged. Roles shift; this should feel low-friction.
