---
name: jira-health-check
description: Scan Jira for project hygiene issues — stale tickets, missing dates, overdue items, empty epics, status mismatches. Use when the user says "jira health check", "check jira health", "project hygiene", or as a proactive check during end-week.
---

# Jira Health Check

Scans team Jira tickets for hygiene issues. Resolve Jira project key and custom field IDs from [shared/config.md](../shared/config.md) and produces a concise health report. Catches problems before they become fires.

## Decision Flow

```
┌──────────────────────────────────────────┐
│  Step 1: Run Core Checks (parallel)      │
│  ┌────────────┬────────────┬───────────┐ │
│  │ Check 1:   │ Check 2:   │ Check 3:  │ │
│  │ Stale      │ Missing    │ Overdue   │ │
│  │ tickets    │ dates      │ items     │ │
│  └────────────┴────────────┴───────────┘ │
└──────────────────┬───────────────────────┘
                   │
        ┌──────────▼──────────┐
        │ Quick mode?         │── Yes ──▶ Skip to Step 3
        └──────────┬──────────┘
                   │ No (default)
┌──────────────────▼───────────────────────┐
│  Step 2: Run Deep Checks                 │
│  ┌─────────────────┬───────────────────┐ │
│  │ Check 4:        │ Check 5:          │ │
│  │ Empty epics     │ Status            │ │
│  │                 │ misalignment      │ │
│  │ ┌─────────────┐ │ ┌───────────────┐ │ │
│  │ │ >20 epics?  │ │ │ For each      │ │ │
│  │ │ Yes: limit  │ │ │ project:      │ │ │
│  │ │ to In Prog  │ │ │ compare parent│ │ │
│  │ │ only        │ │ │ vs children   │ │ │
│  │ └─────────────┘ │ └───────────────┘ │ │
│  └─────────────────┴───────────────────┘ │
└──────────────────┬───────────────────────┘
                   │
        ┌──────────▼──────────┐
        │ Step 3: Score health │
        │ Healthy / Needs      │
        │ Attention / Action   │
        │ Required             │
        └──────────┬──────────┘
                   │
        ┌──────────▼──────────┐
        │ Step 3: Generate     │
        │ health report        │
        └──────────┬──────────┘
                   │
        ┌──────────▼──────────┐
        │ Step 4: Offer next   │
        │ steps (comment on    │
        │ stale? fix statuses? │
        │ add to priorities?)  │
        └─────────────────────┘
```

## Workflow

### Step 1: Run Core Health Checks (3 JQL queries)

Run these three queries in parallel. All use the Cloud ID from [shared/config.md](../shared/config.md).

#### Check 1: Stale In-Progress Tickets

Tickets actively being worked on but with no updates in 14+ days.

```
project = {{JIRA_PROJECT_KEY}} AND statusCategory = "In Progress" AND updated <= -14d ORDER BY updated ASC
```

**Fields:** `summary, status, assignee, updated, issuetype, duedate`

**Flag if:** Any results returned. Group by assignee for actionability.

#### Check 2: Missing Target Dates

Project, Milestone, and Epic tickets that are not Done and have no Target End date set. Makes timeline planning unreliable.

```
project = {{JIRA_PROJECT_KEY}} AND issuetype in (Project, Epic) AND statusCategory != Done AND duedate is EMPTY ORDER BY issuetype ASC, created DESC
```

**Fields:** `summary, status, issuetype, assignee, {{TARGET_START_FIELD}}, {{TARGET_END_FIELD}}, duedate`

**Flag if:** Any results returned. Separate into: (a) missing both start and end dates, (b) missing only end date.

#### Check 3: Overdue Items

Tickets past their due date that aren't Done.

```
project = {{JIRA_PROJECT_KEY}} AND duedate < now() AND statusCategory != Done ORDER BY duedate ASC
```

**Fields:** `summary, status, assignee, duedate, issuetype`

**Flag if:** Any results returned. Include how many days overdue each ticket is.

### Step 2: Run Deep Health Checks (optional, multi-query)

These checks require additional API calls. Run them by default unless the user asks for a quick check.

#### Check 4: Empty Epics (no child tickets)

Epics that exist but have no linked children — may be placeholders that need cleanup or tickets that need to be broken down.

1. Query all non-Done epics:
   ```
   project = {{JIRA_PROJECT_KEY}} AND issuetype = Epic AND statusCategory != Done ORDER BY created DESC
   ```
2. For each epic, query children:
   ```
   parent = [EPIC-KEY]
   ```
3. **Flag if:** Epic has 0 children. Include epic key, summary, and status.

**Performance note:** If more than 20 epics are returned, limit children checks to epics in "In Progress" status category only and note the scope limitation in the report.

#### Check 5: Status Misalignment

Projects where the project-level status doesn't match what the children suggest.

1. Query all non-Done projects:
   ```
   project = {{JIRA_PROJECT_KEY}} AND issuetype = Project AND statusCategory != Done ORDER BY status ASC
   ```
2. For each project, query children:
   ```
   parent = [PROJECT-KEY] ORDER BY created ASC
   ```
3. Flag misalignment:
   - **Project says "In Progress" but all children are "Backlog"** — work hasn't actually started
   - **Project says "Backlog" but some children are "In Progress"** — status needs updating
   - **Project says "In Progress" but all children are "Done"** — project may be ready to close

### Step 3: Generate Health Report

Present the report in the conversation. Format:

```markdown
## Jira Health Check — [Date]

### Summary
- **Stale tickets:** [count] ([X] critical — 30+ days, [Y] warning — 14-29 days)
- **Missing dates:** [count] ([X] projects, [Y] epics)
- **Overdue items:** [count]
- **Empty epics:** [count]
- **Status misalignment:** [count]
- **Overall health:** [Healthy / Needs Attention / Action Required]

---

### [Section for each check with issues found]

#### Stale In-Progress Tickets ([count])
| Ticket | Summary | Assignee | Status | Last Updated | Days Stale |
|--------|---------|----------|--------|--------------|------------|
| [linked] | ... | ... | ... | ... | ... |

**Recommendation:** [Specific action — e.g., "Ping assignees for status update" or "Consider moving back to Backlog if blocked"]

#### Missing Target Dates ([count])
...

#### Overdue Items ([count])
...

#### Empty Epics ([count])
...

#### Status Misalignment ([count])
...

---

### Recommended Actions
1. [Most urgent action]
2. [Second priority]
3. ...
```

### Step 4: Offer Next Steps

After presenting the report, ask:

1. "Want me to add comments to any stale tickets to request status updates?"
2. "Want me to fix any status misalignments in Jira?"
3. "Should I log any of these as items in this week's priorities?"

## Overall Health Scoring

| Score | Criteria |
|-------|----------|
| **Healthy** | 0 overdue, 0 stale > 30 days, ≤ 2 missing dates |
| **Needs Attention** | 1-3 overdue OR any stale > 30 days OR 3-5 missing dates |
| **Action Required** | 4+ overdue OR any stale > 45 days OR 6+ missing dates OR status misalignments on active projects |

## Configuration

| Parameter | Default | Notes |
|-----------|---------|-------|
| Stale threshold (warning) | 14 days | Tickets updated more than 14 days ago |
| Stale threshold (critical) | 30 days | Highlighted separately in report |
| Epic children check limit | 20 epics | If more than 20, limit to In Progress only |
| Quick mode | Off | When on, skip Step 2 deep checks |

The user can override these per-run (e.g., "run health check with 7 day stale threshold").

## Integration Points

- **End of week:** Can be triggered as part of the end-week skill. If issues are found, they can be flagged in the weekly update.
- **Start of week:** Useful to run at start of week to prioritize hygiene fixes.
- **Self-improvement loop:** If the same type of issue appears in 3+ consecutive health checks, log it in `IMPROVEMENTS.md` as a process gap that needs addressing.

## Error Handling

See [shared/config.md](../shared/config.md) for full error handling guidelines.

- **Authentication errors**: Follow `atlassianMcpErrors.mdc` rule
- **JQL query failure**: Report which check failed, proceed with remaining checks. Don't abort the entire health check for a single query failure.
- **Rate limiting**: If hitting rate limits during deep checks (Step 2), pause 30 seconds between epic children queries. Report partial results if rate limiting persists.

## Validation Checklist

- [ ] Check 1 (stale tickets) query executed
- [ ] Check 2 (missing dates) query executed
- [ ] Check 3 (overdue items) query executed
- [ ] Check 4 (empty epics) executed (unless quick mode)
- [ ] Check 5 (status misalignment) executed (unless quick mode)
- [ ] Health report generated with summary scores
- [ ] Each finding includes: ticket link, summary, assignee, specific issue
- [ ] Recommendations are specific and actionable
- [ ] Recommended actions offered to user
- [ ] If recurring issues detected (3+ consecutive checks), logged to IMPROVEMENTS.md
