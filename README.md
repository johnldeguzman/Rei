# Rei (零) — Engineering Management Partner

Rei helps EMs and tech leads keep weekly rhythm, project health, and Jira/Confluence in sync — without leaving [Cursor](https://cursor.sh).

It handles weekly tracking, project planning, Jira/Confluence orchestration, multi-perspective document review, and continuous process improvement — all through plain-English commands.

## Quick Start

1. Clone this repo and open it in Cursor
2. Say **"set me up"** — Rei walks you through an interactive setup wizard
3. Say **"check my setup"** to verify everything is configured
4. Say **"start my week"** to create your first weekly tracking file

That's it. See [README-getting-started.md](README-getting-started.md) for the full guide.

## What's Inside

```
.cursor/
├── rules/               ← 10 behavioral rules (always active)
└── skills/              ← 10 workflow skills (triggered by plain-English commands)
    └── shared/config.md ← centralized config (setup wizard fills this)
SOUL.md                  ← Rei's identity + your preferences + working memory
README-getting-started.md
```

## What You Can Do

### Works immediately (no Jira/Confluence needed)

- **"start my week"** / **"end my week"** — weekly tracking with priorities and executive summaries
- **"mark X as done"** — task completion with checkbox + strikethrough
- **"remember that"** / **"save that"** — persist preferences and decisions to working memory
- **"update for [project]"** — progress updates routed to the current weekly file
- **Share a PRD or design doc** — multi-perspective review from Security, Engineering, Ops, Product, and AI specialists
- **"add this as a project idea"** — capture ideas to the project backlog

### With Jira/Confluence (optional)

- **"create a project"** — Jira project hierarchy with visual planning and Confluence page
- **"sync timeline"** — Gantt charts and 6-month roadmap built from Jira
- **"jira health check"** — stale tickets, missing dates, overdue items, status mismatches
- **"onboard [name]"** — onboarding plan drafted locally, then published to Confluence

## Requirements

- [Cursor](https://cursor.sh) IDE (macOS, Linux, or Windows with WSL)
- Cursor's built-in Atlassian plugin (optional — for Jira/Confluence features)

## License

MIT
