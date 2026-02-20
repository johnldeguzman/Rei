---
name: start-week
description: Create weekly project tracking files with priorities and project carryover. Use when the user says "start my week", "start week", or requests weekly tracking initialization.
---

# Start Week

Creates a new weekly tracking file with priorities, carried-over projects, and proper structure.

## Decision Flow

```
┌──────────────────────────┐
│ Step 1: Calculate dates  │
│ (today → Friday)         │
└────────────┬─────────────┘
             │
  ┌──────────▼──────────┐
  │ Step 2: Check prev   │
  │ week file exists?    │
  └──────────┬──────────┘
             │
     ┌───────▼────────┐
     │ "Week Ended"   │
     │ marker found?  │
     └───┬────────┬───┘
         │        │
       Yes        No
         │        │
         ▼        ▼
  ┌──────────┐  ┌────────────────────┐
  │ Show     │  │ Warn user. Ask:    │
  │ next-wk  │  │ • "end prev week"  │
  │ priorities│  │ • "start anyway"   │
  │ if any   │  │ • "cancel"         │
  └────┬─────┘  └──────┬─────────────┘
       │               │
       └───────┬───────┘
               │
    ┌──────────▼──────────┐
    │ File for this week  │── Yes ──▶ Ask: overwrite
    │ already exists?     │           or open existing?
    └──────────┬──────────┘
               │ No
    ┌──────────▼──────────┐
    │ Step 3: Ask for top │
    │ 4 priorities        │
    │ (narrow if > 4)     │
    └──────────┬──────────┘
               │
    ┌──────────▼──────────┐
    │ Step 4: Copy non-   │
    │ Done projects from  │
    │ previous week       │
    │ (verify status via  │
    │ Jira/all-projects)  │
    └──────────┬──────────┘
               │
    ┌──────────▼──────────┐
    │ Step 5: Create new  │
    │ weekly file with    │
    │ structure           │
    └─────────────────────┘
```

## Workflow

### Step 1: Calculate Dates

1. Get today's date in format `MMMDDYYYY` (e.g., `Jan262026`)
2. Calculate Friday's date (end of work week)
3. Generate filename: `[today]-[friday].md`

### Step 2: Check Previous Week Status (MANDATORY)

1. Find most recent weekly file in weekly-updates folder, checking the current half-year subfolder first (e.g., `1h2026` for Jan–Jun 2026, `2h2025` for Jul–Dec 2025). See [shared/config.md](../shared/config.md) for base path
2. Check for "Week Ended" marker at end of file:
   ```markdown
   ---
   **Week Ended:** [Date/Time]
   **Week Status:** Completed
   ---
   ```
3. If marker exists: Check for "Priorities for Next Week" section and display to user
4. If marker does NOT exist: Warn user and ask:
   - "end previous week" - Run end week workflow first
   - "start anyway" - Proceed without ending previous week
   - "cancel" - Stop workflow

### Step 3: Ask About Week Priorities (MANDATORY)

1. Ask: "What are your top priorities for this week? Please list them (I'll help you narrow to 4 if needed):"
2. If user provides more than 4: Ask them to select top 4
3. Confirm final 4 priorities with user

### Step 4: Copy Projects from Previous Week (MANDATORY)

1. Read previous week file and extract project entries
2. Check each project's status in Jira or all-projects.md (see [shared/config.md](../shared/config.md) for status calculation rules)
3. **Skip projects marked "Done" or "Done ✅"**
4. For each active project:
   - Keep project name/identifier with links
   - Add summary: `Last week: [completed]. **This week, [expected] is expected.**`
   - Add empty "Progress this week:" section

### Step 5: Create New Weekly File

Create file in the current half-year subfolder under the weekly-updates folder (e.g., `weekly-updates/1h2026/`). Create the subfolder if it doesn't exist. Half-year convention: `1h` = Jan–Jun, `2h` = Jul–Dec. See [shared/config.md](../shared/config.md) for base path

## File Format Rules

- Start with "Top Priorities for This Week" section (4 checkbox items)
- Include "Adhoc work to do:" section (checkbox items)
- Include all non-done projects from previous week
- End with "Any fires" and "Any highlights" sections
- Add note about strikethrough syntax for completed tasks

## Date Calculation

- **Today**: Current date as `MMMDDYYYY` (no leading zeros)
- **Friday**:
  - Mon-Thu: Friday of current week
  - Friday: Today's date
  - Sat-Sun: Next Friday

## Error Handling

See [shared/config.md](../shared/config.md) for full error handling guidelines. Key scenarios for this skill:

- **Authentication errors (401/403)**: Follow `atlassianMcpErrors.mdc` rule — display ALL CAPS message, wait for user to toggle MCP tool
- **Previous week file not found**: If no weekly files exist in the folder, inform user this appears to be the first week. Skip project carryover and proceed with priorities only
- **all-projects.md not found**: Inform user and proceed without project status verification. Note which projects couldn't be verified
- **Jira status check failure**: Note which project couldn't be verified and proceed. Report unverified projects at the end
- **File already exists**: If a file for this week already exists, ask user: "A file for this week already exists. Do you want to overwrite it or open the existing one?"

## Validation Checklist

- [ ] Previous week checked for "Week Ended" marker
- [ ] User warned if previous week not ended
- [ ] User asked for top 4 priorities
- [ ] All non-done projects copied from previous week
- [ ] Done projects excluded
- [ ] Project status verified via Jira or all-projects.md
- [ ] File created with proper structure
- [ ] Strikethrough note included at bottom

See [templates.md](templates.md) for file format template and examples.
