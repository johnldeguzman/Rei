# Shared Configuration

Central reference for values, formats, and patterns used across multiple skills. **All skills must reference this file instead of hardcoding values.**

Values marked with `{{PLACEHOLDER}}` must be set during setup. Run `"set me up"` to populate them.

---

## User Configuration

These values are unique to each user/org. The setup skill populates them.

| Parameter | Value | Description |
|-----------|-------|-------------|
| User Name | `{{USER_NAME}}` | Your name (used in SOUL.md and templates) |
| User Role | `{{USER_ROLE}}` | Your role and team |
| Team Name | `{{TEAM_NAME}}` | Your team name (used in skill descriptions and JQL context) |

## Atlassian Configuration

Optional — skip if not using Jira/Confluence.

| Parameter | Value | Description |
|-----------|-------|-------------|
| Atlassian Site | `{{ATLASSIAN_SITE}}` | Your Atlassian domain (e.g., `mycompany.atlassian.net`) |
| Cloud ID | `{{CLOUD_ID}}` | Atlassian Cloud ID |
| Jira Project Key | `{{JIRA_PROJECT_KEY}}` | Jira project key used in JQL (e.g., `ID`, `ENG`, `PLAT`) |
| Target Start Field | `{{TARGET_START_FIELD}}` | Custom field ID for target start date (e.g., `customfield_11227`) |
| Target End Field | `{{TARGET_END_FIELD}}` | Custom field ID for target end date (e.g., `customfield_11228`) |

### Confluence Page IDs

| Page | Page ID | Description |
|------|---------|-------------|
| All Projects Page | `{{ALL_PROJECTS_PAGE_ID}}` | Confluence page that mirrors all-projects.md |
| Done Projects Page | `{{DONE_PROJECTS_PAGE_ID}}` | Confluence page for completed projects |
| Project Timeline Page | `{{TIMELINE_PAGE_ID}}` | Confluence page that mirrors project-timeline.md |
| Space ID | `{{CONFLUENCE_SPACE_ID}}` | Confluence space ID |
| Onboarding Parent Page | `{{ONBOARDING_PARENT_PAGE_ID}}` | Parent page for onboarding docs |
| Technical Discovery Parent | `{{TECH_DISCOVERY_PARENT_ID}}` | Parent page for technical discovery docs |
| Active Projects Parent | `{{ACTIVE_PROJECTS_PARENT_ID}}` | Parent page for active project docs |

### Jira Issue Type IDs

| Issue Type | ID | Hierarchy Level |
|-----------|-----|----------------|
| Project | `{{ISSUE_TYPE_PROJECT}}` | 3 |
| Discovery Milestone | `{{ISSUE_TYPE_DISCOVERY}}` | 2 |
| Delivery Milestone | `{{ISSUE_TYPE_DELIVERY}}` | 2 |
| Rollout Milestone | `{{ISSUE_TYPE_ROLLOUT}}` | 2 |
| Epic | `{{ISSUE_TYPE_EPIC}}` | 1 |
| Technical Task | `{{ISSUE_TYPE_TASK}}` | 0 |

---

## File Locations

All paths are **relative to workspace root**. Skills resolve these against the Cursor workspace directory at runtime.

| File | Relative Path |
|------|--------------|
| all-projects.md | `all-projects.md` |
| done-projects.md | `done-projects.md` |
| project-timeline.md | `project-timeline.md` |
| weekly-updates | `weekly-updates/` (organized by half-year subfolders: `1h2026`, `2h2025`, etc.) |
| performance-reviews | `performance-reviews/` |
| notes | `notes/` |

---

## Date Formats

| Context | Format | Example |
|---------|--------|---------|
| Weekly file names | `MMMDDYYYY` (no leading zeros) | `Jan262026-Jan302026.md` |
| Week ended marker | `MMM DD, YYYY HH:MM` | `Jan 23, 2026 17:00` |
| Done project completion date | `MMM DD, YYYY` | `Jan 15, 2026` |
| Onboarding start date | `MM/DD/YYYY` | `02/03/2026` |
| Weekly executive summary headers | `[Mon] - [Fri]` (short month + day) | `Jan 19 - Jan 24` |
| Performance review period | `H1 YYYY` or `H2 YYYY` | `H1 2026` |

---

## Jira Queries

### Primary project query

```
project = {{JIRA_PROJECT_KEY}} AND issuetype = Project AND statusCategory != Done ORDER BY status ASC, created DESC
```

This query returns all active, planned, and backlog projects from Jira. **Jira is the source of truth** for the project list.

### Jira Query Fields

| Field | Jira API Field Name | Used For |
|-------|-------------------|----------|
| Summary | `summary` | Ticket/project names |
| Status | `status` | Gantt markers, table status |
| Target Start | `{{TARGET_START_FIELD}}` | Gantt start dates (preferred over `created`) |
| Target End | `{{TARGET_END_FIELD}}` | Gantt end dates (use alongside `duedate`) |
| Assignee | `assignee` | Ticket table assignee column |
| Due Date | `duedate` | Gantt end dates |
| Created | `created` | Gantt start date fallback |
| Resolution Date | `resolutiondate` | Done date for completed items |

### Gantt Status Mapping

| Jira Status | Gantt Marker |
|-------------|-------------|
| `Done`, `Merged` | `:done` |
| `In Progress`, `Developing`, `Peer Review` | `:active` |
| `Backlog`, `To Do` | `:crit` |

---

## Status Calculation Rules

1. **If project ticket exists**: Use status directly from Jira
2. **If no project ticket**: Calculate from epic statuses:
   - "Done" ONLY if ALL epics are "Done"
   - "In Progress" if ANY epic is "In Progress" or "Backlog"
3. **Status progression**: Backlog → In Progress → Done

---

## all-projects.md Entry Format

```markdown
## [Project Title]

**Confluence URL:** [Confluence page URL]
[ONLY include this line if a Confluence URL actually exists]

**Jira Project Ticket:** [[PROJECT-KEY] - [Project Name]](https://{{ATLASSIAN_SITE}}/browse/[PROJECT-KEY])

**Status:** [Status]

**Jira Epic Tickets:**
- [[EPIC-KEY-1] - [Epic Name]](https://{{ATLASSIAN_SITE}}/browse/[EPIC-KEY-1]) (Status: [Status])
- [[EPIC-KEY-2] - [Epic Name]](https://{{ATLASSIAN_SITE}}/browse/[EPIC-KEY-2]) (Status: [Status])

**Week of [Monday] - [Friday]:**
- **Status:** [Executive summary - project update/status only]

---
```

---

## Error Handling

All skills that call Jira or Confluence APIs must follow these guidelines:

### Authentication Errors (401, 403)
- Follow the `atlassianMcpErrors.mdc` rule: display the re-authentication message and wait for user to toggle the MCP tool and confirm
- **DO NOT** automatically retry — wait for user confirmation

### File Not Found
- If a required file does not exist:
  - Tell the user which file is missing and its expected path
  - Ask if they want to create it or provide an alternative path
  - **DO NOT** silently skip the step

### API Failures (404, 500, Timeout, Rate Limit)
- Tell the user which API call failed and the error
- For 404: Confirm the resource ID/key is correct with the user
- For 500/timeout: Wait 10 seconds and retry once. If still fails, inform the user
- For rate limit (429): Wait 30 seconds and retry once. If still fails, inform the user

### CLI Fallback (Before Manual Checklist)
- When MCP fails after re-auth retry, **attempt the operation via CLI** before falling back to a manual checklist
- Fallback chain: `acli` (auto-install if missing on macOS/Linux) → `curl` to Atlassian REST API → manual checklist
- Auth: API token + email via env vars (`ATLASSIAN_API_TOKEN`, `ATLASSIAN_EMAIL`), `~/.atlassian-credentials`, or user-provided
- Jira base: `https://{{ATLASSIAN_SITE}}/rest/api/3` | Confluence base: `https://{{ATLASSIAN_SITE}}/wiki/api/v2`
- **Important**: Use `/rest/api/3/search/jql` for JQL search (legacy `/rest/api/3/search` has been removed)
- See `atlassianMcpErrors.mdc` Step 3 for full CLI operation mapping and auth details

### Graceful Degradation (Write Failures)
- When Jira/Confluence **write** operations fail via both MCP and CLI, **do not discard the intended changes**
- Capture each failed write (target, operation, payload, reason) and accumulate them
- Present a single **Manual Follow-Up Checklist** at the end of the workflow
- **Local file writes are unaffected** — always write to local files even if Jira/Confluence is down
- See `atlassianMcpErrors.mdc` Step 4 for full details and checklist format
