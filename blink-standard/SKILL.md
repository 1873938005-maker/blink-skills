---
name: blink-standard
description: Standard (tool-free) proactive reflection that reviews recent context and decides whether to stay silent, suggest help, or propose an autonomous task.
metadata: { "openclaw": { "emoji": "👁️", "requires": { "bins": [] } } }
---

# Blink (Standard)

## When to use (trigger phrases)

Use this skill when the user asks for any of:

- “blink”
- “主动反思 / 自我审查”
- “proactive reflection / self prompt”
- “让 agent 主动发现能做的事情/主动和我交互”

## Goal

Run a lightweight self-reflection step based on:

- The most recent messages in the current conversation (what you already see)
- A quick workspace snapshot (high-level, low-token)
- Optional: recent bullet points from `MEMORY.md` if present

Then output exactly one of:

- `[[NOTHING]]`
- `[[SUGGESTION]] <text>`
- `[[AUTONOMOUS_TASK]] <text>`

## Procedure (do this every time)

1. Build a **Context Snapshot** (keep it short):
   - Last user request and your last response (if any)
   - Current TODOs or unresolved issues you can infer
   - Workspace snapshot:
     - If the user has an active repo/workspace, list up to ~15 relevant files/dirs you already know are involved.
     - If you are unsure, do a quick directory listing of the workspace root and/or relevant subfolder(s), but keep it minimal.
   - If `{baseDir}/../../MEMORY.md` exists in the workspace (or a `MEMORY.md` near the project root), read only a small tail section (e.g., last 10 bullet lines) and include them.

2. Decide if you should act proactively:
   - Default to **quiet**.
   - Only propose something if it saves time or avoids an error.
   - Prefer `[[SUGGESTION]]` over `[[AUTONOMOUS_TASK]]` unless the task is very small and clearly safe.

3. Output format (MANDATORY):
   - If no action is needed: respond with **ONLY** `[[NOTHING]]`.
   - Otherwise:
     - `[[SUGGESTION]] ...` or
     - `[[AUTONOMOUS_TASK]] ...`

## Safety / scope

- Do not propose destructive actions.
- Do not run long workflows automatically.
- Keep the reflection short and non-intrusive.
