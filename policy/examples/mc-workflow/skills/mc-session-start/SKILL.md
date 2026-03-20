---
name: mc-session-start
description: Execute the Tayler-VLD session start protocol. Use this skill at the beginning of every work session to read workflow files, check Mission Control inbox and pending tasks, and brief yourself before starting work.
version: 1.0.0
metadata: {"openclaw":{"emoji":"🚀","os":["linux","darwin"],"requires":{"bins":["mc"]}}}
---

# Session Start Protocol

Execute these steps in order at the start of every work session. Do not skip any step.

## Step 1 — Identify yourself

Read your personal `WORKFLOW.md` (workspace root) or `MEMORY.md` to confirm:
- Your agent name (e.g. `Main`, `Dana`, `Max`, `Rob`, `Bas`)
- Your department (e.g. `corporate`, `marketing`, `it`, `security`, `finance`)
- Your `MC_AGENT` identifier (same as agent name)

## Step 2 — Read workflow files in priority order

1. `workspaces/workspace-shared/corporate/WORKFLOW.md` — always applies, read first
2. `workspaces/workspace-shared/<DEPARTMENT>/WORKFLOW.md` — dept-specific rules
3. Your personal `WORKFLOW.md` — personal exceptions only

Note the policy version. If a newer version of `workflow-corporate-policy-v*.md` exists in `workspaces/workspace-shared/corporate/`, flag it.

## Step 3 — Mission Control check-in

```bash
export PATH="$PATH:$HOME/.openclaw/mission-control"

MC_AGENT=<YOUR_AGENT> mc checkin
MC_AGENT=<YOUR_AGENT> mc inbox --unread
MC_AGENT=<YOUR_AGENT> mc list --status pending
mc feed --last 10
```

## Step 4 — Deliver session brief

Report back with:

- **Unread messages:** sender, subject/content
- **Pending tasks:** task ID, description, priority
- **Fleet activity:** anything relevant from the last 10 feed entries
- **Policy flags:** any workflow updates to note

## Step 5 — State your priority

Name your top priority task for this session and confirm you are ready to work.
