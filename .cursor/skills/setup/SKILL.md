---
name: setup
description: Interactive setup wizard that configures Rei for a new user. Prompts for identity, team info, and optional Atlassian configuration, then writes SOUL.md, shared/config.md, and scaffolds the workspace.
---

# Setup Wizard

Interactive first-run setup for new Rei users. Walks through identity, team, Atlassian configuration (optional), and workspace scaffolding in a single guided flow.

## Template Source

All starter files live in `template/` at the workspace root. The setup skill reads templates from there and writes personalized versions to their working locations. See `template/README.md` for the full manifest.

## When This Activates

- User says "set me up", "setup", "configure Rei", "get started", or "initialize"
- First-run detection: SOUL.md still contains `{{USER_NAME}}` placeholders

## Decision Flow

```
START
  │
  ├─ Detect: Does SOUL.md have {{USER_NAME}} placeholders?
  │   ├─ Yes → Full setup flow
  │   └─ No → Ask: "Rei is already configured for [name]. Run setup again? This will overwrite your current config."
  │       ├─ Yes → Full setup flow
  │       └─ No → Exit
  │
  ├─ Step 1: Identity (MANDATORY)
  │   └─ Prompt for name, role, team, communication style
  │
  ├─ Step 2: Atlassian (OPTIONAL)
  │   ├─ "Do you use Jira and Confluence?"
  │   │   ├─ Yes → Prompt for site URL and project key
  │   │   │   ├─ Attempt MCP auto-discovery
  │   │   │   │   ├─ MCP available → Auto-fetch issue types, fields, Confluence spaces
  │   │   │   │   └─ MCP unavailable → Ask user or leave as placeholders
  │   │   │   └─ Show what was discovered, confirm with user
  │   │   └─ No → Skip, mark Atlassian as unconfigured
  │   │
  ├─ Step 3: Write Config Files
  │   └─ Write SOUL.md, shared/config.md with collected values
  │
  ├─ Step 4: Scaffold Workspace
  │   └─ Create folders and starter files that don't exist
  │
  ├─ Step 5: Summary + Next Steps
  │   └─ Show what was configured, what was skipped, what to do next
  │
  END
```

## Workflow

### Step 1: Identity (MANDATORY)

Prompt the user for their identity. Present all questions together — don't drip-feed one at a time.

**Collect these values:**

| Value | Prompt | Required | Default |
|-------|--------|----------|---------|
| Name | "What's your name?" | Yes | — |
| Role | "What's your role and team? (e.g., Engineering Manager, Platform)" | Yes | — |
| Team Name | "What's your team name? (used in skill descriptions and reports)" | Yes | — |
| Communication Style | "How do you prefer to communicate? (e.g., direct, detailed, async-first)" | No | "Direct, efficient" |

### Step 2: Atlassian Configuration (OPTIONAL)

Ask: **"Do you use Jira and Confluence? (Rei works without them — weekly tracking, task management, and agent team reviews are all local-only.)"**

If **no**: skip to Step 3. Mark Atlassian as unconfigured. Skills that require Jira/Confluence will note this at runtime.

If **yes**:

#### 2a: Collect minimum required values (always ask)

| Value | Prompt | Example |
|-------|--------|---------|
| Atlassian Site | "What's your Atlassian site URL? (e.g., mycompany.atlassian.net)" | `mycompany.atlassian.net` |
| Jira Project Key | "What's your Jira project key? (the prefix on your tickets, e.g., ENG, PLAT, ID)" | `ENG` |

#### 2b: Auto-discover via MCP (if available)

Once you have the site URL and project key, attempt to discover the remaining config automatically using the Atlassian MCP tools. **Tell the user:** "Let me try to auto-detect your Jira and Confluence settings..."

**Discovery sequence** — run these in order, skip any that fail:

| What to discover | MCP call | How to extract |
|-----------------|----------|---------------|
| **Issue type IDs** | `searchJiraIssuesUsingJql` with JQL: `project = {KEY} ORDER BY created DESC` (maxResults=1), then inspect the returned issue's `issuetype` field. Also try: `getJiraIssue` on a known ticket to see available fields. | Map issue type names (Epic, Task, etc.) to their IDs from the response schema. |
| **Custom field IDs** (Target Start/End) | `getJiraIssue` on any ticket in the project — request all fields. Look for fields with names containing "Target start", "Target end", or "Start date", "End date". | Custom fields appear as `customfield_NNNNN` in the response. Match by display name. |
| **Confluence spaces** | `searchConfluencePages` or `getConfluenceSpacesByName` — search for spaces the user has access to. | Present the list and ask the user to pick their team's space. |
| **Confluence pages** | Once space is identified, `getConfluencePageDescendants` on the space root, or search for pages named "All Projects", "Done Projects", "Project Timeline". | Match by page title. If found, auto-fill the page IDs. |
| **Cloud ID** | Often available from MCP connection metadata or from any successful API response headers. | Extract if available, otherwise leave as placeholder. |

**Important rules for auto-discovery:**
- **Never silently fail.** If a discovery call fails (auth error, timeout, 404), note what couldn't be discovered and move on — don't block setup.
- **Don't trigger the re-auth flow.** If MCP auth fails during setup, just say: "Couldn't connect to Atlassian — I'll leave those fields as placeholders. You can fill them later or run setup again."
- **Always confirm with the user.** After discovery, show what was found and ask: "Does this look right?" before writing to config. Example:

```
I found the following from your Jira/Confluence instance:

**Issue Types:**
- Epic: 10004
- Task: 10931
- [etc.]

**Custom Fields:**
- Target Start: customfield_11227
- Target End: customfield_11228

**Confluence:**
- Space: "Engineering" (ID: 1234567)
- Found page "All Projects" (ID: 9876543)
- Found page "Project Timeline" (ID: 8765432)

Does this look right? (I'll write these to your config.)
```

#### 2c: Fill gaps manually (for anything MCP couldn't find)

For any values that auto-discovery didn't resolve, ask the user — but only for values they're likely to know:

| Value | Ask if not discovered | Fallback |
|-------|----------------------|----------|
| Target Start Field | "I couldn't auto-detect your 'Target Start' custom field. Do you know the field ID? (Leave blank to skip)" | Leave as `{{TARGET_START_FIELD}}` |
| Target End Field | Same | Leave as `{{TARGET_END_FIELD}}` |
| Confluence page IDs | "I couldn't find pages named 'All Projects' or 'Done Projects'. Do you have page IDs for these? (Leave blank — you can set them up later)" | Leave as placeholders |

**Don't ask for:** Cloud ID, issue type IDs, or space IDs if MCP couldn't find them — these are too obscure for most users. Leave as placeholders and note them in the summary.

#### 2d: If MCP is not available at all

If no MCP tools are available (not configured, not enabled), fall back to a minimal manual flow:

| Value | Prompt | Fallback |
|-------|--------|----------|
| Target Start Field | "Custom field ID for 'Target Start' date? (Check Jira admin → Issues → Custom fields, or leave blank)" | Leave as `{{TARGET_START_FIELD}}` |
| Target End Field | Same | Leave as `{{TARGET_END_FIELD}}` |

Skip all Confluence and issue type ID questions. Tell the user: "Atlassian MCP isn't connected right now, so I can't auto-detect your settings. I've saved your site URL and project key — you can run setup again after enabling MCP to auto-fill the rest."

### Step 3: Write Config Files

#### 3a: Write SOUL.md

Read the template from `template/SOUL.md`. Replace:
- `{{USER_NAME}}` → collected name
- `{{USER_ROLE}}` → collected role
- Communication style line → collected style (or default)

Do NOT change the personality sections, self-improvement loop, or behavioral rules — those are Rei's core identity.

#### 3b: Write shared/config.md

Read the template from `template/config.md`. Replace all `{{PLACEHOLDER}}` values with collected values. Write the result to `.cursor/skills/shared/config.md`. For any values the user skipped, **leave the placeholder as-is** — this makes it easy to find and fill later.

### Step 4: Scaffold Workspace

Create these files/folders only if they don't already exist:

| Path | What to create |
|------|---------------|
| `all-projects.md` | Empty project tracker with header |
| `done-projects.md` | Empty done projects file with header |
| `project-timeline.md` | Empty timeline file with header |
| `weekly-updates/` | Directory |
| `notes/` | Directory |
| `project-ideas/` | Directory |
| `performance-reviews/` | Directory |
| `IMPROVEMENTS.md` | Only if missing — use the standard template |

#### Starter file templates

**all-projects.md** (if new):
```markdown
# All Projects

*Last synced: [not yet synced]*

---

<!-- Add projects here using "create a project" or manually in the format documented in shared/config.md -->
```

**done-projects.md** (if new):
```markdown
# Done Projects

Completed projects with contributors and completion dates.

---

<!-- Projects move here automatically when marked Done via the done-projects skill -->
```

**project-timeline.md** (if new):
```markdown
# Project Timeline

*Last synced from Jira: [not yet synced]*

---

<!-- Run "sync timeline" to populate from Jira -->
```

### Step 5: Summary + Next Steps

Present a clear summary of what was configured:

```
## Setup Complete

**Identity:**
- Name: [name]
- Role: [role]
- Team: [team]

**Atlassian:** [Configured / Not configured]
[If configured: Site: [site], Project Key: [key]]
[If partially configured: list what was filled vs. still placeholder]

**Workspace files created:**
- [list any new files/folders created]

**What's ready:**
- ✅ Weekly tracking ("start my week", "end my week")
- ✅ Task management ("mark X as done")
- ✅ Agent team reviews (share a PRD or design doc)
- ✅ Self-improvement loop (IMPROVEMENTS.md)
[If Atlassian configured:]
- ✅ Jira project creation ("create a project")
- ✅ Timeline sync ("sync timeline")
- ✅ Jira health checks ("jira health check")

**Still needs setup (if any):**
- [ ] [List any {{PLACEHOLDER}} values still remaining]
- [ ] [E.g., "Confluence page IDs — needed for Confluence sync"]
- [ ] [E.g., "Jira issue type IDs — needed for project creation"]

**Try it out:** Say "start my week" to create your first weekly tracking file.
```

## Critical Rules

- **Never skip the identity step.** Name and role are required.
- **Atlassian is always optional.** Never make the user feel like Jira/Confluence is required. Core workflows work without it.
- **MCP-first for Atlassian config.** If MCP is available, always attempt auto-discovery before asking the user for field IDs. Most EMs won't know their custom field IDs or issue type IDs — infer them.
- **Don't block setup on MCP failures.** If MCP calls fail, leave values as placeholders and move on. Setup must always complete.
- **Confirm discovered values.** Always show the user what was auto-detected and get a "looks right" before writing config.
- **Don't flood the user with questions.** Group related questions together. Only ask for values MCP couldn't discover and the user is likely to know.
- **Preserve what exists.** If a file already has content (e.g., all-projects.md has projects), don't overwrite it. Only create scaffolding for missing files.
- **Show what's still incomplete.** If placeholders remain, list them clearly so the user knows what to fill later.
- **Leave SOUL.md personality intact.** Only replace the "Who You Are" section. Don't touch personality, self-improvement, or behavioral sections.

## Validation

After setup, confirm:
- [ ] SOUL.md has user's name and role (no `{{USER_NAME}}` remaining)
- [ ] shared/config.md has at minimum: team name populated
- [ ] If Atlassian configured: site URL and project key are populated
- [ ] Required workspace files/folders exist
- [ ] Summary presented with clear next steps
- [ ] Any remaining placeholders listed explicitly
