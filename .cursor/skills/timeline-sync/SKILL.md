---
name: timeline-sync
description: Sync project-timeline.md from Jira as the source of truth. Produces two views — a detailed epic/milestone Gantt for active projects and a project-level 6-month roadmap including backlog/planned projects. Use when the user says "sync timeline", "update timeline", or as an optional step during end-week.
---

# Timeline Sync

Regenerates `project-timeline.md` entirely from Jira data.

**Config:** Resolve Jira project key (`{{JIRA_PROJECT_KEY}}`), custom field IDs (`{{TARGET_START_FIELD}}`, `{{TARGET_END_FIELD}}`), and Atlassian site URL (`{{ATLASSIAN_SITE}}`) from [shared/config.md](../shared/config.md). Produces two views: a 6-month roadmap at project granularity and detailed Gantt charts at epic/milestone granularity for each active project.

**Jira is the sole source of truth** for all dates, statuses, and assignees. Done projects are excluded from the timeline entirely.

## Decision Flow

```
┌───────────────────────────────────┐
│ Step 0: Fetch Jira data +         │
│ compare against current timeline  │
└──────────────┬────────────────────┘
               │
    ┌──────────▼──────────┐
    │ Timeline-relevant   │── No ──▶ Report "no changes",
    │ data changed?       │          skip (unless forced)
    └──────────┬──────────┘
               │ Yes
    ┌──────────▼──────────┐
    │ Step 1: Query Jira  │
    │ for all non-Done    │
    │ Project tickets     │
    └──────────┬──────────┘
               │
    ┌──────────▼──────────┐
    │ Categorize:         │
    │ Active / Planned /  │
    │ Backlog             │
    └──────────┬──────────┘
               │
    ┌──────────▼───────────────────────┐
    │ Step 2: Fetch children           │
    │ (ONLY for active projects)       │
    │ Project → Milestones → Epics     │
    │                                  │
    │ Planned/backlog: project-level   │
    │ data only (already from Step 1)  │
    └──────────┬───────────────────────┘
               │
    ┌──────────▼──────────────────────────┐
    │ Step 3–4: Build views (ALL data     │
    │ collected before generating output) │
    │ ┌────────┬──────────┬─────────────┐ │
    │ │View 3: │ View 2:  │ View 1:     │ │
    │ │Visual  │ 6-month  │ Detailed    │ │
    │ │roadmap │ roadmap  │ per-project │ │
    │ │table   │ Gantt    │ Gantt       │ │
    │ └────────┴──────────┴─────────────┘ │
    └──────────┬──────────────────────────┘
               │
    ┌──────────▼──────────┐
    │ Step 5: Present     │
    │ summary + flag      │
    │ missing dates /     │
    │ fetch failures      │
    └──────────┬──────────┘
               │
    ┌──────────▼──────────┐
    │ User approves?      │── No ──▶ Stop, do not write
    └──────────┬──────────┘
               │ Yes
    ┌──────────▼──────────┐
    │ Step 6: Overwrite   │
    │ project-timeline.md │
    └─────────────────────┘
```

## Workflow

### Step 0: Check Whether Timeline Data Changed (MANDATORY)

Before regenerating `project-timeline.md`:

1. Fetch Jira project data for timeline sync (Step 1 + required child ticket fetches for active projects)
2. Compare the newly fetched scheduling/status fields against the current timeline data snapshot:
   - project/epic/milestone status
   - start/end/due dates (`{{TARGET_START_FIELD}}`, `{{TARGET_END_FIELD}}`, `duedate` — resolve field IDs from shared/config.md)
   - ticket additions/removals relevant to timeline sections
3. If there are **no timeline-relevant changes**, do **not** rewrite `project-timeline.md`
4. Report "No Jira timeline changes detected" to the user and skip to completion unless the user explicitly asks for a forced rebuild

### Step 1: Query Jira for All Non-Done Projects

1. Query Jira directly for all Project-type tickets in the project that are not Done:
   ```
   project = {{JIRA_PROJECT_KEY}} AND issuetype = Project AND statusCategory != Done ORDER BY status ASC, created DESC
   ```
2. This is the **single source of truth** for which projects to include — do NOT rely on `all-projects.md` for the project list
3. Categorize results:
   - **Active**: status category = "In Progress" (e.g., Delivering, In Progress)
   - **Planned**: status = "Planned"
   - **Backlog**: status = "Backlog" or "To Do"

### Step 2: Fetch Children for Active Projects (MANDATORY)

**For each active project** (from Step 1):
1. Use JQL to get all children (milestones + epics):
   ```
   parent = [PROJECT-KEY] ORDER BY created ASC
   ```
   Then for each milestone, get its children (epics):
   ```
   parent = [MILESTONE-KEY] ORDER BY created ASC
   ```
2. For each ticket collect: key, summary, status, assignee, due date, target start (`{{TARGET_START_FIELD}}`), target end (`{{TARGET_END_FIELD}}`), created date, resolution date

**Planned/backlog projects** already have their project-level data from Step 1 — no children needed for the roadmap view.

**Collect ALL data before generating any output.**

### Step 2b: Compute Pre-Check Summary for Specialist

Before building the views, compute deterministic facts from the fetched Jira data. These will be passed to the embedded Product Manager specialist (per `agentTeam.mdc`) as pre-computed metrics, so the specialist can focus on *schedule conflicts, unrealistic durations, and missing buffers* rather than counting.

Compute and store:
- **Project counts:** total non-Done, active, planned, backlog
- **Tickets with missing dates:** count and list (key + summary), broken down by: missing both start and end, missing only end
- **Fetch failures:** count and which tickets could not be retrieved
- **Date range:** earliest project start → latest project end, total span in months
- **Active project summary:** for each active project: milestone count, epic count, % of epics with dates, % of epics Done
- **Scheduling gaps:** any project where end date is before today (overdue), any project with 0 epics
- **Changes since last sync:** which projects/epics had date or status changes (from Step 0 comparison)

Pass these as the `## Pre-Computed Metrics` block when spawning the embedded Product Manager specialist. The specialist then analyzes timeline conflicts, unrealistic durations, and recommends adjustments based on verified facts.

### Step 3a: Build View 3 — Visual Roadmap Table (MANDATORY for Confluence)

Generate a month-column table with `██` block indicators for each project's active months. This is the primary visual on the Confluence page and **MUST always be included** when syncing to Confluence.

1. Calculate month columns: current month through end of roadmap (typically 10–12 months)
2. For each project (active + planned with dates), place `██` in every month where the project's date range overlaps
3. Format: `**[Short Name]** ([KEY]) — [Status]` in the Project column
4. This table goes immediately after the `## 6-Month Roadmap` heading, before the Project Details table

See [templates.md](templates.md) → "View 3: Visual Roadmap Table" for the template and rules.

### Step 3b: Build View 2 — 6-Month Roadmap (Project Granularity)

1. Calculate date range: today through today + 6 months
2. For **active projects**:
   - Prefer `Target start` (`{{TARGET_START_FIELD}}`) for start date, fall back to `created`
   - Prefer `Target end` (`{{TARGET_END_FIELD}}`) or `duedate` for end date
   - If no project-level dates: use min(created) of earliest epic as start, max(duedate) of latest epic as end
   - If still no dates: flag to user
3. For **planned/backlog projects with dates**: include as bars in the Gantt (use `{{TARGET_START_FIELD}}` for start, `duedate` or `{{TARGET_END_FIELD}}` for end)
4. For **planned/backlog projects without dates**: list in an "Unscheduled Projects" table below the Gantt
5. Generate Mermaid Gantt chart at project granularity

See [templates.md](templates.md) for the roadmap Gantt template.

### Step 4: Build View 1 — Detailed Active Projects (Epic/Milestone Granularity)

For each active project, generate:

1. **Mermaid Gantt chart**:
   - One `section` per milestone
   - One task per epic within each milestone
   - Dates from Jira (due date, created date)
   - Status markers derived from Jira status:
     - `Done` / `Merged` → `:done`
     - `In Progress` / `Developing` / `Peer Review` → `:active`
     - `Backlog` / `To Do` → `:crit`
   - If an epic has no due date in Jira, flag it in the summary (Step 5) and use end of current quarter as a placeholder

2. **Ticket Summary Table**:
   - Grouped by milestone
   - Columns: Ticket (linked), Summary, Status, Due, Assignee — all from Jira
   - Include milestones and epics (not individual tasks)

See [templates.md](templates.md) for the detailed Gantt and ticket table templates.

### Step 5: Present Summary and Confirm (MANDATORY)

Show the user:
- Total projects synced (active + backlog)
- Number of tickets queried
- Any tickets with **missing dates** (list them)
- Any tickets that could not be fetched (list them with error)
- Preview of both views

**Wait for user approval before writing the file.**

### Step 6: Write project-timeline.md

1. Overwrite `project-timeline.md` (see [shared/config.md](../shared/config.md) for file path) with the full regenerated content
2. Structure: Legend → 6-Month Roadmap (View 3 Visual Timeline + View 2 Gantt + Project tables) → Active Project Details (View 1)
3. Done projects are NOT included
4. **Confluence sync**: When pushing to Confluence, **ALWAYS** include the View 3 Visual Roadmap Table (month columns with `██` blocks) — this is the primary visual on the Confluence page and must never be omitted

## Output Structure

```markdown
# Project Timeline

Synced from Jira. Last synced: [MMM DD, YYYY HH:MM]

---

## Legend

- **Status markers**: :done = Done/Merged | :active = In Progress/Developing | :crit = Backlog/To Do
- **Source of truth**: All dates, statuses, and assignees come directly from Jira

---

## 6-Month Roadmap

[Mermaid Gantt — project-level bars]

### Unscheduled Projects

[Table of backlog projects without dates]

---

## Active Project Details

### [Project Name] ([KEY]) — [Status]

- **Jira**: [link]
- **Confluence**: [link if exists]

[Mermaid Gantt — epic/milestone level]

[Ticket Summary Table]

---
```

## Critical Rules

- **Jira is the ONLY source of truth** — no dates or statuses from weekly updates or local files
- **Full regeneration** — the file is overwritten each sync (not patched)
- **Done projects are excluded** — they live in `done-projects.md` only
- **NEVER write the file without user approval** in Step 5
- **Collect ALL Jira data before generating output** — do not generate partial views
- **No-op on unchanged data** — if Jira timeline-relevant fields did not change, skip rewriting `project-timeline.md` unless user requests force refresh

## Error Handling

See [shared/config.md](../shared/config.md) for full error handling guidelines. Key scenarios for this skill:

- **Any Jira error**: Follow `atlassianMcpErrors.mdc` rule — display ALL CAPS re-auth message, wait for user to toggle MCP tool, max 1 retry
- **Specific ticket fetch failure**: Note which ticket(s) couldn't be fetched in the Step 5 summary. Proceed with available data — do not abort the entire sync
- **No dates in Jira for a project**: Flag it to the user in Step 5. Ask if they want to assign estimated dates or exclude the project from the Gantt (it can still appear in the Unscheduled table)
- **Jira query returns no projects**: Inform user that no non-Done Project tickets were found. Verify the project key (from shared/config.md) and issue type are correct. Verify the project key and issue type are correct

## Validation Checklist

- [ ] Jira timeline data change check completed first
- [ ] If no timeline changes: file rewrite skipped and user informed
- [ ] Jira queried for all non-Done Project tickets in {{JIRA_PROJECT_KEY}} project
- [ ] All active project tickets fetched from Jira (project + milestones + epics)
- [ ] All backlog project tickets fetched from Jira (project level only)
- [ ] View 3 (Visual Roadmap Table with ██ month blocks) generated — MANDATORY for Confluence
- [ ] View 2 (6-Month Roadmap) generated at project granularity
- [ ] Unscheduled projects listed in table (if any)
- [ ] View 1 (Detailed Active) generated for each active project at epic/milestone granularity
- [ ] Tickets with missing dates flagged to user
- [ ] Failed ticket fetches reported to user
- [ ] Summary presented and user approved before writing
- [ ] project-timeline.md overwritten with regenerated content
- [ ] "Last synced" timestamp included at top of file

See [templates.md](templates.md) for Mermaid Gantt templates and ticket table format.
