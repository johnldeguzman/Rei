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
  │   │   ├─ Yes → Prompt for site URL, project key, field IDs
  │   │   │   ├─ "Do you know your Confluence page IDs?"
  │   │   │   │   ├─ Yes → Prompt for page IDs
  │   │   │   │   └─ No → Leave as placeholders, note in output
  │   │   │   └─ "Do you know your Jira issue type IDs?"
  │   │   │       ├─ Yes → Prompt for issue type IDs
  │   │   │       └─ No → Leave as placeholders, note in output
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

If **yes**, collect:

#### Required Atlassian values

| Value | Prompt | Example |
|-------|--------|---------|
| Atlassian Site | "What's your Atlassian site URL? (e.g., mycompany.atlassian.net)" | `mycompany.atlassian.net` |
| Jira Project Key | "What's your Jira project key? (the prefix on your tickets, e.g., ENG, PLAT, ID)" | `ENG` |

#### Optional Atlassian values (ask but don't require)

| Value | Prompt | Fallback |
|-------|--------|----------|
| Target Start Field | "Custom field ID for 'Target Start' date? (Check Jira admin or leave blank)" | Leave as `{{TARGET_START_FIELD}}` |
| Target End Field | "Custom field ID for 'Target End' date?" | Leave as `{{TARGET_END_FIELD}}` |
| Cloud ID | "Atlassian Cloud ID? (Leave blank if unsure)" | Leave as `{{CLOUD_ID}}` |
| Confluence Space ID | "Confluence Space ID? (Leave blank if unsure)" | Leave as `{{CONFLUENCE_SPACE_ID}}` |
| All Projects Page ID | "Confluence Page ID for your 'All Projects' page? (Leave blank to skip Confluence sync)" | Leave as `{{ALL_PROJECTS_PAGE_ID}}` |
| Done Projects Page ID | "Confluence Page ID for your 'Done Projects' page?" | Leave as `{{DONE_PROJECTS_PAGE_ID}}` |
| Timeline Page ID | "Confluence Page ID for your project timeline page?" | Leave as `{{TIMELINE_PAGE_ID}}` |
| Onboarding Parent Page ID | "Parent page ID for onboarding docs?" | Leave as `{{ONBOARDING_PARENT_PAGE_ID}}` |
| Tech Discovery Parent ID | "Parent page ID for technical discovery docs?" | Leave as `{{TECH_DISCOVERY_PARENT_ID}}` |
| Active Projects Parent ID | "Parent page ID for active project pages?" | Leave as `{{ACTIVE_PROJECTS_PARENT_ID}}` |

#### Jira Issue Type IDs (ask but don't require)

Ask: **"Do you know your Jira issue type IDs? (These are specific to your Jira instance. If unsure, leave them blank and we'll use defaults.)"**

If yes, collect: Project, Discovery Milestone, Delivery Milestone, Rollout Milestone, Epic, Technical Task IDs.

If no, leave as `{{ISSUE_TYPE_*}}` placeholders.

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
- **Don't flood the user with questions.** Group related questions together. For Atlassian, show required fields first, then ask "want to configure advanced settings?" for the optional ones.
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
