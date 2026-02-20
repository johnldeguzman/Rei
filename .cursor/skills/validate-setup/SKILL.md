---
name: validate-setup
description: Validates that Rei is correctly configured. Checks SOUL.md, shared/config.md, workspace files, and optionally tests Atlassian connectivity. Produces a pass/fail health report.
---

# Validate Setup

Checks that Rei's configuration is complete and functional. Runs three tiers of validation — identity, workspace, and connectivity — and produces a clear pass/fail report.

## When This Activates

- User says "check my setup", "validate setup", "is Rei configured?", "health check on setup"
- Optionally run after the setup skill completes
- Useful after upgrading rules/skills to verify nothing broke

## Workflow

### Tier 1: Identity & Config (always runs)

Check that personal configuration is populated — no `{{PLACEHOLDER}}` values remaining in critical fields.

| Check | File | What to verify | Severity |
|-------|------|---------------|----------|
| User name set | `SOUL.md` | "Who You Are" section has a real name, not `{{USER_NAME}}` | 🔴 Required |
| User role set | `SOUL.md` | Role line has a real value, not `{{USER_ROLE}}` | 🔴 Required |
| Team name set | `shared/config.md` | `{{TEAM_NAME}}` is replaced | 🔴 Required |
| Atlassian site | `shared/config.md` | `{{ATLASSIAN_SITE}}` is replaced (if using Atlassian) | 🟡 Optional |
| Jira project key | `shared/config.md` | `{{JIRA_PROJECT_KEY}}` is replaced (if using Atlassian) | 🟡 Optional |
| Custom field IDs | `shared/config.md` | `{{TARGET_START_FIELD}}` and `{{TARGET_END_FIELD}}` are replaced | 🟡 Optional |
| Confluence page IDs | `shared/config.md` | At least `{{ALL_PROJECTS_PAGE_ID}}` is replaced (if using Confluence sync) | 🟡 Optional |

**How to check:** Read each file and search for `{{` — any remaining double-brace placeholders indicate incomplete config.

### Tier 2: Workspace Files (always runs)

Check that required files and folders exist.

| Check | Path | Severity |
|-------|------|----------|
| SOUL.md exists | `SOUL.md` | 🔴 Required |
| Config exists | `.cursor/skills/shared/config.md` | 🔴 Required |
| all-projects.md exists | `all-projects.md` | 🔴 Required |
| done-projects.md exists | `done-projects.md` | 🟡 Recommended |
| project-timeline.md exists | `project-timeline.md` | 🟡 Recommended |
| IMPROVEMENTS.md exists | `IMPROVEMENTS.md` | 🟡 Recommended |
| weekly-updates/ exists | `weekly-updates/` | 🔴 Required |
| notes/ exists | `notes/` | 🟡 Recommended |

**For missing recommended files:** Note them but don't fail. They'll be created automatically when the relevant skill runs.

### Tier 3: Connectivity (only if Atlassian is configured)

Only run this tier if `shared/config.md` has a non-placeholder Atlassian site URL and Jira project key.

| Check | How | Severity |
|-------|-----|----------|
| MCP available | Attempt a read-only Jira call (e.g., get a single issue or run a simple JQL query) | 🟡 Warning if fails |
| Project key valid | Run JQL: `project = {JIRA_PROJECT_KEY} AND issuetype = Project ORDER BY created DESC` with maxResults=1 | 🟡 Warning if 0 results |
| Confluence reachable | Attempt to read the All Projects page (if page ID is configured) | 🟡 Warning if fails |

**If MCP is not available or auth fails:** Don't trigger the re-auth flow. Just note: "Atlassian connectivity could not be verified. This may mean MCP needs to be toggled on, or auth has expired. Run a workflow that uses Jira to test."

**If MCP works but project key returns 0 results:** Note: "Project key '{KEY}' returned no results. Verify this is the correct Jira project key."

### Output: Health Report

Present the results as a structured report in the chat:

```
## Setup Health Check

### Identity & Config
- ✅ User name: [name]
- ✅ User role: [role]
- ✅ Team name: [team]
- ✅ Atlassian site: [site]  (or ⏭️ Skipped — not using Atlassian)
- ✅ Jira project key: [key]  (or ⏭️ Skipped)
- ⚠️ Custom field IDs: Still placeholder — needed for timeline sync
- ⚠️ Confluence page IDs: Still placeholder — needed for Confluence sync

### Workspace Files
- ✅ SOUL.md
- ✅ all-projects.md
- ✅ weekly-updates/
- ✅ shared/config.md
- ⚠️ project-timeline.md — missing (will be created on first "sync timeline")
- ⚠️ done-projects.md — missing (will be created on first project completion)

### Connectivity (Atlassian)
- ✅ Jira: Connected, project key valid (returned N projects)
- ✅ Confluence: All Projects page reachable
(or)
- ⏭️ Skipped — Atlassian not configured

### Result: [PASS / PASS WITH WARNINGS / NEEDS SETUP]

[If PASS]: "Rei is fully configured. Try 'start my week' to get going."
[If PASS WITH WARNINGS]: "Core setup is complete. The warnings above are optional — fill them when you need the related features."
[If NEEDS SETUP]: "Required config is missing. Run 'set me up' to complete setup."

### Remaining Placeholders (if any)
| File | Placeholder | Needed For |
|------|------------|-----------|
| shared/config.md | {{TARGET_START_FIELD}} | Timeline sync, project planning |
| shared/config.md | {{ALL_PROJECTS_PAGE_ID}} | Confluence sync |
```

## Result Categories

| Result | Criteria |
|--------|----------|
| **PASS** | All Tier 1 required checks pass, all Tier 2 required files exist, Tier 3 connectivity passes (if configured) |
| **PASS WITH WARNINGS** | All required checks pass, but optional/recommended items are missing or placeholder |
| **NEEDS SETUP** | Any Tier 1 required check fails (name, role, or team name still placeholder) or Tier 2 required files missing |

## Critical Rules

- **Read-only.** This skill never writes files. It only reads and reports.
- **Don't trigger re-auth.** If Atlassian connectivity fails, note it and move on. Don't display the ALL CAPS re-auth message — this is a health check, not a workflow.
- **Be specific about what's missing.** Don't say "config incomplete" — say exactly which placeholder is still there and what feature it blocks.
- **Offer the fix.** If NEEDS SETUP, tell the user to run "set me up". If PASS WITH WARNINGS, tell them which features are limited until the warnings are resolved.
