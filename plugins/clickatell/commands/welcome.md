---
description: "Universal day-zero induction for every Clickatell employee — what just got installed and why"
---

# /clickatell:welcome

The universal welcome for Clickatell employees. ~5 minutes. Re-runnable.

## Process

### Step 0: Pin Clickatell + Load User Context (single call)

ALWAYS first. So any organic conversation that follows the welcome is anchored on Clickatell, regardless of what workspace the user had selected before.

Call `get_user_context` with:

- `workspace_id`: `"f06499c2-a2a8-4e7d-ad02-c66d6fd46873"` (Clickatell — pins the session as a side effect)
- `command_name`: `"welcome"`
- `plugin_name`: `"clickatell"`
- `_llm_model`, `_intent: "user_request"`, `_reason`, `_source_plugin: "clickatell"`, `_source_command: "welcome"`

Do NOT call `select_workspace` separately — `get_user_context` with `workspace_id` does both jobs in one round-trip.

### Step 1: Check First-Run State

Check `~/.clickatell/welcomed.json`:

- If absent → run the full version (Steps 2–7)
- If present → ask: "You've done this before. Want the full version or a quick recap?"
  - Full → Steps 2–7
  - Quick recap → Steps 2, 5, 7 only (welcome, what-it-isn't reminder, wrap)

### Step 2: Open

Open with a short, warm welcome IN CLICKATELL'S VOICE (not Stratafy's). Acknowledge that something new just got installed and that the next 5 minutes will explain what it is.

> [Welcome script v1 — Clickatell-signed-off content goes here once approved by Pieter or designated comms reviewer.]

### Step 3: The Why

Two-to-three sentences from Clickatell leadership about why they're rolling this out. Concrete, not abstract. Must be in Clickatell's voice, signed off by Pieter or comms.

> [Why-script v1 — placeholder until Pieter signs off.]

### Step 4: The What

Three concrete things employees can ask Claude now that they couldn't before:

1. *"Ask Claude about Clickatell's foundation — try `/stratafy:foundation`"*
2. *"Ask Claude to draft something that reflects our values"*
3. *"Ask Claude what it knows about Clickatell"*

### Step 5: What It Isn't

Honest boundaries. Each line must be true and verifiable:

- Doesn't read your emails or messages unless you point it at them
- Doesn't report your activity to your manager, IT, or Clickatell
- Conversations stay on your machine
- Doesn't have all the answers — Stratafy is being populated, some context will be thin in the early weeks
- Not a replacement for talking to your team

### Step 6: Questions

Open the floor: "Any questions about how this works, what's connected, or what we'll do with it?"

Answer from the curated Q&A reference in the `welcome` skill. For questions outside that set:

1. Ask consent: "I don't have an answer to that one. Can I capture your question to share with the team?"
2. If yes → append to `~/.clickatell/welcome-questions.log` with timestamp
3. Always point to {{named-human-contact}} at {{contact-email}}

### Step 7: Wrap

Single closing line in Clickatell's voice. Mention `/clickatell:help` for any time and `/clickatell:welcome` to re-run.

Then write `~/.clickatell/welcomed.json`:

```json
{
  "version": "1.0",
  "welcomed_at": "{{ISO8601 timestamp}}",
  "welcome_script_version": "1.0"
}
```

## Voice

Throughout this command, speak in CLICKATELL'S voice, not Stratafy's. The welcome script is co-authored with Clickatell and signed off by Pieter or designated comms.

## Rules

1. NEVER skip "what it isn't" — it's the load-bearing trust moment
2. NEVER promise behaviours the plugin can't actually deliver
3. ALWAYS get explicit consent before logging questions
4. ALWAYS point to the named human contact for unsolved questions
5. Re-runs default to recap; full version is opt-in
