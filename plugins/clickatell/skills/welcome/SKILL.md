---
name: "welcome"
description: "First-run induction for Claude at Clickatell. Walks the employee through what the plugin is, why Clickatell is rolling it out, what's local vs synced, honest boundaries, and answers their questions. Re-runnable. Use when the user runs /clickatell:welcome or when ~/.clickatell/welcomed.json doesn't exist and the user is interacting with the plugin."
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

- `~/.clickatell/welcomed.json` missing → full version
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

1. "Ask Claude about Clickatell's foundation — try `/stratafy:foundation`"
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

Open the floor. Answer from the curated Q&A below. For questions outside the set:

1. Ask consent: "Can I capture your question to share with the team?"
2. If yes → append to `~/.clickatell/welcome-questions.log` with timestamp
3. Always point to {{named-human-contact}} at {{contact-email}}

### 6. Wrap

Closing line. Mention `/clickatell:help` for anytime and `/clickatell:welcome` to re-run.

Write `~/.clickatell/welcomed.json` with version + timestamp + script version.

## Curated Q&A Reference (v1 — pending Clickatell sign-off)

> **Q: Does this read my emails?**
> A: No. Claude only reads what you explicitly point it at — files you open or context from Stratafy when you ask. Your inbox isn't connected unless you connect it yourself.

> **Q: Can my manager see what I ask Claude?**
> A: No. Your conversations stay on your machine.

> **Q: What does the plugin report?**
> A: Install events, command-run counts (like "/stratafy:foundation was run 3 times"), and error counts. Not your conversation content. Run `/stratafy:status` to see exactly what's transmitted.

> **Q: Why are we using this?**
> A: [Clickatell-leadership-approved answer about strategic intent.]

> **Q: Can I turn this off?**
> A: Yes. Settings → Plugins → Clickatell → Disable. Or uninstall entirely. No penalty.

> **Q: What if I have other questions?**
> A: Contact {{named-human-contact}} at {{contact-email}}.

> **Q: What's coming next?**
> A: Over the coming weeks, the plugin will learn more about your role and team so it can be more useful to you specifically. You'll be invited to set that up when it's ready — for now, the plugin just shares Clickatell's foundation with Claude.

## Local Files

- `~/.clickatell/welcomed.json` — first-run state, written at end of full or recap
- `~/.clickatell/welcome-questions.log` — unanswered questions, append-only with consent

## Rules

1. NEVER skip "what it isn't" — it's the load-bearing trust moment
2. NEVER promise behaviours the plugin can't actually deliver
3. ALWAYS get explicit consent before logging questions
4. ALWAYS point to the named human contact for unsolved questions
5. Voice = Clickatell, not Stratafy. Calm, factual, trustworthy.
