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

1. Build a **Context Snapshot** (STRICT token limits - max ~1200 tokens for context):
   - **Recent conversation**: Use ONLY the last user message + your last response (2 turns max). Do not include earlier history.
   - **Workspace**: List at most **5** files/directories. Only include files you know are actively being worked on. If unsure, skip entirely rather than listing everything.
   - **MEMORY.md**: Only read if the last user message explicitly mentions memory, TODO, or unresolved tasks. If reading, limit to **last 3 bullet lines** maximum.
   
   ⚠️ **Hard Constraint**: If context would exceed ~1200 tokens, prioritize: conversation (keep) > file list (truncate first) > memory bullets (drop if needed).

2. Decide if you should act proactively:
   - Default to **quiet**.
   - Only propose something if it saves time or avoids an error.
   - Prefer `[[SUGGESTION]]` over `[[AUTONOMOUS_TASK]]` unless the task is very small and clearly safe.

3. Output format (MANDATORY - be concise):
   - If no action is needed: respond with **ONLY** `[[NOTHING]]`.
   - Otherwise:
     - `[[SUGGESTION]] <brief suggestion under 100 tokens>` or
     - `[[AUTONOMOUS_TASK]] <brief task description under 100 tokens>`
   
   Keep all outputs short. One sentence is enough.

## Safety / scope

- Do not propose destructive actions.
- Do not run long workflows automatically.
- Keep the reflection short and non-intrusive.
