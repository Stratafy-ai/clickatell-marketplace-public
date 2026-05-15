---
description: "Role-aware induction — quick brief on what Clickatell is focused on right now, where you sit in it, what that means for your week, plus three concrete ways Claude helps in YOUR role and an invitation to dig deeper"
---

# /clickatell:onboard-me

The role-aware induction that comes after `/clickatell:welcome`. Welcome establishes the trust pitch and what the plugin is. Onboard-me makes it personal: where Clickatell is right now, where YOU sit in it, what that means for your day-to-day, and three concrete ways Claude becomes useful in YOUR role.

Feels like a real 1:1 induction conversation — not an interview. Open with what's going on at Clickatell, locate you in it, translate the company-level picture into your specific reality, then invite the questions you actually have before closing.

Conversational, ~10 minutes. Re-runnable when your role or focus shifts.

## Process

### Step 0: Pin Clickatell + Load User Context (always first)

Call `get_user_context` with:

- `workspace_id`: `"f06499c2-a2a8-4e7d-ad02-c66d6fd46873"` (Clickatell — pins the session as a side effect)
- `command_name`: `"onboard-me"`
- `plugin_name`: `"clickatell"`
- `_llm_model`, `_intent: "user_request"`, `_reason`, `_source_plugin: "clickatell"`, `_source_command: "onboard-me"`

Capture from the response: their `personal` context (values, forward_anchor, current_chapter), their `role` context if any (mandate, job_title, department), their `lens`. If `_meta.needs_onboarding: true` or `role` is null, you'll be filling more of these in this session.

### Step 0b: Parallel Brief Pull

Immediately after `get_user_context` returns, fire in parallel:

- `get_workspace_snapshot(sections: ["foundation", "strategies", "key_priorities"])` — the brief in one call
- Fall back to `list_key_priorities(filter: status=active)` if `key_priorities` isn't an accepted section name

What you need from the response:

- **Foundation:** mission tagline, 5 named values with one-line descriptions
- **Strategies:** the 2 most relevant to the user's role by matching `department` and `mandate` keywords
- **Key Priorities:** top 2–3 active KPs with one-line description each

Step 2 below depends on this data. Don't open the brief without it.

### Step 1: First-run vs re-run — the register decision

Resolve the project root and check for the onboarding flag:

```bash
PROJECT_ROOT="${CLAUDE_PROJECT_DIR:-$(pwd)}"
FLAG="$PROJECT_ROOT/.clickatell/onboarded.json"
test -f "$FLAG" && echo "PRESENT" || echo "ABSENT"
```

The flag sets the **register**, not just whether to run the full flow. Two different human moments:

- **ABSENT → first run.** Possibly the person's first-ever contact with Stratafy/Claude here. Open **warm and introductory** (Step 2A).
- **PRESENT → re-run.** They've done this; role/focus shifted. Open **tight** (Step 2B). Something changed → full flow. Nothing changed → skip to Step 8 wrap.

Never use first-run warmth on a re-run, or re-run brevity on a first run. And NEVER use weekly-rhythm phrasing ("your week", "this week", "set the week") — that's `/clickatell:lets-go`'s job. onboard-me is a one-time (re-runnable) role induction, not a Monday ritual.

### Step 2A: First run — warm welcome, then the brief

Lead with a genuine welcome and an orientation to what's about to happen. Then the brief (data from Step 0b), unhurried, as orientation not status report.

> "Welcome to Clickatell — and to working with Claude here.
>
> I'm going to take about 10 minutes to make sure Claude genuinely understands this company and your role in it — so from day one it's useful for your actual work, not generic. No forms, just a conversation, and you can stop me with a question any time.
>
> Here's the lay of the land."

Then, if role context exists (`role` is populated):

> "Clickatell exists to `{{mission tagline}}`. Three things are doing most of the work company-wide right now:
>
> - **`{{KP1.title}}`** — `{{KP1.one_line}}`
> - **`{{KP2.title}}`** — `{{KP2.one_line}}`
> - **`{{KP3.title}}`** — `{{KP3.one_line}}`
>
> You're coming in as **`{{role.job_title}}`** in **`{{role.department}}`** — that puts you most directly in the path of:
>
> - **`{{Strategy A.title}}`** — `{{Strategy A.tagline}}`
> - **`{{Strategy B.title}}`** — `{{Strategy B.tagline}}`
>
> The mandate we have on file for you: \"`{{role.mandate, first ~200 chars}}`\"
>
> Does that land? Anything you'd put differently?"

Pause for the user. This is the trust-build moment. A new person correcting their own mandate is a *good* sign — capture it and weave through the rest.

If `role` is null:

> "Clickatell exists to `{{mission tagline}}`. Three things are doing most of the work right now: `{{KP1}}`, `{{KP2}}`, `{{KP3}}`. I don't have your role on file yet — so once I understand your day-to-day, I'll tell you which two strategies and which two of the five values are most alive for your work. Let's start there."

### Step 2B: Re-run — tight re-anchor (no welcome)

They've onboarded before; skip the welcome.

> "Welcome back. Quick re-anchor — here's what's moved company-wide since last time:
>
> - **`{{KP1.title}}`** — `{{KP1.one_line}}`
> - **`{{KP2.title}}`** — `{{KP2.one_line}}`
> - **`{{KP3.title}}`** — `{{KP3.one_line}}`
>
> Stratafy still has you as **`{{role.job_title}}`** in **`{{role.department}}`**. Anything changed in your role or focus?"

- Changed → continue into the full flow (Step 3 onward), overwrite the file at the end
- Nothing changed → jump to Step 8 wrap; no rewrite

### Step 3: What this means for you — translation

Translate the brief into the user's specific reality. Two or three sentences. Role + relevant KPs + values, woven together:

> "For you — sitting in `{{role.job_title}}` with `{{role.mandate fragment}}` — two of those Key Priorities are where I'd expect most of your time to go: **`{{KP_X}}`** and **`{{KP_Y}}`**.
>
> The five Clickatell values sit underneath:
>
> - **CHAMPIONS** — Do the right thing
> - **CURIOUS** — Ask why
> - **COLLABORATIVE** — Succeed as a team
> - **COURAGEOUS** — True grit
> - **CREATIVE** — Find a way
>
> Most days, two of them do disproportionate work for any one person. In ten minutes I'll land which two drive your decisions, what your day-to-day actually looks like, and three concrete ways Claude becomes useful in YOUR role."

This is the connective tissue: company-priorities × your-role × values = your-specific-reality.

### Step 4: Day-to-day check (compressed — one question)

The brief did the heavy lifting. ONE grounded question:

> *"In a typical week, what are the two or three things that take most of your time?"*

Capture their actual words. Don't paraphrase — their vocabulary is what makes Claude useful later.

If their answer drifts away from the role context you opened with, flag it conversationally: *"Interesting — that's a bit different from the mandate on file. Should we update it?"* Don't auto-update; just surface.

### Step 5: Active values

Show the 5 values briefly (don't re-explain — Step 3 covered them).

> "Of the five values — CHAMPIONS, CURIOUS, COLLABORATIVE, COURAGEOUS, CREATIVE — which two are most active in how you make decisions?"

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

### Step 7: Questions or more info

Before closing, invite the user to dig deeper. This is the on-ramp from "system describing itself" to "user driving":

> "Before I close — anything you want to dig into?
>
> I can show you more on:
>
> - Any of the Key Priorities or strategies above
> - How decisions and risks are tracked at Clickatell
> - The foundation in more detail (`/clickatell:foundation`)
> - What's connected and what stays private (`/stratafy:status`)
> - Anything else"

If they ask a substantive question (e.g. *"tell me more about KP2"*, *"what's actually happening on the AI-Augmented Workforce strategy?"*, *"what's tracked about me?"*) — answer using workspace data. Pull the specific entity (`get_key_priority`, `get_strategy`, etc.) and walk them through it. Don't rush them back to the wrap.

If they say *"no, all good"* — proceed to Step 8.

This step is the difference between form-filling and conversation. Pause here. Don't rush past.

### Step 8: Wrap and write state

Close with:

> "From now on when you ask Claude something, it knows you're `{{role}}` in `{{department}}` working on `{{Strategy A}}` with `{{value1}}` and `{{value2}}` as your active values. Try one of the three patterns above, or just ask Claude something about your work — see what changes when it has context.
>
> Two commands turn this into a habit: `/clickatell:lets-go` on Monday morning to set the week, and `/clickatell:call-it-a-week` on Friday to close it. Try them this week.
>
> `/clickatell:onboard-me` re-runnable any time your role shifts. `/clickatell:help` for support."

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
  "primary_strategy_id": "{{strategy from Step 2 brief (most-active for role) or user override}}",
  "primary_strategy_name": "{{strategy name}}",
  "active_values": ["{{value1}}", "{{value2}}"],
  "weekly_focus": "{{user's answer from Step 4, first 200 chars}}",
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

1. ALWAYS branch the open by first-run vs re-run state (Step 2A warm welcome vs Step 2B tight re-anchor). Never the same register for both — warmth on a re-run patronises; brevity on a first run is cold on someone's first day.
2. NEVER use weekly-rhythm phrasing ("your week", "this week", "set the week") — that's `/clickatell:lets-go`. This is a one-time role induction, not a Monday ritual.
3. ALWAYS pull foundation + strategies + key_priorities in parallel (Step 0b) right after `get_user_context` — the brief depends on that data
4. NEVER paraphrase the user's answers — use their actual words when reflecting back role, focus, values
5. NEVER generic-ize the concrete patterns in Step 6 — anchor each one to a specific strategy, KP, or value
6. NEVER skip Step 7 (Questions/more info) — it's the difference between form-filling and conversation. Pause there. Answer substantively if asked.
7. NEVER write to `~/.clickatell/` — always the project-rooted path
8. ALWAYS pin Clickatell workspace via `get_user_context(workspace_id)` before anything else
9. The point is making Claude useful for THIS user's work, not collecting data. If the user wants to skip a question, skip it.
10. Re-runs are encouraged — roles shift, focus changes, this should feel low-friction to redo
