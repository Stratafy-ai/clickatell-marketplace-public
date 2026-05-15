---
name: "onboard-me"
description: "Role-aware induction for a Clickatell employee — opens with a brief on what Clickatell is focused on right now (foundation + active Key Priorities + strategies relevant to their role), translates that against their mandate and active values, captures their day-to-day, and invites questions. Produces three concrete role-specific patterns for using Claude and invites the user to dig deeper before closing. Re-runnable when role shifts. Use when the user runs /clickatell:onboard-me or when <project-root>/.clickatell/onboarded.json doesn't exist and the user has already completed welcome."
---

# Onboard Me — Clickatell role-specific induction

The role-aware induction that follows `/clickatell:welcome`. Welcome establishes trust and what the plugin is. Onboard-me makes it personal: where Clickatell is right now, where YOU sit in it, what that means for your week, and three concrete ways Claude becomes useful in YOUR role.

Feels like a real 1:1 induction — not an interview. Open with what's going on at Clickatell, locate the user in it, translate the company-level picture into their day-to-day reality, then invite questions.

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

Capture from the response: `personal` (values, forward_anchor, current_chapter), `role` (mandate, job_title, department, primary_lens), `_meta.needs_onboarding`. This determines whether the session is filling new context or extending what exists.

## Data Pulls (parallel, immediately after workspace pin)

Right after `get_user_context` returns, fire these in parallel to pre-load the brief:

- `get_workspace_snapshot(sections: ["foundation", "strategies", "key_priorities"])` — the company brief in one call
- Fall back to `list_key_priorities(filter: status=active)` if `key_priorities` isn't an accepted section name in your snapshot tool

From these, extract:

- **Foundation:** mission tagline, vision, 5 named values with one-line descriptions
- **Strategies:** all active strategies; identify the 2 most relevant to the user's role by matching `department` and `mandate` keywords
- **Key Priorities:** the top 2–3 active KPs with one-line description each (these are the cross-strategy lenses doing most of the company-wide work right now)

The brief in Step 2 below depends on this data being present. Don't open the conversation without it.

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

### 1. Brief — Where Clickatell is + where you sit

This is the load-bearing change. Don't open with a question. Open with a 30-second brief that locates the user inside the company picture. The data pulls above made this possible — use them.

If role context exists (`role` is populated):

> *"Quick brief before we get into your week.*
>
> *Clickatell — {{mission tagline}}. Right now, three things are doing most of the work company-wide:*
>
> *- **{{KP1.title}}** — {{KP1.one_line}}*
> *- **{{KP2.title}}** — {{KP2.one_line}}*
> *- **{{KP3.title}}** — {{KP3.one_line}}*
>
> *You're sitting in **{{role.job_title}}** in **{{role.department}}** — that puts you most directly in the path of:*
>
> *- **{{Strategy A.title}}** — {{Strategy A.tagline}}*
> *- **{{Strategy B.title}}** — {{Strategy B.tagline}}*
>
> *Your mandate, as Stratafy has it: \"{{role.mandate truncated to ~200 chars}}\"*
>
> *Make sense? Anything off?"*

Pause here. This is the trust-build moment. If they say "actually I don't really do X anymore, I focus on Y" — capture it and weave through the rest of the conversation. Don't barrel into Step 2 if there's a correction in front of you.

If `role` is null:

> *"Quick brief before we get into your work. Clickatell is focused on three things right now:*
>
> *- **{{KP1.title}}** — {{KP1.one_line}}*
> *- **{{KP2.title}}** — {{KP2.one_line}}*
> *- **{{KP3.title}}** — {{KP3.one_line}}*
>
> *The work ladders into a handful of active strategies. Once I know what your day-to-day looks like, I can tell you which two of those strategies — and which two of the five values — are most alive for you. Sound good?"*

### 2. What this means for you — translation

Translate the brief into the user's specific reality. Role + relevant KPs + foundation values, woven into a 2–3 sentence narrative:

> *"For you — sitting in {{role.job_title}} with {{role.mandate fragment, e.g. \"the AI-Augmented Workforce rollout\"}} — two of those Key Priorities are where I'd expect most of your week to live: **{{KP_X}}** and **{{KP_Y}}**.*
>
> *The five Clickatell values sit underneath all of that:*
>
> *- **CHAMPIONS** — Do the right thing*
> *- **CURIOUS** — Ask why*
> *- **COLLABORATIVE** — Succeed as a team*
> *- **COURAGEOUS** — True grit*
> *- **CREATIVE** — Find a way*
>
> *Most days, two of them do disproportionate work for any one person. In ten minutes I'll land which two drive your decisions, what your week actually looks like, and three concrete ways Claude becomes useful in YOUR role."*

This is the translation step the user needs. It connects company-level priorities + their specific role + the values that anchor decisions. They should walk out of Step 2 knowing exactly where they fit.

### 3. Day-to-day check (compressed — one question)

The brief did the heavy lifting. This is calibration, not interview. ONE grounded question:

> *"In a typical week, what are the two or three things that take most of your time?"*

Capture their actual words. Don't paraphrase — their vocabulary is what makes Claude useful later.

If their answer drifts away from the role context you opened with, that's a signal — surface it conversationally: *"Interesting — that's a bit different from the mandate we have on file. Should we update it?"* Don't auto-update; just flag.

### 4. Active values

Show the 5 values briefly. Don't re-explain — the brief in Step 2 covered them.

> *"Of the five values — CHAMPIONS, CURIOUS, COLLABORATIVE, COURAGEOUS, CREATIVE — which two are most active in how you make decisions?"*

Capture their two. These become the values Claude leans on when drafting against the foundation.

### 5. Three concrete ways Claude helps you

Now you have everything: role + active strategies + relevant KPs + two active values + week-shape. Propose three concrete patterns. DON'T generic-ize. Each pattern names a specific strategy, KP, or value.

Templates by role family — adapt to the specific user:

**Sales / Customer-facing:** customer-email-against-value / prospect-message-strategy-check / call-prep-with-context
**Engineering / Product:** technical-decision-against-strategy / one-pager-grounded-in-value / strategy-tree-impact-walk-through
**Operations / Internal:** short-process-doc-in-voice / rollout-risk-review / meeting-summary-with-decision-routing
**Leadership / Functional heads:** initiative-measurability-review / team-update-grounded-in-strategy / people-picture-scan

For each one, give a concrete sentence the user could literally paste back into Claude tomorrow. Examples:

> *"Try saying: 'Draft a customer email about the WhatsApp API rate limit change that respects CHAMPIONS — keep it under 150 words.'"*

### 6. Questions or more info (NEW — before wrap)

Invite the user to dig further. This is the on-ramp from "system describing itself" to "user driving":

> *"Before I close — anything you want to dig into?*
>
> *I can show you more on:*
>
> *- Any of the Key Priorities or strategies above*
> *- How decisions and risks are tracked at Clickatell*
> *- The foundation in more detail (`/clickatell:foundation`)*
> *- What's connected and what stays private (`/stratafy:status`)*
> *- Anything else"*

If they ask a substantive question (e.g., *"Tell me more about KP2"*, *"What's the AI-Augmented Workforce strategy actually doing?"*, *"What's tracked about me?"*) — answer it using workspace data. Pull the specific entity (`get_key_priority`, `get_strategy`, etc.) and walk them through it. Don't rush them back to the wrap.

If they say *"no, all good"* — proceed to wrap.

This step matters: it's the difference between onboarding-as-form-filling and onboarding-as-conversation. The user should feel they could stay and ask anything; they're not on rails.

### 7. Wrap and write state

Close with:

> *"From now on when you ask Claude something, it knows you're {{role}} sitting in {{department}} with {{Strategy A}} as the most-active strategy and {{value1}} and {{value2}} as your active values. Try one of the three patterns above, or just ask Claude about your work — see what changes when it has context.*
>
> *Two ways to make this stick as a habit: run `/clickatell:lets-go` on Monday morning to set the week with the workspace context, then `/clickatell:call-it-a-week` on Friday to close it out. Those two together turn this from 'I installed Claude' into 'Claude is how I work.'"*

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

1. ALWAYS open with the brief (Step 1), not with a question. The brief is what makes this feel like induction, not interview.
2. ALWAYS pull foundation + strategies + key_priorities in parallel right after `get_user_context` — the brief depends on that data being present
3. NEVER paraphrase the user's answers — use their actual words when reflecting back role, focus, values
4. NEVER generic-ize the patterns in Step 5 — every concrete pattern must reference a specific strategy, KP, or value
5. NEVER skip Step 6 (Questions/more info) — it's the difference between form-filling and conversation. Pause there, answer substantively if asked
6. NEVER write to `~/.clickatell/` — always the absolute project-rooted path
7. ALWAYS pin Clickatell workspace first via `get_user_context(workspace_id)`
8. The point is making Claude useful for THIS user — not collecting data. Skip questions freely if the user signals they want to.
9. Re-runs are encouraged. Roles shift; this should feel low-friction.
