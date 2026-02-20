---
name: jira-project
description: Create Jira projects with visual planning, milestone selection, Confluence pages, and all-projects.md tracking. Use when the user says "create a project" or requests Jira project creation.
---

# Jira Project Creation

Creates Jira tickets using a standardized hierarchy with visual planning and approval workflow.

## Hierarchy Structure

```
Project (Level 3)
  └── Milestone (Level 2: Discovery/Delivery/Rollout)
      └── Epic (Level 1)
          └── Technical Task (Level 0)
```

## Workflow

### Step 0: Project ID & Milestone Selection (MANDATORY)

1. **Ask for Project ID**: "What is the Project ID where this should be created?"
2. **Ask for Milestones**: "Do you need Discovery, Delivery, and Rollout milestones? (yes/no)"
   - If no: Ask which specific milestones to create

### Step 1: Visual Planning (MANDATORY - DO NOT SKIP)

1. Create ASCII tree diagram of complete structure
2. Present to user
3. **Wait for "create" approval before proceeding**

### Step 2: Create Tickets After Approval

1. Create Project ticket
2. Create Milestone tickets (only approved ones)
3. Create Epic tickets (if Discovery Milestone exists, include "Discovery" epic)
4. Create Technical Task tickets
5. Fill in description fields with context

### Step 3: Confluence Page (MANDATORY)

1. Ask: "Do you need a Confluence page? (yes/no)"
2. If yes, ask: "Which section: 'Technical Discovery', 'Active', or 'Done'?"
3. Create page with JQL query for finding epics
4. See [shared/config.md](../shared/config.md) for Confluence parent page IDs and Cloud ID

### Step 4: Update all-projects.md (MANDATORY)

1. Add project to `all-projects.md` (see [shared/config.md](../shared/config.md) for file path and entry format)
2. Include: Title, Confluence URL (if exists), Jira Project Ticket link, Status, Epic tickets with links

### Step 5: Overlap Detection & Approval (MANDATORY - DO NOT SKIP)

Before updating project-timeline.md, check for scheduling conflicts:

1. **Read `project-timeline.md`** and compare the new project's start/end dates against ALL existing projects in the Gantt chart
2. **If overlaps exist**, present a dedicated **Overlap Analysis** section:

   a. **ASCII timeline visualization** showing how projects overlap:
   ```
   Apr       May       Jun       Jul
   |---------|---------|---------|---------|
    Project Beta (PRJ-1002)
    ████████████████████
    Apr 1 ──── May 29
              NEW PROJECT (ID-XXXX)
              ████████████████████
              May 1 ──── Jun 30

    Overlap zones:
    ══════════╗
              ║ Project Beta + NEW: May 1 – May 29 (29 days)
              ╚══════════
   ```

   b. **Overlap summary table**:
   | Overlap | Projects | Period | Duration |
   |---------|----------|--------|----------|

   c. **Options** — suggest concrete alternatives:
      - Keep as-is (accept overlap)
      - Shift the new project to avoid overlap
      - Shift existing projects forward (flag cascade effects)
      - Compress/extend durations

3. **Explicitly ask**: "Do you want to proceed with the overlap, or adjust dates?"
4. **Wait for user decision** before updating any files or Jira tickets
5. If the user adjusts dates, **cascade check**: show the ripple effect on downstream projects and confirm again
6. After approval, update Jira target start/end dates for any shifted projects

### Step 6: Update project-timeline.md (MANDATORY)

1. Add project to `project-timeline.md` (see [shared/config.md](../shared/config.md) for file path)
2. Add to the **Gantt chart** in the appropriate section (`Active` if in progress, `Planned` if upcoming) with the correct date range and `:active` or `:crit` marker
3. Add to the **Project Owners table** with Ticket ID, Start/End dates, and Team (use "TBD" if no assignee yet)
4. If the project has target start/end dates, use those. If not, flag to the user that dates are needed for the timeline
5. If any projects were shifted during overlap resolution, update their dates in the Gantt chart, Project Owners table, AND Jira

## Visual Diagram Format

```
Project: [Project Name] ([ABBREVIATION])

├── [Milestone Name] ([Milestone Type])
│   ├── Epic: [Epic Name]
│   │   ├── Technical Task: [Task Description]
│   │   └── Technical Task: [Task Description]
│   └── Epic: [Epic Name]
│       └── Technical Task: [Task Description]
│
└── [Milestone Name] ([Milestone Type])
    └── Epic: [Epic Name]
        └── Technical Task: [Task Description]
```

## Naming Conventions

- **Project**: `[Name] ([ABBREVIATION])` → `Wiz Platform Evaluation (WIZPLAT)`
- **Milestone**: `[Phase Name] ([Type])` → `Discovery Phase (Discovery Milestone)`
- **Epic**: Descriptive theme → `Current State & Requirements Analysis`
- **Task**: `[Verb] [Subject] [Context]` → `Review Wiz UVM documentation for scanner integration`

## Issue Type IDs

See [shared/config.md](../shared/config.md) for PRD project Issue Type IDs and hierarchy levels.

## Critical Rules

- **NEVER** create tickets without Project ID
- **NEVER** create tickets without visual diagram approval
- **ALWAYS** wait for "create" command
- If Discovery Milestone exists, include at least one "Discovery" epic

## Error Handling

See [shared/config.md](../shared/config.md) for full error handling guidelines. Key scenarios for this skill:

- **Authentication errors (401/403)**: Follow `atlassianMcpErrors.mdc` rule — display ALL CAPS message, wait for user to toggle MCP tool
- **Jira ticket creation failure**: If any ticket fails to create, stop and report which tickets were created and which failed. Ask user if they want to retry the failed ones or roll back
- **Confluence page creation failure**: Report the error. The Jira tickets are still valid — ask if user wants to retry Confluence or skip it
- **all-projects.md not found**: Inform user of expected path. Ask if they want to create it or skip this step
- **Partial ticket creation**: If the process fails mid-way (e.g., Project created but Milestones fail), report what exists and what's missing so user can decide next steps

## Validation Checklist

- [ ] Project ID confirmed with user
- [ ] Milestones selected (Discovery/Delivery/Rollout)
- [ ] Visual diagram presented and approved by user
- [ ] All tickets created (Project, Milestones, Epics, Tasks)
- [ ] Descriptions filled in with context
- [ ] Confluence page created (if requested)
- [ ] all-projects.md updated with new project entry
- [ ] Overlap analysis presented with visualization (if overlaps exist)
- [ ] User explicitly approved or resolved overlaps before proceeding
- [ ] project-timeline.md updated (Gantt chart + Project Owners table)
- [ ] Shifted projects updated in Jira (if any dates changed)
- [ ] All Jira links verified as working

See [templates.md](templates.md) for full examples, all-projects.md format, and action verb reference.
