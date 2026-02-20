---
name: end-week
description: End the week by creating executive summaries, syncing with all-projects.md, verifying Jira statuses, and optionally syncing to Confluence. Use when the user says "end my week", "end week", or requests weekly summary and sync.
---

# End Week

Wraps up the week by summarizing project updates, creating executive summaries, and syncing with project tracking.

## Decision Flow

```
┌─────────────────────────────────┐
│  Step 1: Find current week file │
└──────────────┬──────────────────┘
               │
    ┌──────────▼──────────┐
    │ Step 2: Parse weekly │
    │ updates + reconcile  │
    │ linked sources       │
    └──────────┬──────────┘
               │
    ┌──────────▼──────────┐
    │ Any projects missing │── No ──▶ Continue
    │ updates?             │
    └──────────┬──────────┘
               │ Yes
    ┌──────────▼──────────┐
    │ Ask user for updates │
    └──────────┬──────────┘
               │
    ┌──────────▼──────────┐
    │ Step 3: Create exec  │
    │ summaries (fires +   │
    │ project summaries)   │
    └──────────┬──────────┘
               │
    ┌──────────▼──────────┐
    │ Step 4: Next week    │── No ──▶ Skip
    │ priorities? (ask)    │
    └──────────┬──────────┘
               │ Yes
    ┌──────────▼──────────┐
    │ Collect priorities   │
    │ from user (explicit) │
    └──────────┬──────────┘
               │
    ┌──────────▼──────────┐
    │ Step 5: Sync         │
    │ all-projects.md      │
    │ (status only)        │
    └──────────┬──────────┘
               │
    ┌──────────▼──────────┐
    │ Step 6: Verify Jira  │── Fail ──▶ Note unverified,
    │ ticket statuses      │            continue
    └──────────┬──────────┘
               │
    ┌──────────▼──────────┐
    │ Step 7: Sync to      │── No ──▶ Skip
    │ Confluence? (ask)    │
    └──────────┬──────────┘
               │ Yes
    ┌──────────▼──────────┐
    │ Push to Confluence   │── Fail ──▶ Do NOT mark ended,
    └──────────┬──────────┘            ask skip or retry
               │
    ┌──────────▼──────────┐
    │ Step 7.5: Sync       │── No ──▶ Skip
    │ timeline? (ask)      │
    └──────────┬──────────┘
               │ Yes
    ┌──────────▼──────────┐
    │ Run timeline-sync    │
    └──────────┬──────────┘
               │
    ┌──────────▼──────────┐
    │ Step 8: Mark week    │
    │ as ended (LAST)      │
    └─────────────────────┘
```

## Workflow

### Step 1: Identify Current Week File

1. Get today's date in format `MMMDDYYYY`
2. Locate weekly file in the weekly-updates folder (see [shared/config.md](../shared/config.md) for path)

### Step 2: Read and Parse Weekly Updates

1. Read the weekly file
2. Extract project entries, progress notes, fires, accomplishments
3. **If the user provides linked sources (Confluence/Jira/docs), fetch and reconcile them (MANDATORY)**:
   - Fetch each linked source directly
   - Extract concrete project-relevant updates (not just link/title references)
   - Compare against current weekly bullets
   - Add missing concrete updates into the appropriate project "Progress this week:" section
   - If nothing new is found, explicitly state that no additional concrete updates were found
4. **Check for projects without updates (MANDATORY)**:
   - Flag projects with empty "Progress this week:" sections
   - Ask user: "I noticed these projects don't have updates: [list]. Do you have any updates? (yes/no)"
   - If yes: Collect updates and add to file

### Step 3: Create Executive Summaries

1. **Create Fires Summary (at top)**:
   ```markdown
   ### Fires - Week of [Monday] - [Friday]
   
   **[Fire Name] ([Date])** - [Ticket]: [Description]. **Resolution:** [Status].
   ```

2. **Create Project Summaries**:
   ```markdown
   ### [Project Name] - Week of [Monday] - [Friday]
   
   **Current Status:** [Summary]
   
   **Accomplishments:**
   - [List - ONLY if content exists in weekly update]
   ```

3. Add executive summaries to weekly file under "## Weekly Executive Summary"

### Step 4: Ask About Next Week Priorities

1. Ask: "Do you want to add priorities for next week? (yes/no)"
2. If yes: Ask the user explicitly for the exact priorities they want added (freeform list or numbered items)
3. Do not infer, auto-rollover, or reuse unchecked items from this week unless the user explicitly asks for rollover
4. Add the user-provided priorities under "## Priorities for Next Week"

### Step 5: Sync with all-projects.md

1. Read `all-projects.md` (see [shared/config.md](../shared/config.md) for path and entry format)
2. For each project, update with:
   - Week date range
   - Status/project update (technical details)
   - **DO NOT sync fires or accomplishments**

### Step 6: Verify Jira Ticket Statuses

1. For each project, verify Jira status
2. Update epic statuses
3. Calculate project status using the rules in [shared/config.md](../shared/config.md)

### Step 7: Sync to Confluence (Optional)

1. Ask: "Do you want to sync all-projects.md to Confluence? (yes/no)"
2. If yes: Update the All Projects Confluence page (see [shared/config.md](../shared/config.md) for page ID and Cloud ID)

### Step 7.5: Sync Project Timeline (Optional)

1. Ask: "Do you want to sync the Project Timeline? (yes/no)"
2. If yes: Run the timeline-sync skill workflow (see [timeline-sync/SKILL.md](../timeline-sync/SKILL.md))
3. If no: Skip and proceed to mark week as ended

### Step 8: Mark Week as Ended (MANDATORY - LAST STEP)

**Only after ALL steps complete**, add:
```markdown
---
**Week Ended:** [Date/Time in format: MMM DD, YYYY HH:MM]
**Week Status:** Completed
---
```

## Critical Rules

- **Fires** are summarized separately at the top — NOT in project summaries
- Only include "Accomplishments" if content exists in weekly update
- Only sync status/project updates to all-projects.md (no fires/accomplishments)
- Week marked as ended ONLY after all steps complete
- **Priorities must be explicit**: never assume next-week priorities from current-week tasks; ask the user and use only what they provide
- **Linked source reconciliation is required**: when user provides links, include concrete extracted updates, not only a citation/reference line

## Error Handling

See [shared/config.md](../shared/config.md) for full error handling guidelines. Key scenarios for this skill:

- **Authentication errors (401/403)**: Follow `atlassianMcpErrors.mdc` rule — display ALL CAPS message, wait for user to toggle MCP tool
- **Weekly file not found**: If the current week file doesn't exist, tell the user and suggest running "start week" first
- **Empty weekly file**: If the file exists but has no project entries, warn the user before proceeding with empty summaries
- **Jira status check failure**: If a Jira call fails, note which project couldn't be verified and proceed with the rest. Report unverified projects at the end
- **Confluence sync failure**: Inform user which page failed. Retry once after 10 seconds. Do not mark week as ended if sync was requested but failed — ask user if they want to skip sync or retry

## Validation Checklist

- [ ] Weekly file read and parsed
- [ ] Linked sources fetched and concrete updates reconciled (if user provided links)
- [ ] Projects without updates identified and user asked
- [ ] Fires executive summary created (if fires exist)
- [ ] Project executive summaries created
- [ ] User asked about next week priorities
- [ ] If priorities are added, user explicitly provided the priority list (no inferred rollover)
- [ ] all-projects.md synced (status only)
- [ ] Jira statuses verified
- [ ] User asked about Confluence sync
- [ ] User asked about Project Timeline sync
- [ ] Week marked as ended (LAST step)

See [templates.md](templates.md) for executive summary format and examples.
