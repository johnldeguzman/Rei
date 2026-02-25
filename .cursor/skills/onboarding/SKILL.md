---
name: onboarding
description: Create onboarding plans for new team members joining the team. Generates a local draft for review, then publishes to Confluence. Use when the user says "new person joining the team", "onboard someone", "create onboarding plan", or mentions a new hire starting.
---

# Onboarding Plan

Creates an onboarding plan for a new team member joining the team (team name from shared/config.md). Produces a local draft for approval, then publishes to Confluence.

## Workflow

### Step 1: Gather Required Information

Ask the user for:

1. **Name** of the new person
2. **Start date** (format: MM/DD/YYYY)
3. **Buddy** name (the person assigned as their onboarding buddy)

If any of these are missing, ask before proceeding.

### Step 2: Create Local Draft

1. Generate the onboarding plan using the template in [templates.md](templates.md)
2. Replace all placeholders:
   - `[NEW_PERSON_NAME]` with the new hire's full name
   - `[START_DATE]` with their start date
   - `[BUDDY_NAME]` with the buddy's name
3. Save the draft to: `notes/onboarding-[firstname-lowercase].md` (resolve path from shared/config.md)
4. Present the draft to the user and ask: **"Does this draft look good? I can make changes, or if you approve, I'll create the Confluence page."**

### Step 3: Handle User Feedback

- If the user requests changes, update the local draft and present again
- If the user approves (says "looks good", "approved", "yes", "go ahead", "publish", etc.), proceed to Step 4

### Step 4: Publish to Confluence (After Approval Only)

1. Look up the buddy's Atlassian account using `lookupJiraAccountId` with the buddy's name
2. Create a new Confluence page:
   - See [shared/config.md](../shared/config.md) for Cloud ID, Space ID, and Onboarding Parent Page ID
   - **Title**: `Onboarding Plan - [NEW_PERSON_NAME]`
   - **contentFormat**: `markdown`
   - **body**: The approved onboarding plan content
3. Return the Confluence page URL to the user

### Step 5: Confirm Completion

Tell the user:
- The Confluence page has been created
- Provide the link
- Remind them the local draft is still at `notes/onboarding-[firstname].md` for reference

## Error Handling

See [shared/config.md](../shared/config.md) for full error handling guidelines. Key scenarios for this skill:

- **Authentication errors (401/403)**: Follow `atlassianMcpErrors.mdc` rule — display ALL CAPS message, wait for user to toggle MCP tool
- **Buddy lookup failure**: If `lookupJiraAccountId` fails or returns no results, ask user for the buddy's Atlassian email or account ID. Proceed with page creation regardless — the @mention can be fixed later
- **Confluence page creation failure**: Report the error. The local draft is preserved — ask user if they want to retry or create the page manually using the draft
- **Draft save failure**: If the local file can't be saved, present the draft content in chat so user can copy it. Then proceed to Confluence publish if approved

## Validation Checklist

- [ ] Name, start date, and buddy collected from user
- [ ] Local draft created at `notes/onboarding-[firstname].md`
- [ ] All placeholders replaced (no `[NEW_PERSON_NAME]`, `[START_DATE]`, `[BUDDY_NAME]` remaining)
- [ ] Draft presented to user for review
- [ ] User approved draft before Confluence publish
- [ ] Buddy's Atlassian account looked up
- [ ] Confluence page created under correct parent page
- [ ] Link provided to user

See [templates.md](templates.md) for the full onboarding plan template.
