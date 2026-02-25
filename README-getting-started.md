# Rei (零) — Getting Started

Rei is an engineering management partner that runs inside Cursor. It handles weekly tracking, project planning, Jira/Confluence orchestration, multi-perspective document review, and continuous process improvement — all through plain-English commands.

This guide covers setup, usage, and customization.

---

## Table of Contents

1. [Quick Start](#quick-start)
2. [How It Works](#how-it-works)
3. [Setup](#1-setup-interactive)
4. [Core Mode vs. Full Mode](#2-core-mode-vs-full-mode)
5. [Your Identity (SOUL.md)](#3-your-identity-soulmd)
6. [Configuration (shared/config.md)](#4-configuration-sharedconfigmd)
7. [Rules](#5-rules)
8. [Workflows](#6-workflows)
9. [Skills Reference](#7-skills-reference)
10. [The Self-Improvement Loop](#8-the-self-improvement-loop)
11. [Atlassian Integration](#9-atlassian-integration-full-mode)
12. [Tips](#10-tips)
13. [Global Setup (Optional)](#11-global-setup-optional)

---

## Quick Start

1. Clone or download this repo and open it in Cursor
2. Say **"set me up"** — Rei walks you through an interactive setup wizard
3. Say **"check my setup"** to verify everything is configured
4. Say **"start my week"** to create your first weekly tracking file

That's it. The setup wizard handles identity, Atlassian config (optional), and workspace scaffolding.

**Try this first:** After setup, say **"start my week"**, then **"update for [your project name]"** to see Rei in action.

Everything below is reference for customizing further.

---

## How It Works

Rei has three layers:

| Layer | What it is | When to edit |
|-------|-----------|-------------|
| **Engine** | Rules (`.cursor/rules/`) and skills (`.cursor/skills/`) — the behavioral logic | Rarely. Customizable, but works out of the box. |
| **Config** | `SOUL.md` and `.cursor/skills/shared/config.md` — your identity, team, and Atlassian settings | Once during setup, then occasionally. |
| **Data** | `all-projects.md`, `weekly-updates/`, `notes/`, etc. — your actual work | Every day. This is yours. |

Files in the repo root (`SOUL.md`, `all-projects.md`, etc.) ship with `{{PLACEHOLDER}}` syntax. When you say "set me up", the setup skill reads these placeholders, prompts for your values, and writes personalized working files. Your working files are separate from the template defaults.

---

## 1. Setup (Interactive)

Say **"set me up"** and Rei will:

1. **Ask for your identity** — name, role, team, communication style
2. **Ask about Atlassian** — site URL, project key, custom fields (all optional)
3. **Write your config** — populates `SOUL.md` and `.cursor/skills/shared/config.md`
4. **Scaffold workspace** — creates any missing files and folders
5. **Show a summary** — what's configured, what's still placeholder, what to try first

If you've already run setup and want to change something, run it again — it'll ask before overwriting.

### Manual setup (alternative)

If you prefer to configure manually:

1. Edit `SOUL.md` — fill in the "Who You Are" section with your name, role, and preferences
2. Edit `.cursor/skills/shared/config.md` — replace `{{PLACEHOLDER}}` values with your Atlassian details
3. Create starter files for `all-projects.md`, `done-projects.md`, `project-timeline.md` if they don't exist

### Validate your setup

Say **"check my setup"** at any time. Rei checks:
- Identity fields are populated (not placeholder)
- Required workspace files exist
- Atlassian connectivity works (if configured)

---

## 2. Core Mode vs. Full Mode

Rei works in two modes depending on whether Atlassian (Jira/Confluence) is configured.

**Core Mode works without any external accounts or integrations.**

### Core Mode (no Atlassian)

| Feature | Trigger |
|---------|---------|
| Weekly tracking | "start my week", "end my week" |
| Task management | "mark X as done", "update for [project]" |
| Working memory | "remember that", "save that", "note that for next time" |
| Agent team reviews | Share a PRD, RFC, or design doc |
| Project idea capture | "add this as a project idea" |
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

## 3. Your Identity (SOUL.md)

Rei's personality and behavioral directives live in `SOUL.md`. The setup wizard fills the "Who You Are" section; you can customize everything else.

| Section | What to change |
|---------|---------------|
| **Who You Are** | Your name, role, communication style, expectations (filled by setup) |
| **Core Personality** | Adjust tone — Rei defaults to warm-but-direct with no fluff |
| **Working Memory** | Persists preferences, people context, and key decisions across sessions. Say "remember that" or Rei will proactively offer to log decisions during conversations. |
| **What I Never Do** | Add or remove constraints based on your preferences |
| **How I Communicate** | Set defaults — concise vs. thorough, technical depth, formality level |

Changes to `SOUL.md` take effect immediately — next conversation.

---

## 4. Configuration (shared/config.md)

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

## 5. Rules

Rules live in `.cursor/rules/` and control Rei's behavior every conversation:

| Rule | What it does |
|------|-------------|
| `soul.mdc` | Loads Rei's core identity from SOUL.md |
| `contextAwareness.mdc` | Checks workspace state before responding to non-trivial requests |
| `memory.mdc` | Manages persistent memory — write gate, decision capture, staleness checks |
| `weeklyUpdates.mdc` | Routes "update for ..." messages to the correct weekly file |
| `taskCompletion.mdc` | Handles "mark X as done" with checkbox + strikethrough |
| `projectPlanning.mdc` | Enforces minimum project duration and single-project sequencing |
| `projectIdeas.mdc` | Captures project ideas to the backlog when triggered |
| `selfImprovement.mdc` | Logs observations to IMPROVEMENTS.md when rules miss edge cases |
| `agentTeam.mdc` | Detects when content warrants multi-perspective review |
| `atlassianMcpErrors.mdc` | Handles Atlassian plugin errors with fallback chain (MCP → CLI → manual checklist) |

### Customizing rules

- Edit any `.mdc` file directly — changes take effect next conversation.
- To add a new rule, create a new `.mdc` file in `.cursor/rules/`.
- To disable a rule, delete or rename it (e.g., `taskCompletion.mdc.disabled`).
- If you don't use Jira/Confluence, you can safely remove `atlassianMcpErrors.mdc` and `projectPlanning.mdc`.

---

## 6. Workflows

All workflows are triggered by plain-English commands. No special syntax needed.

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
| **"add this as a project idea"** | Captures an idea to the project backlog for future triage |

### Multi-perspective review (Agent Team)

| Say this | What happens |
|----------|-------------|
| **Share a PRD, RFC, or design doc** | Auto-spawns specialist agents for multi-perspective review |
| **"is this ready to launch?"** | Runs launch readiness assessment |
| **"run a team review on this"** | Manually triggers a full team review |
| **"research [topic]"** | Layered research — maps the landscape, then deep dives per domain |

The agent team writes findings to `.agent-team/` and synthesizes a verdict with top risks and recommendations. Specialists: Security, Engineering, Ops, Product, AI.

### Team operations

| Say this | What happens |
|----------|-------------|
| **"onboard [name]"** | Creates an onboarding plan, drafts locally then publishes to Confluence |

### Setup & validation

| Say this | What happens |
|----------|-------------|
| **"set me up"** | Interactive setup wizard — identity, Atlassian config, workspace scaffolding |
| **"check my setup"** | Validates config completeness, workspace files, and optional Atlassian connectivity |

---

## 7. Skills Reference

Skills are multi-step workflow definitions in `.cursor/skills/`.

| Skill | Trigger | Mode |
|-------|---------|------|
| `setup` | "set me up" | Core |
| `validate-setup` | "check my setup" | Core |
| `start-week` | "start my week" | Core |
| `end-week` | "end my week" | Core (Jira/Confluence optional) |
| `agent-team` | Auto-detected or "run a team review" | Core |
| `done-projects` | Project status → Done | Core |
| `jira-project` | "create a project" | Full |
| `timeline-sync` | "sync timeline" | Full |
| `jira-health-check` | "jira health check" | Full |
| `onboarding` | "onboard [name]" | Full |

---

## 8. The Self-Improvement Loop

Rei monitors how rules and workflows perform and flags issues proactively.

- **In the moment:** Flags edge cases and proposes fixes right then.
- **At breakpoints:** Surfaces non-urgent patterns at end-of-week.
- **Everything gets logged** in `IMPROVEMENTS.md` with: what happened, impact, proposed fix, status.

Lifecycle: **Proposed** → **In Progress** (you approve) → **Completed** (fix is live).

---

## 9. Atlassian Integration (Full Mode)

If you use Jira and Confluence, enable the Atlassian plugin in Cursor:

**Cursor Settings → Extensions → Atlassian** — enable the built-in Atlassian plugin (`plugin-atlassian-atlassian`). Follow the prompts to authenticate with your Atlassian account.

### Auth and error handling

When Atlassian auth expires, Rei follows a 4-step fallback chain:

1. **Plugin retry** — displays re-auth message, waits for you to toggle the plugin off/on
2. **CLI fallback** — tries `acli` or `curl` against the REST API
3. **Manual checklist** — captures intended changes so nothing is lost
4. **Local files unaffected** — workspace writes proceed regardless

### What you'll need

- **Jira project key** (e.g., `ENG`, `PLAT`) — required for JQL queries
- **Atlassian site URL** (e.g., `mycompany.atlassian.net`) — required for API calls
- **Confluence page IDs** — optional, needed for Confluence sync
- **Custom field IDs** — optional, needed for timeline sync and project planning

The setup wizard collects all of these. You can also fill them in `.cursor/skills/shared/config.md` directly.

---

## 10. Tips

- **Run "set me up" first.** It handles everything. Manual setup is the fallback.
- **Run "check my setup" if something seems off.** It'll tell you exactly what's missing.
- **Keep the weekly file current.** Rei builds summaries and reports from what's logged there.
- **Use the triggers.** Plain-English commands activate tested, multi-step workflows.
- **Say "remember that" when something's worth keeping.** Preferences, people context, and key decisions persist in Working Memory across sessions.
- **Say when something's off.** Rei will propose a system fix, not just a one-time adjustment.
- **Review IMPROVEMENTS.md periodically.** Clearing proposed items keeps the system sharp.
- **Let the agent team review important docs.** Share PRDs or design docs for multi-perspective analysis.

---

## 11. Global Setup (Optional)

By default, Rei only runs in the workspace where its rules and skills live. If you want Rei available across **all** Cursor workspaces, you can set up global symlinks.

### How it works

1. Create global directories and symlink each file/folder:
   - `~/.cursor/rules/*.mdc` → symlinks to `.cursor/rules/*.mdc`
   - `~/.cursor/skills/*` → symlinks to `.cursor/skills/*`
2. Symlink `SOUL.md` to your home directory: `~/SOUL.md`

This gives you single-load in every workspace (no token duplication) with a single source of truth in this repo.

### Things to know

- Edits to existing files propagate automatically through symlinks — no re-sync needed.
- Only **new** files need a symlink created.
- Rules load globally but only do useful work when the workspace has the right data files (e.g., `all-projects.md`, `weekly-updates/`). They won't interfere in unrelated projects.
- If you move this workspace, symlinks break — re-run the symlink commands from the new location.

---

**Last Updated:** February 25, 2026
