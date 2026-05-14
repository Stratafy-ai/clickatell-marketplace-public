---
name: "welcome"
description: "First-run induction for Claude at Clickatell. Walks the employee through what the plugin is, why Clickatell is rolling it out, what's local vs synced, honest boundaries, and answers their questions. Re-runnable. Use when the user runs /clickatell:welcome or when <project-root>/.clickatell/welcomed.json doesn't exist and the user is interacting with the plugin."
---

# Welcome — Clickatell Onboarding

The first-run induction. ~5 minutes. Re-runnable.

## Voice

Throughout this skill, speak in CLICKATELL'S voice, not Stratafy's. The welcome script is co-authored with Clickatell and signed off by Pieter or designated comms.

## Workspace Pin + User Context (single call, always first)

Before anything else, call `get_user_context` with `workspace_id` set. This pins Clickatell, logs the session, and returns user calibration data in one round-trip:

```
get_user_context(
  workspace_id: "f06499c2-a2a8-4e7d-ad02-c66d6fd46873",
  command_name: "welcome",
  plugin_name: "clickatell",
  _llm_model: "<your model>",
  _intent: "user_request",
  _reason: "Loading user context and pinning Clickatell workspace for welcome flow",
  _source_plugin: "clickatell",
  _source_command: "welcome"
)
```

Do NOT call `select_workspace` separately — passing `workspace_id` to `get_user_context` does both jobs. This pinning ensures any organic conversation after the welcome is anchored on Clickatell, regardless of which workspace the user had previously selected.

## Re-Run Behaviour

Cache file lives at `<project-root>/.clickatell/welcomed.json`. Resolve `<project-root>` via Bash (`PROJECT_ROOT="${CLAUDE_PROJECT_DIR:-$(pwd)}"`) — see *Local Files* below for full path-resolution rules.

- File missing → full version
- File present → ask: full or recap?
  - Full → all stages
  - Recap → stages 1, 5, 7 (welcome, what-it-isn't reminder, wrap)

## Conversation Arc

### 1. Open

Short warm welcome in Clickatell's voice. Acknowledge the new install, signal the next 5 min.

> [Welcome script v1 — Clickatell-signed-off content goes here once approved.]

### 2. The Why

2–3 sentences from Clickatell leadership about why this is happening. Must be Pieter-or-comms-approved.

### 3. The What

Three concrete asks employees can make of Claude now:

1. "Ask Claude about Clickatell's foundation — try `/clickatell:foundation`"
2. "Ask Claude to draft something that reflects our values"
3. "Ask Claude what it knows about Clickatell"

### 4. What It Isn't

Honest boundaries. Each line must be verifiable:

- Doesn't read your emails or messages unless you point it at them
- Doesn't report your activity to your manager, IT, or Clickatell
- Conversations stay on your machine
- Doesn't have all the answers — Stratafy is being populated, some context will be thin in the early weeks
- Not a replacement for talking to your team

### 5. Questions

Open the floor with: *"Any questions about how this works, what's connected, or what we'll do with it? I'll answer from what I know, or capture it for the team if it's outside my reference."*

**Then STOP and wait for the user's response.** Do not continue to Step 6 in the same turn. The welcome is interactive; this stage is genuinely conversational. You should hand back to the user and wait for them to either ask a question or signal they're done (e.g. "no questions," "all good," "let's wrap").

When the user replies:

- **If they ask a question** in the curated Q&A below → answer from the Q&A. Then ask if they have more.
- **If the question is outside the Q&A set** →
  1. Ask consent: *"I don't have an answer to that one. Can I capture your question to share with the team?"*
  2. If yes → append to `$PROJECT_ROOT/.clickatell/welcome-questions.log` (resolve `$PROJECT_ROOT` via Bash — see *Local Files* below)
  3. Always point to {{named-human-contact}} at {{contact-email}}
  4. Then ask if they have more questions.
- **If they signal done** (or say no questions) → proceed to Step 6.

The default path for an employee with no questions is: open the floor → user says "I'm good" / "no questions" → proceed to wrap. The interactive pause is non-negotiable; rendering the question prompt and immediately wrapping defeats the trust pitch.

### 6. Wrap

Closing line. Recommend three follow-ups in this order:

1. **`/clickatell:onboard-me`** — the natural next step. *"That conversation captures your role, the strategies your work ladders into, and two of our five values that are most active for you — so Claude actually knows what you do. ~10 min, re-runnable."*
2. **`/clickatell:help`** — reference card, available anytime
3. **`/clickatell:welcome`** — re-run this welcome

Then write the welcomed-state file with project-root path discipline:

```bash
PROJECT_ROOT="${CLAUDE_PROJECT_DIR:-$(pwd)}"
mkdir -p "$PROJECT_ROOT/.clickatell"
echo "$PROJECT_ROOT/.clickatell/welcomed.json"   # absolute path to pass to Write
```

Then pass the absolute path (e.g. `/Users/X/projects/foo/.clickatell/welcomed.json`) to Write. NEVER pass `~/.clickatell/welcomed.json` or a relative `.clickatell/welcomed.json` — the Write tool does not expand `~` or resolve relative paths.

Same path discipline applies to `welcome-questions.log` if you append captured questions in Step 5.

**Migration note:** if `~/.clickatell/welcomed.json` exists from an older plugin version, treat it as if the welcome hasn't happened yet (run the full version), then write the new project-scoped file. The old home-directory location is invisible to file tools in Claude Desktop / Cowork.

## Curated Q&A Reference (v1 — pending Clickatell sign-off)

> **Source of truth:** Welcome Q&A — Clickatell Cowork Plugin v1 (Stratafy workspace document `5edf6413-d05e-4ae2-a9f5-4cd9041a66ad`, linked under the *AI-Augmented Workforce* strategy → *Pilot Rollout & Onboarding* initiative). The entries below are an embedded copy synced from that document for runtime use. Updates land in Stratafy first, get signed off by Pieter, then engineering pushes the plugin update.


> **Q: Does this read my emails?**
> A: No. Claude only reads what you explicitly point it at — files you open or context from Stratafy when you ask. Your inbox isn't connected unless you connect it yourself.

> **Q: Can my manager see what I ask Claude?**
> A: No. Your conversations stay on your machine.

> **Q: What does the plugin report?**
> A: Install events, command-run counts (like "/clickatell:foundation was run 3 times"), and error counts. Not your conversation content. Run `/stratafy:status` to see exactly what's transmitted.

> **Q: Why are we using this?**
> A: [Clickatell-leadership-approved answer about strategic intent.]

> **Q: Who's managing this plugin?**
> A: Clickatell. The install is centrally managed, so it stays up to date and consistent across the team. If something feels off — the plugin's behaving oddly, reporting too much, or you'd just like to talk to someone about it — contact {{named-human-contact}} at {{contact-email}}.

> **Q: What if I have other questions?**
> A: Contact {{named-human-contact}} at {{contact-email}}.

> **Q: What's coming next?**
> A: Over the coming weeks, the plugin will learn more about your role and team so it can be more useful to you specifically. You'll be invited to set that up when it's ready — for now, the plugin just shares Clickatell's foundation with Claude.

## Local Files

All files live inside the user's project folder (NOT `$HOME`). In Claude Desktop / Cowork, only the selected project folder is visible to file tools; files in `$HOME` are silently invisible on the next run.

- `<project-root>/.clickatell/welcomed.json` — first-run state, written at end of full or recap
- `<project-root>/.clickatell/welcome-questions.log` — unanswered questions, append-only with consent

### Resolving the project root

Use Bash to resolve in this order:

1. `$CLAUDE_PROJECT_DIR` environment variable if set (Claude Code CLI)
2. The selected project folder's absolute path (Claude Desktop / Cowork — exposed in the system prompt)
3. `$(pwd)` as a final fallback

Standard snippet:

```bash
PROJECT_ROOT="${CLAUDE_PROJECT_DIR:-$(pwd)}"
mkdir -p "$PROJECT_ROOT/.clickatell"
```

Always pass the **absolute** path to Read/Write tools. Never pass `~/...` (tilde isn't expanded) or a bare `.clickatell/...` (treated as relative to wherever Write thinks it is, which in Cowork is the scratchpad, not the project).

## Rules

1. NEVER skip "what it isn't" — it's the load-bearing trust moment
2. NEVER promise behaviours the plugin can't actually deliver
3. ALWAYS get explicit consent before logging questions
4. ALWAYS point to the named human contact for unsolved questions
5. Voice = Clickatell, not Stratafy. Calm, factual, trustworthy.
