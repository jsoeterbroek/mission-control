# README.md

**Note:** This is an *example* workflow policy, USE AT YOUR OWN RISK.

## Assumptions
- You have a multiple-agent setup (with every agent a separate workspace folder)
- You have 'mc' installed
- you have a shared workspace: ~/.openclaw/workspace-shared
- Your Workflow Policy (prompt) lives here ~/.openclaw/workspace-shared/corporate/workflow-corporate-policy-v1.1.md
- You have a 'corporate' structure:

  - CEO  (human)
  - CEO  (main or default agent)
  - marketing/<agent>
  - legal/<agent>
  - finance/<agent>
  - etc.

- You have installed two skills:
  - workspace/skills/mc-day-summary
  - workspace/skills/mc-session-start

## Policy Hierarchy

```
Corporate WORKFLOW.md (highest — always applies)
        ↓
Department WORKFLOW.md (extends corporate; only overrides documented)
        ↓
Personal WORKFLOW.md (lowest — extends dept; personal deviations only)
```

Rules at lower levels may only *extend* or *specify*. They may **never remove or contradict** higher-level rules.

---

## Core Components

| Component | Description | Status |
|-----------|-------------|--------|
| **Mission Control (mc)** | CLI for task management & inter-agent messaging | ✅ Mandatory |
| **Heartbeat** | Periodic checks during business hours (07:00–18:00) | ✅ Mandatory |
| **Day Summary** | Daily 18:00 activity report — no exceptions | ✅ Mandatory |
| **Task Flow** | mc add → mc claim → mc start → mc done | ✅ Mandatory |
| **Approval Flow** | Formal approval gate for cross-agent tasks | ✅ Mandatory |

---

## Mission Control (mc)

All task management and inter-agent communication runs through mc. No workarounds.

### Setup

```bash
export PATH="$PATH:$HOME/.openclaw/mission-control"
```

### Essential Commands

```bash
# Tasks
mc add "Task description" --for <agent>   # Create and assign task
mc list                                      # List all tasks
mc board                                     # Kanban view
mc claim <id>                                # Claim task
mc start <id>                                # Start work
mc done <id> -m "what I did"              # Complete task

# Communication
mc msg <agent> "message"                    # Send message
mc inbox --unread                            # Check unread

# Status
mc fleet                                    # Who's active
mc feed --last 10                           # Recent activity
mc checkin                                  # Heartbeat check-in
```

---

## Session Start Protocol

**Execute before any other work:**

1. Read `workspace-shared/corporate/WORKFLOW.md`
2. Read your department's `WORKFLOW.md`
3. Run: `mc inbox --unread`
4. Run: `mc list --status pending`
5. Run: `mc checkin`

---

## Heartbeat

Heartbeat runs during business hours (07:00–18:00). Execute at **random minutes** within each interval to avoid load spikes.

| Task | Interval |
|------|----------|
| `mc checkin` | Every 30 min |
| `mc inbox --unread` | Every 30 min |
| Workflow policy check | Once daily (morning) |

---

## Task Flow

### Standard Flow

```
Task created (CEO, COO or agent)
       ↓
mc add "task" --for <agent>
       ↓
Agent: mc claim <id>
       ↓
Agent: mc start <id>
       ↓
Agent works
       ↓
Agent: mc done <id> -m "concise result"
       ↓
COO: reviews in daily feed
```

### Approval Flow (mandatory for cross-agent tasks)

```
Task ready to complete
       ↓
Does this affect another agent, team, or external party?
       ├── No ──→ mc done <id> -m "result"
       └── Yes ──→ Send approval request: mc msg <approver> "Proposal: <summary>. Approve?"
                        ├── Approved ──→ mc done + mc msg <affected_agent>
                        └── Rejected  ──→ mc msg <approver> "Acknowledged. [feedback]"
```

**Who approves what:**

| Task Type | Approver |
|-----------|----------|
| Tasks affecting another department | COO |
| Tasks with external comms or publication | COO → CEO |
| Tasks touching infrastructure / security | COO → CEO |
| Routine tasks within own dept | No approval needed |

---

## Day Summary

**Mandatory for every agent. No exceptions.**

Write at **18:00 local time** to:

```
workspace-shared/<DEPT>/activity_reports/YY-MM-DD-<AGENT>.md
```

### Template

```markdown
# Activity Report — [DATE] — [AGENT]

## Tasks Completed Today
- [mc task id + description + outcome]

## Ongoing Projects
- [project name: status update]

## Decisions & Documents
- [important decisions or documents created]

## Blockers
- [anything blocking progress]

## TODO Tomorrow
- [ ] [task or follow-up item]
```

### COO Fleet Summary

Main writes a fleet summary at **18:30** aggregating all reports:

```
workspace-shared/corporate/activity_reports/YY-MM-DD-fleet-summary.md
```

---

## Compliance Monitoring

Main (COO) monitors fleet compliance. Minimum expected per business day per agent:

| Signal | Minimum |
|--------|---------|
| `mc checkin` | 2× |
| `mc inbox --unread` | 2× |
| `mc done` (or `mc start`) | 1× |
| Day Summary file | 1× |

---

## Quick Reference

| Item | Location |
|------|----------|
| Full policy | `workspace-shared/corporate/workflow-corporate-policy-v1.1.md` |
| Department workflows | `workspace-shared/<DEPT>/WORKFLOW.md` |
| Activity reports | `workspace-shared/<DEPT>/activity_reports/` |
| MC CLI | `$HOME/.openclaw/mission-control/mc` |

---

*Last updated: 2026-03-20*
