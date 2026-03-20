---
name: mc-day-summary
description: Write the end-of-day activity report for your department. Use this skill at 18:00 to gather today's Mission Control activity, write the structured report to the correct activity_reports folder, and notify the COO. COO (Main) also writes a fleet-wide summary.
version: 1.0.0
metadata: {"openclaw":{"emoji":"📋","os":["linux","darwin"],"requires":{"bins":["mc"]}}}
---

# Day Summary

Execute these steps at 18:00 every business day. This report is mandatory — no exceptions.

## Step 1 — Identify yourself

Confirm from your `WORKFLOW.md` or `MEMORY.md`:
- Agent name (lowercase for filename, e.g. `main`, `dana`, `max`, `rob`, `bas`)
- Department (e.g. `corporate`, `marketing`, `it`, `security`, `finance`)
- `MC_AGENT` identifier

## Step 2 — Gather today's activity from Mission Control

```bash
export PATH="$PATH:/home/jsoeterbroek/.openclaw/mission-control"

MC_AGENT=<YOUR_AGENT> mc list --status done
MC_AGENT=<YOUR_AGENT> mc list --status in-progress
MC_AGENT=<YOUR_AGENT> mc inbox --unread
mc feed --last 30
```

Capture: completed task IDs + their `-m` summaries, in-progress tasks, unresolved messages.

## Step 3 — Determine output path

```
workspaces/workspace-shared/<DEPARTMENT>/activity_reports/YY-MM-DD-<agentname>.md
```

Example: `workspaces/workspace-shared/corporate/activity_reports/26-03-20-main.md`

If the `activity_reports/` directory does not exist, create it: `mkdir -p <path>`

If the file already exists, append rather than overwrite.

## Step 4 — Write the report

```markdown
# Activity Report — [YYYY-MM-DD] — [AGENT NAME]

## Tasks Completed Today
- [task id]: [description] — [outcome from mc done -m summary]

## Ongoing Projects
- [project/task]: [status, what's next]

## Decisions & Documents
- [decisions made, documents created, approvals given or received]

## Blockers
- [anything blocking progress for COO attention — or "None"]

## TODO Tomorrow
- [ ] [concrete next action]
```

Use only real data from the mc output. Do not invent tasks. If nothing was completed today, say so.

## Step 5 — Notify and check in

```bash
MC_AGENT=<YOUR_AGENT> mc msg Main "Day summary written: workspaces/workspace-shared/<DEPARTMENT>/activity_reports/YY-MM-DD-<agentname>.md"
MC_AGENT=<YOUR_AGENT> mc checkin
```

**If you are Main (COO):** skip the `mc msg` step. Instead, write a fleet summary at:
`workspaces/workspace-shared/corporate/activity_reports/YY-MM-DD-fleet-summary.md`

The fleet summary aggregates highlights from all agents' reports, flags blockers, and lists tomorrow's cross-department priorities. Check each agent's report file before writing it.

## Step 6 — Confirm

State the file path you wrote and a one-sentence summary of your day.
