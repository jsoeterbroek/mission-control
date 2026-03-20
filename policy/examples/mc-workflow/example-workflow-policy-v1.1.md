# Workflow Policy

**Version:** 1.1
**Date:** N/A
**Author:** N/A
**Status:** Active
**Supersedes:** v1.0

---

## What Changed in v1.1

This version promotes the "Recommended Improvements" from v1.0 into actual **policy directives**. Nothing is pending — these are now required behaviors. Additionally:

- Day Summary is now mandatory, not aspirational
- Department WORKFLOW.md standard is defined and enforced
- Approval flow is fully formalized with clear ownership
- Heartbeat intervals are tightened
- Implementation scripts are provided to bootstrap compliance

---

## Policy Hierarchy

```
Corporate WORKFLOW.md  (highest — always applies)
        ↓
Department WORKFLOW.md  (extends corporate; only overrides must be documented)
        ↓
Personal WORKFLOW.md  (lowest — extends dept; only personal deviations allowed)
```

**Rule:** Lower levels may only *extend* or *specify*. They may **never remove or contradict** a higher-level rule.

---

## Core Components

| Component | Description | Status |
|-----------|-------------|--------|
| **COORDINATION** | Mission Control CLI for tasks & messaging | ✅ Mandatory |
| **HEARTBEAT** | Periodic checks during business hours | ✅ Mandatory |
| **DAY SUMMARY** | Daily 18:00 activity report — no exceptions | ✅ Mandatory |
| **TASK FLOW** | mc add → mc claim → mc start → mc done | ✅ Mandatory |
| **APPROVAL FLOW** | Formal approval gate for cross-agent tasks | ✅ Mandatory |

---

## Mission Control (mc)

Mission Control is the coordination layer for all OpenClaw agents. All task management and inter-agent communication must go through mc. No workarounds.

### Path Setup

```bash
export PATH="$PATH:$HOME/.openclaw/mission-control"
```

This must be set in each agent's environment or shell profile.

### Task Commands

```bash
mc add "Task description" --for <agent>   # Create and assign task
mc list                                    # List all tasks
mc board                                   # Kanban view
mc claim <id>                              # Claim task
mc start <id>                              # Start work
mc done <id> -m "what I did"              # Complete task with summary
```

### Communication Commands

```bash
mc msg <agent> "message"                   # Send message to agent
mc inbox --unread                          # Check unread messages
```

### Status Commands

```bash
mc fleet                                   # Fleet overview (who's active)
mc feed --last 10                          # Recent activity across fleet
mc checkin                                 # Heartbeat check-in
```

### COO (Main) Start-of-Day Sequence

```bash
MC_AGENT=Main mc inbox --unread
MC_AGENT=Main mc list --status pending
mc feed --last 20
```

---

## Session Start Protocol

Every agent must execute this on session start, **before any other work**:

1. Read this corporate WORKFLOW.md
2. Read your department's WORKFLOW.md (if it has dept-specific rules)
3. Read your personal WORKFLOW.md
4. Run: `MC_AGENT=<self> mc inbox --unread`
5. Run: `MC_AGENT=<self> mc list --status pending`
6. Run: `mc checkin`

---

## Heartbeat

Heartbeat runs during business hours **07:00–18:00**. Each agent checks-in at **random minutes within each interval** to avoid load spikes. The heartbeat must be configured in `~/.openclaw/workspace/HEARTBEAT.md`.

### Heartbeat Schedule

| Task | Interval | Notes |
|------|----------|-------|
| `mc checkin` | Every 30 min | Required — proves liveness |
| `mc inbox --unread` | Every 30 min | Required — follow up outstanding |
| Check for workflow policy updates | Once daily (morning) | On session start |

### Heartbeat Configuration Template (`HEARTBEAT.md`)

```markdown
# HEARTBEAT.md — <AGENT_NAME>

## Schedule
- 30min: mc checkin + mc inbox --unread
- daily: workflow policy check

## Notes
- <Agent-specific exceptions or overrides>
```

---

## Task Flow

### Standard Flow

```
Task created (agent)
       ↓
mc add "task" --for <agent>
       ↓
Agent: mc claim <id>
       ↓
Agent: mc start <id>
       ↓
Agent works
       ↓
Agent: mc done <id> -m "concise result summary"
       ↓
(COO): reviews in daily feed + aggregates in Day Summary
```

### Approval Flow (for tasks with cross-agent or external impact)

The following decision tree is **mandatory** for all task completions:

```
Task ready to complete
       ↓
Does this affect another agent, team, or external party?
       ├── No ──→ mc done <id> -m "result"
       └── Yes ──→ Send approval request: mc msg <approver> "Proposal: <summary>. Approve?"
                        ├── Approved ──→ mc done <id> -m "result" + mc msg <affected_agent> "task X approved, proceeding"
                        └── Rejected  ──→ mc msg <approver> "Acknowledged. <revised plan or question>"
```

**Who approves what:**

| Task Type | Approver |
|-----------|----------|
| Tasks affecting another department | COO |
| Tasks with external comms or publication | COO → CEO |
| Tasks touching infrastructure / security | COO  → CEO |
| Routine tasks within own dept | No approval needed |

**Example (the #9→#10 pattern):**
1. Dana completes task #9 (website review proposal)
2. `mc msg Main "Proposal: [doc location]. Needs your review for #10 handoff to Max."`
3. COO reviews, escalate if needed
4. If approved: `mc add "Website updates" --for it-coder` + `mc msg Max "Task created #10 — see proposal at [location]"`
5. If rejected: `mc msg Dana "Not approved — reason: [X]. Please revise."`

---

## Day Summary

**This is mandatory. No exceptions.**

Every agent writes a Day Summary at **18:00 local time**.

### File Location

```
workspace-shared/<DEPARTMENT>/activity_reports/YY-MM-DD-<AGENTNAME>.md
```

Examples:
- `workspace-shared/corporate/activity_reports/26-03-20-main.md`
- `workspace-shared/marketing/activity_reports/26-03-20-dana.md`
- `workspace-shared/it/activity_reports/26-03-20-max.md`

### Template

```markdown
# Activity Report — [DATE] — [AGENT]

## Tasks Completed Today
- [mc task id + description + outcome]

## Ongoing Projects
- [project name: status update]

## Decisions & Documents
- [any important decisions made or documents created]

## Blockers
- [anything blocking progress — for COO attention]

## TODO Tomorrow
- [ ] [task or follow-up item]
```

### COO Aggregation

Main agent (COO) writes a daily fleet summary at 18:30 (after individual reports):

```
workspace-shared/corporate/activity_reports/YY-MM-DD-fleet-summary.md
```

The fleet summary aggregates cross-departmental highlights, flags blockers, and lists next-day priorities.

---

## Department WORKFLOW.md Standard

Each department workspace must have a `WORKFLOW.md` that:

1. References this corporate policy
2. Documents **only** department-specific deviations or additions
3. Is maintained by the department's lead agent

### Required Template

```markdown
# [DEPARTMENT] WORKFLOW.md

This department follows the corporate workflow policy.
See: workspace-shared/corporate/WORKFLOW.md (always read that first)

## Department: [DEPARTMENT NAME]
**Lead Agent:** [agent name]
**Agents:** [list]

## Department-Specific Tasks

[List any recurring tasks unique to this department — if none, write "None"]

## Deviations from Corporate Policy

[Document any approved exceptions here — if none, write "None"]

## Department Heartbeat Additions

[Any extra heartbeat tasks — if none, write "None"]
```

**Existing files that are invalid copies of corporate policy must be replaced with this template.**

---

## Workflow Execution Order

On every session:

1. Read WORKFLOW.md files (corporate → dept → personal)
2. Check for policy updates (version number in filename)
3. Run session start protocol (mc inbox, mc list, mc checkin)
4. Execute heartbeat tasks at configured intervals
5. Work on assigned tasks, follow task flow and approval flow
6. Write Day Summary at 18:00

---

## Compliance & Monitoring

Main agent (COO) monitors compliance via:

```bash
mc feed --last 50                  # Check recent agent activity
mc fleet                           # Who is active / last seen
```

Agents who are not checking in regularly should be flagged to CEO.

Minimum expected signals per business day per agent:

| Signal | Minimum |
|--------|---------|
| `mc checkin` | 2× |
| `mc inbox --unread` | 2× |
| `mc done` (or `mc start`) | 1× |
| Day Summary file | 1× |

---

## Implementation Scripts

### Script 1: Bootstrap Activity Report Directories

Run once to create all `activity_reports/` directories:

```bash
#!/usr/bin/env bash
# bootstrap-activity-reports.sh
# Creates activity_reports folders for all departments

BASE="workspaces/workspace-shared"

DEPARTMENTS=(corporate marketing it finance security business legal)

for dept in "${DEPARTMENTS[@]}"; do
  dir="$BASE/$dept/activity_reports"
  mkdir -p "$dir"
  echo "Created: $dir"
done

echo "Done. All activity_reports directories created."
```

### Script 2: Write Day Summary (agent use)

Each agent can use this as a starting template:

```bash
#!/usr/bin/env bash
# write-day-summary.sh
# Usage: ./write-day-summary.sh <AGENT_NAME> <DEPARTMENT>
# Example: ./write-day-summary.sh main corporate

AGENT="${1:-unknown}"
DEPT="${2:-corporate}"
DATE=$(date +%y-%m-%d)
LONGDATE=$(date +%Y-%m-%d)
FILE="workspaces/workspace-shared/$DEPT/activity_reports/$DATE-$AGENT.md"

if [ -f "$FILE" ]; then
  echo "Day summary already exists: $FILE"
  exit 1
fi

cat > "$FILE" << EOF
# Activity Report — $LONGDATE — $AGENT

## Tasks Completed Today
-

## Ongoing Projects
-

## Decisions & Documents
-

## Blockers
-

## TODO Tomorrow
- [ ]
EOF

echo "Created: $FILE"
```

### Script 3: Check Compliance (COO use)

```bash
#!/usr/bin/env bash
# check-compliance.sh
# Reports which agents have NOT written a Day Summary today

BASE="workspaces/workspace-shared"
DATE=$(date +%y-%m-%d)

AGENTS=(
  "main:corporate"
  "dana:marketing"
  "max:it"
  "bas:finance"
  "rob:security"
)

echo "=== Day Summary Compliance Check: $DATE ==="
echo ""

for entry in "${AGENTS[@]}"; do
  AGENT="${entry%%:*}"
  DEPT="${entry##*:}"
  FILE="$BASE/$DEPT/activity_reports/$DATE-$AGENT.md"
  if [ -f "$FILE" ]; then
    echo "  ✅ $AGENT ($DEPT)"
  else
    echo "  ❌ $AGENT ($DEPT) — MISSING"
  fi
done

echo ""
echo "Run from repo root."
```

### Script 4: Validate Department WORKFLOW.md Files

```bash
#!/usr/bin/env bash
# validate-workflows.sh
# Checks that department WORKFLOW.md files reference corporate policy
# and are not stale copies

BASE="workspaces/workspace-shared"
CORPORATE_REF="workspace-shared/corporate/WORKFLOW.md"
PROBLEMS=0

echo "=== Workflow File Validation ==="
echo ""

for wf in "$BASE"/*/WORKFLOW.md "$BASE"/*/*/WORKFLOW.md; do
  [ -f "$wf" ] || continue
  [[ "$wf" == *"/corporate/WORKFLOW.md" ]] && continue

  if grep -q "$CORPORATE_REF" "$wf"; then
    echo "  ✅ $wf"
  else
    echo "  ❌ $wf — does not reference corporate policy"
    PROBLEMS=$((PROBLEMS + 1))
  fi
done

echo ""
if [ "$PROBLEMS" -gt 0 ]; then
  echo "Found $PROBLEMS workflow file(s) needing attention."
  exit 1
else
  echo "All workflow files are compliant."
fi
```

---

## Known Issues (from v1.0 evaluation)

| 1 | Calendar not implemented | Tracked as pending (heartbeat task marked TBA) |
| 2 | Approval flow was informal | Formal decision tree + ownership table added |

---

## Open Items

| # | Item | Owner | Priority |
|---|------|-------|----------|
| 2 | Implement Calendar integration | CEO/COO | MEDIUM |
| 3 | Replace invalid dept WORKFLOW.md files | CEO | HIGH |
| 4 | Run bootstrap-activity-reports.sh | CEO | HIGH |
| 5 | Validate all dept workflows (script 4) | CEO | MEDIUM |

---

## Appendix: mc Quick Reference

```bash
export PATH="$PATH:$HOME/.openclaw/mission-control"

# Tasks
mc add "Task" --for <agent>        # Create + assign
mc list                             # All tasks
mc board                            # Kanban view
mc claim <id>                       # Claim task
mc start <id>                       # Start work
mc done <id> -m "what I did"       # Complete

# Communication
mc msg <agent> "message"            # Send message
mc inbox --unread                   # Check unread

# Status
mc fleet                            # Fleet overview
mc feed --last 10                   # Recent activity
mc checkin                          # Heartbeat check-in
```

---

*Document created: 2026-03-20*
*Version 1.1 — supersedes v1.0*
