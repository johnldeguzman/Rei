# Done Projects Templates

## Done Project Entry Format

```markdown
## [Project Name]

**Confluence URL:** [URL if exists]

**Jira Project Ticket:** [[PROJECT-KEY] - [Project Name]](https://{{ATLASSIAN_SITE}}/browse/[PROJECT-KEY])

**Completed:** [MMM DD, YYYY]

**Contributors:** [Name1, Name2, Name3]

**Jira Epic Tickets:**
- [[EPIC-KEY] - [Epic Name]](https://{{ATLASSIAN_SITE}}/browse/[EPIC-KEY]) (Status: Done ✅)

**Final Status Summary:**
[Most recent weekly status summary]

---
```

## Example Entry

```markdown
## Project Alpha - Technical Walkthrough (ALPHATW)

**Confluence URL:** https://{{ATLASSIAN_SITE}}/wiki/spaces/TEAM/pages/123456789/Project+Alpha+Technical+Walkthrough

**Jira Project Ticket:** [PROJ-100 - Project Alpha - Technical Walkthrough (ALPHATW)](https://{{ATLASSIAN_SITE}}/browse/PROJ-100)

**Completed:** Jan 15, 2026

**Contributors:** Engineer A, Engineer B, Engineer C

**Jira Epic Tickets:**
- [PROJ-101 - Discovery](https://{{ATLASSIAN_SITE}}/browse/PROJ-101) (Status: Done ✅)
- [PROJ-102 - Documentation](https://{{ATLASSIAN_SITE}}/browse/PROJ-102) (Status: Done ✅)

**Final Status Summary:**
Technical walkthrough completed and presented to stakeholders. All documentation finalized and published to Confluence. Discovery phase findings incorporated into final recommendations.

---
```

## Entry Rules

- Add projects in **reverse chronological order** (most recent first)
- **Confluence URL**: Only include if a Confluence page exists for the project
- **Completed date**: Use `MMM DD, YYYY` format (see [shared/config.md](../shared/config.md) for date formats)
- **Contributors**: All engineers who worked on the project — ask user if not clear from Jira
- **Final Status Summary**: Use the most recent weekly executive summary for this project
