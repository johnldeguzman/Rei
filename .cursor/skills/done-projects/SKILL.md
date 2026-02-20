---
name: done-projects
description: Manage completed projects by moving them from all-projects.md to done-projects.md with completion date and contributors. Use when a project status changes to "Done" or when reviewing/updating all-projects.md.
---

# done-projects Management

Moves completed projects from `all-projects.md` to the dedicated `done-projects.md` file with completion date and contributor information.

## Workflow

### Step 1: Identify done-projects

1. Read `all-projects.md` (see [shared/config.md](../shared/config.md) for file path)
2. Find projects with status "Done ✅"
3. Cross-reference with recent weekly updates

### Step 2: Gather Contributor Information (MANDATORY)

1. Check Jira tickets for assignee information
2. Check weekly updates and project notes
3. **If missing**: Ask user: "Who worked on [Project Name]? Please provide contributor names."
4. Wait for response before proceeding

### Step 3: Move to done-projects File

1. Read `done-projects.md` (see [shared/config.md](../shared/config.md) for file path)
2. Add done project using the entry format in [templates.md](templates.md)
3. Add projects in reverse chronological order (most recent first)

### Step 4: Update Active Projects Section

1. Remove done projects from `all-projects.md` active section
2. Maintain proper formatting and separators

### Step 5: Sync to Confluence (MANDATORY)

1. Update `done-projects` Confluence page with `done-projects.md` content
2. Update All Projects Confluence page (see shared/config.md for page ID) with updated `all-projects.md` content
3. See [shared/config.md](../shared/config.md) for all Confluence page IDs and Cloud ID
4. Confirm successful sync

## Critical Rules

- **NEVER** move project without completion date
- **NEVER** move project without contributor information
- **ALWAYS** ask user for contributors if not clear
- **ALWAYS** sync to both Confluence pages after moving
- **ALWAYS** move projects to `done-projects.md` file, NOT to a section in `all-projects.md`

## Error Handling

See [shared/config.md](../shared/config.md) for full error handling guidelines. Key scenarios for this skill:

- **Authentication errors (401/403)**: Follow `atlassianMcpErrors.mdc` rule — display ALL CAPS message, wait for user to toggle MCP tool
- **File not found**: If `all-projects.md` or `done-projects.md` is missing, inform user of the expected path and ask how to proceed
- **Confluence sync failure**: Inform user which page failed. Retry once after 10 seconds for 500/timeout. If still failing, confirm page ID is still valid
- **Partial sync**: If one Confluence page syncs but the other fails, tell the user which succeeded and which failed — do not silently skip

## Validation Checklist

- [ ] Done projects identified
- [ ] Contributor info gathered (asked user if missing)
- [ ] Projects moved to `done-projects.md` file
- [ ] Projects added with completion date and contributors
- [ ] Ticket links preserved
- [ ] Final status summary included
- [ ] Active section cleaned up in `all-projects.md`
- [ ] done-projects Confluence page synced
- [ ] All Projects Confluence page synced (page ID from shared/config.md)

See [templates.md](templates.md) for entry format and examples.
