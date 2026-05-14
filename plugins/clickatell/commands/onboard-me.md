---
description: "Role-aware onboarding — connect your day-to-day work at Clickatell to the strategy, the values, and concrete ways Claude can help in YOUR role"
---

# /clickatell:onboard-me

The role-aware deepening that comes after `/clickatell:welcome`. Welcome establishes the trust pitch and what the plugin is. Onboard-me makes it personal: who you are, what you do at Clickatell, which strategies your work ladders into, and three concrete ways Claude becomes useful in YOUR role.

Conversational, ~10 minutes. Re-runnable when your role or focus shifts.

## Process

### Step 0: Pin Clickatell + Load User Context (always first)

Call `get_user_context` with:

- `workspace_id`: `"f06499c2-a2a8-4e7d-ad02-c66d6fd46873"` (Clickatell — pins the session as a side effect)
- `command_name`: `"onboard-me"`
- `plugin_name`: `"clickatell"`
- `_llm_model`, `_intent: "user_request"`, `_reason`, `_source_plugin: "clickatell"`, `_source_command: "onboard-me"`

Capture from the response: their `personal` context (values, forward_anchor, current_chapter), their `role` context if any (mandate, job_title), their `lens`. If `_meta.needs_onboarding: true` or `role` is null, you'll be filling more of these in this session.

### Step 1: Check First-Run State

Resolve the project root and check for the onboarding flag:

```bash
PROJECT_ROOT="${CLAUDE_PROJECT_DIR:-$(pwd)}"
FLAG="$PROJECT_ROOT/.clickatell/onboarded.json"
test -f "$FLAG" && echo "PRESENT" || echo "ABSENT"
```

- If absent → run the full version (Steps 2–7)
- If present → ask: "You've done onboarding before. Want to refresh — anything changed in your role or focus?" If yes → full version. If no → skip to Step 7 wrap.

### Step 2: Open with what we already know

Open with what `get_user_context` returned, not a blank slate. If we know their job title and mandate from Stratafy, reflect it back:

> "Stratafy has you down as `{{job_title}}` at Clickatell, with `{{mandate}}` as your mandate. I want to spend the next 10 minutes making sure Claude actually understands what you do day-to-day — so it stops being a generic AI and becomes useful for YOUR work. Sound good?"

If we don't know their role yet, lead with: *"Let me ask a few quick things so Claude stops being generic and starts being useful for your specific work."*

### Step 3: Day-to-day check

Ask one or two grounded questions, depending on whether you already have role context:

- *"In a typical week at Clickatell, what are the two or three things that take the most of your time?"*
- *"What's the work that, when it goes well, you feel like the day went well?"*

Capture their actual words. Don't paraphrase — their vocabulary is what makes Claude useful later.

### Step 4: Strategy connection

Based on their role, pull the strategies their work most likely ladders into. Use `get_workspace_snapshot` with `sections: ["strategies"]` if you don't already have them from Step 0.

Surface 2–3 most relevant strategies with one-line descriptions:

> "Three of Clickatell's active strategies that most directly touch your work:
> - **{{Strategy A}}** — {{tagline}}
> - **{{Strategy B}}** — {{tagline}}
> - **{{Strategy C}}** — {{tagline}}
>
> Which one feels most alive in your work right now?"

Capture their answer. This becomes the strategy lens they reference in their day-to-day Claude usage.

### Step 5: Values activation

Show the 5 Clickatell values briefly. Use `get_workspace_snapshot` with `sections: ["foundation"]` if not cached.

> "Clickatell has five values. Most days, two of them are doing the actual work in how you make decisions. Which two feel most active for you?"
>
> - **CHAMPIONS** — Do the right thing
> - **CURIOUS** — Ask why
> - **COLLABORATIVE** — Succeed as a team
> - **COURAGEOUS** — True grit
> - **CREATIVE** — Find a way

Capture their two. These become the values Claude leans on when drafting against the foundation.

### Step 6: Three concrete ways Claude helps you

Based on (role + active strategy + two active values), propose three concrete patterns. Examples by role-type — adapt to the specific user:

**Sales / Customer-facing:**
1. *"Draft a customer email about [X] that respects [their chosen value, e.g. CHAMPIONS]."*
2. *"Review this prospect message and tell me whether it lands as `{{Strategy A}}` would want it to."*
3. *"Help me prep for a call with [account] — pull what we know about their context."*

**Engineering / Product:**
1. *"Review this technical decision against `{{Strategy B}}` and tell me what's missing."*
2. *"Draft a one-pager on [proposal] that connects it to our values — especially `{{their value}}`."*
3. *"Walk me through how this change touches the strategy tree."*

**Operations / Internal:**
1. *"Help me write a process doc that's actually short — keep it under 300 words, in our voice."*
2. *"Review this rollout plan for risk."*
3. *"Summarise this meeting and call out what should become a decision in Stratafy."*

**Leadership / Functional Heads:**
1. *"Read this initiative and tell me what's not measurable yet."*
2. *"Draft a quarterly update to my team that connects what we're doing to `{{Strategy A}}`."*
3. *"Walk me through the team's people picture — concentration risk, capacity, hiring gaps."*

For each pattern, name the specific value or strategy it's anchored in. Don't generic-ize.

### Step 7: Wrap and write state

Close with:

> "From now on when you ask Claude something, it knows you're `{{role}}` working on `{{Strategy A}}` with `{{value1}}` and `{{value2}}` as your active values. Try one of the three things above, or just ask Claude something about your work — see what changes when it has context.
>
> Run `/clickatell:onboard-me` again any time your role shifts. `/clickatell:welcome` for the introduction recap. `/clickatell:help` for support."

Then write the state file with project-root path discipline:

```bash
PROJECT_ROOT="${CLAUDE_PROJECT_DIR:-$(pwd)}"
mkdir -p "$PROJECT_ROOT/.clickatell"
echo "$PROJECT_ROOT/.clickatell/onboarded.json"   # absolute path for Write tool
```

JSON shape:

```json
{
  "version": "1.0",
  "onboarded_at": "{{ISO8601 timestamp}}",
  "role": "{{user-provided or get_user_context value}}",
  "primary_strategy_id": "{{strategy id from Step 4}}",
  "primary_strategy_name": "{{strategy name from Step 4}}",
  "active_values": ["{{value1}}", "{{value2}}"],
  "weekly_focus": "{{user's answer from Step 3, first 200 chars}}",
  "onboarding_script_version": "1.0"
}
```

NEVER pass `~/.clickatell/onboarded.json` to Write. Always use the absolute project-rooted path.

## Voice

Speak in CLICKATELL'S voice, not Stratafy's. Practical, warm, direct. No corporate HR language. The point is to make Claude useful for the user's specific work — not to put them through an HR process.

## What this command does NOT do (v1)

- **Does not auto-update role_context in Stratafy.** The captured role/values stay local. If the user wants their Stratafy role context updated, they edit it via the Stratafy UI.
- **Does not change any workspace state.** Read-only against Stratafy; write-only against local project files.
- **Does not require a manager or HR review.** The captured data is between Claude and the user.

## Provenance Context

On every MCP call, include:

- `_source_plugin`: `"clickatell"`
- `_source_command`: `"onboard-me"`
- `_change_reasoning`: brief description (e.g., "Loading user context for role-aware onboarding")

## Rules

1. NEVER paraphrase the user's answers — use their actual words when you reflect back roles, focus, values
2. NEVER generic-ize the concrete patterns in Step 6 — anchor each one to a specific strategy or value
3. NEVER write to `~/.clickatell/` — always the project-rooted path
4. ALWAYS pin Clickatell workspace via `get_user_context(workspace_id)` before anything else
5. The point is making Claude useful for THIS user's work, not collecting data. If the user wants to skip a question, skip it.
6. Re-runs are encouraged — roles shift, focus changes, this should feel low-friction to redo
