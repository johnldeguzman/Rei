# Rei (零) — Engineering Management Partner

Rei is an AI engineering management partner that runs inside [Cursor](https://cursor.sh). It handles weekly tracking, project planning, Jira/Confluence orchestration, multi-perspective document review, and continuous process improvement — all through natural language.

## Quick Start

1. Clone this repo and open it in Cursor
2. Say **"set me up"** — Rei walks you through an interactive setup wizard
3. Say **"check my setup"** to verify everything is configured
4. Say **"start my week"** to create your first weekly tracking file

That's it. See `README-getting-started.md` for the full guide.

## What's Inside

```
.cursor/
├── rules/          ← 9 behavioral rules (always active)
└── skills/         ← 11 workflow skills (triggered by natural language)
    └── shared/config.md  ← centralized config (setup wizard fills this)
SOUL.md             ← Rei's identity + preferences + working memory (setup wizard fills this)
README-getting-started.md  ← Full documentation
```

## What You Can Do

### Works immediately (no Jira/Confluence needed)
- **"start my week"** / **"end my week"** — weekly tracking with priorities and summaries
- **"mark X as done"** — task completion with checkbox + strikethrough
- **"remember that"** / **"save that"** — persist preferences and decisions to working memory
- **"update for [project]"** — progress updates to the weekly file
- **Share a PRD or design doc** — multi-perspective review (Security, Engineering, Ops, Product)
- **"create performance reviews"** — review templates from local data

### With Jira/Confluence (optional)
- **"create a project"** — Jira project hierarchy with Confluence page
- **"sync timeline"** — Gantt charts and 6-month roadmap from Jira
- **"jira health check"** — stale tickets, missing dates, overdue items
- **"onboard [name]"** — onboarding plan published to Confluence

## Requirements

- [Cursor](https://cursor.sh) IDE
- Atlassian MCP server (optional — for Jira/Confluence features)
