# Rei (零) — Getting Started

Rei is an engineering management partner that runs inside Cursor. It handles weekly tracking, project planning, Jira/Confluence orchestration, multi-perspective document review, and continuous process improvement — all through natural language commands.

This guide covers how to set Rei up for yourself, how to use it, and how to make it yours.

---

## Quick Start

The fastest way to get going:

1. Clone or download this repo and open it in Cursor
2. Say **"set me up"** — Rei will walk you through an interactive setup wizard
3. Say **"check my setup"** to verify everything is configured
4. Say **"start my week"** to create your first weekly tracking file

That's it. The setup wizard handles identity, Atlassian config (optional), and workspace scaffolding. Everything below is reference for customizing further.

---

## How It Works

Rei has four layers:

| Layer | What it is | You touch it? |
|-------|-----------|--------------|
| **Engine** | Rules (`.cursor/rules/`) and skills (`.cursor/skills/`) — the behavioral logic | Rarely. Customizable, but works out of the box. |
| **Config** | `SOUL.md` and `shared/config.md` — your identity, team, and Atlassian settings | Once during setup, then occasionally. |
| **Data** | `all-projects.md`, `weekly-updates/`, `notes/`, etc. — your actual work | Every day. This is yours. |
| **Templates** | `template/` — starter files with `{{PLACEHOLDER}}` syntax for new users | Never (unless you're customizing the setup experience). |

The `template/` folder contains clean, portable versions of all config and data files with placeholder syntax. When a new user says "set me up", the setup skill reads from `template/`, prompts for their values, and writes personalized working files. Your actual working files are never touched by the template system.

---

## Core Mode vs. Full Mode

Rei works in two modes depending on whether Atlassian (Jira/Confluence) is configured:

### Core Mode (no Atlassian)

Works immediately with no external dependencies:

| Feature | Trigger |
|---------|---------|
| Weekly tracking | "start my week", "end my week" |
| Task management | "mark X as done", "update for [project]" |
| Agent team reviews | Share a PRD, RFC, or design doc |
| Performance reviews | "create performance reviews" (local data only) |
| Self-improvement loop | Automatic — logs to IMPROVEMENTS.md |

### Full Mode (with Atlassian)

Everything in Core, plus:

| Feature | Trigger |
|---------|---------|
| Jira project creation | "create a project" |
| Timeline sync | "sync timeline" |
| Jira health checks | "jira health check" |
| Confluence sync | Automatic during end-of-week |
| Onboarding → Confluence | "onboard [name]" (publishes to Confluence) |

To enable Full Mode, say "set me up" and answer "yes" when asked about Jira/Confluence.

---

## 1. Setup (Interactive)

Say **"set me up"** and Rei will:

1. **Ask for your identity** — name, role, team, communication style
2. **Ask about Atlassian** — site URL, project key, custom fields (all optional)
3. **Write your config** — populates `SOUL.md` and `shared/config.md`
4. **Scaffold workspace** — creates any missing files and folders
5. **Show a summary** — what's configured, what's still placeholder, what to try first

If you've already run setup and want to change something, run it again — it'll ask before overwriting.

### Manual setup (alternative)

If you prefer to configure manually:

1. Copy `template/SOUL.md` to `SOUL.md` — fill in the "Who You Are" section
2. Copy `template/config.md` to `.cursor/skills/shared/config.md` — replace `{{PLACEHOLDER}}` values
3. Copy starter files from `template/` for `all-projects.md`, `done-projects.md`, etc.

### Validate your setup

Say **"check my setup"** at any time. Rei checks:
- Identity fields are populated (not placeholder)
- Required workspace files exist
- Atlassian connectivity works (if configured)

---

## 2. Your Identity (SOUL.md)

Rei's personality and behavioral directives live in `SOUL.md`. The setup wizard fills the "Who You Are" section; you can customize everything else.

| Section | What to change |
|---------|---------------|
| **Who You Are** | Your name, role, communication style, expectations (filled by setup) |
| **Core Personality** | Adjust tone — Rei defaults to warm-but-direct with no fluff |
| **What I Never Do** | Add or remove constraints based on your preferences |
| **How I Communicate** | Set defaults — concise vs. thorough, technical depth, formality level |

Changes to `SOUL.md` take effect immediately — next conversation.

---

## 3. Configuration (shared/config.md)

`.cursor/skills/shared/config.md` centralizes all variable values used across skills:

| Section | What it contains |
|---------|-----------------|
| **User Configuration** | Your name, role, team name |
| **Atlassian Configuration** | Site URL, project key, custom field IDs, Confluence page IDs, issue type IDs |
| **File Locations** | Workspace-relative paths for all tracked files |
| **Date Formats** | Naming conventions for weekly files, timestamps, etc. |
| **Jira Queries** | JQL templates with your project key |
| **Error Handling** | Fallback chain behavior for API failures |

Values use `{{PLACEHOLDER}}` syntax. The setup wizard replaces them; anything left as placeholder is easy to find and fill later.

---

## 4. Rules

Rules live in `.cursor/rules/` and control Rei's behavior every conversation:

| Rule | What it does |
|------|-------------|
| `soul.mdc` | Loads Rei's core identity from SOUL.md |
| `contextAwareness.mdc` | Checks workspace state before responding to non-trivial requests |
| `weeklyUpdates.mdc` | Routes "update for ..." messages to the correct weekly file |
| `taskCompletion.mdc` | Handles "mark X as done" with checkbox + strikethrough |
| `projectPlanning.mdc` | Enforces minimum project duration and single-project sequencing |
| `selfImprovement.mdc` | Logs observations to IMPROVEMENTS.md when rules miss edge cases |
| `agentTeam.mdc` | Detects when content warrants multi-perspective review |
| `atlassianMcpErrors.mdc` | Handles Atlassian errors with fallback chain: MCP → CLI → manual checklist |

### Customizing rules

- Edit any `.mdc` file directly — changes take effect next conversation.
- To add a new rule, create a new `.mdc` file in `.cursor/rules/`.
- To disable a rule, delete or rename it (e.g., `taskCompletion.mdc.disabled`).
- If you don't use Jira/Confluence, you can remove `atlassianMcpErrors.mdc` and `projectPlanning.mdc`.

---

## 5. Workflows

All workflows are triggered by natural language. No special syntax needed.

### Weekly operations

| Say this | What happens |
|----------|-------------|
| **"start my week"** | Creates a new weekly file with your top 4 priorities and active project carryover |
| **"end my week"** | Executive summaries, Jira verification, Confluence sync, next-week priorities |
| **"update for [project]"** | Adds progress to the current weekly file under the matching project |
| **"mark [task] as done"** | Checks off the task with `- [x] ~~strikethrough~~` |

### Project management (Full Mode)

| Say this | What happens |
|----------|-------------|
| **"create a project"** | Walks through Jira project creation with visual planning and Confluence page |
| **"sync timeline"** | Rebuilds project-timeline.md from Jira (skips if data unchanged) |
| **"jira health check"** | Scans for stale tickets, missing dates, overdue items, status mismatches |

### Multi-perspective review (Agent Team)

| Say this | What happens |
|----------|-------------|
| **Share a PRD, RFC, or design doc** | Auto-spawns specialist agents for multi-perspective review |
| **"is this ready to launch?"** | Runs launch readiness assessment |
| **"run a team review on this"** | Manually triggers a team review |

The agent team writes findings to `.agent-team/` and synthesizes a verdict with top risks and recommendations. Specialists: Security, Engineering, Ops, Product.

### People operations

| Say this | What happens |
|----------|-------------|
| **"create performance reviews"** | Generates review templates from Jira/Confluence data |
| **"onboard [name]"** | Creates an onboarding plan, drafts locally then publishes to Confluence |

### Setup & validation

| Say this | What happens |
|----------|-------------|
| **"set me up"** | Interactive setup wizard — identity, Atlassian config, workspace scaffolding |
| **"check my setup"** | Validates config completeness, workspace files, and optional Atlassian connectivity |

---

## 6. The Self-Improvement Loop

Rei monitors how rules and workflows perform and flags issues proactively.

- **In the moment:** Flags edge cases and proposes fixes right then.
- **At breakpoints:** Surfaces non-urgent patterns at end-of-week.
- **Everything gets logged** in `IMPROVEMENTS.md` with: what happened, impact, proposed fix, status.

Lifecycle: **Proposed** → **In Progress** (you approve) → **Completed** (fix is live).

---

## 7. Skills Reference

Skills are multi-step workflow definitions in `.cursor/skills/`. Complex skills include ASCII decision-flow diagrams.

| Skill | Trigger | Mode |
|-------|---------|------|
| `setup` | "set me up" | Core |
| `validate-setup` | "check my setup" | Core |
| `start-week` | "start my week" | Core |
| `end-week` | "end my week" | Core (Jira/Confluence optional) |
| `agent-team` | Auto-detected or manual | Core |
| `performance-reviews` | "create performance reviews" | Core (Jira optional) |
| `jira-project` | "create a project" | Full |
| `timeline-sync` | "sync timeline" | Full |
| `done-projects` | Project status → Done | Full |
| `jira-health-check` | "jira health check" | Full |
| `onboarding` | "onboard [name]" | Full |

---

## 8. Atlassian Integration (Full Mode)

If you use Jira and Confluence, enable the Atlassian MCP server in Cursor.

### Auth and error handling

When Atlassian auth expires, Rei follows a 4-step fallback chain:

1. **MCP retry** — displays re-auth message, waits for you to toggle MCP off/on
2. **CLI fallback** — tries `acli` or `curl` against the REST API (cross-platform: skips Homebrew on Windows)
3. **Manual checklist** — captures intended changes so nothing is lost
4. **Local files unaffected** — workspace writes proceed regardless

### What you'll need

- **Jira project key** (e.g., `ENG`, `PLAT`) — required for JQL queries
- **Atlassian site URL** (e.g., `mycompany.atlassian.net`) — required for API calls
- **Confluence page IDs** — optional, needed for Confluence sync
- **Custom field IDs** — optional, needed for timeline sync and project planning

The setup wizard collects all of these. You can also fill them in `shared/config.md` directly.

---

## 9. Tips

- **Run "set me up" first.** It handles everything. Manual setup is the fallback.
- **Run "check my setup" if something seems off.** It'll tell you exactly what's missing.
- **Keep the weekly file current.** Rei builds summaries and reports from what's logged there.
- **Use the triggers.** Natural language commands activate tested, multi-step workflows.
- **Say when something's off.** Rei will propose a system fix, not just a one-time adjustment.
- **Review IMPROVEMENTS.md periodically.** Clearing proposed items keeps the system sharp.
- **Let the agent team review important docs.** Share PRDs or design docs for multi-perspective review.

---

**Last Updated:** February 20, 2026

**This is a living document.** As you customize Rei for your workflow, update this README to reflect your setup. Rei can help with that too — say "update the README."
