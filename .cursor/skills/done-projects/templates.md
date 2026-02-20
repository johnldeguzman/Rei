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
## Example Project - Technical Walkthrough (EPTW)

**Confluence URL:** https://{{ATLASSIAN_SITE}}/wiki/spaces/ID/pages/123456789/Example+Project+Technical+Walkthrough

**Jira Project Ticket:** [PRJ-1001 - Example Project - Technical Walkthrough (EPTW)](https://{{ATLASSIAN_SITE}}/browse/PRJ-1001)

**Completed:** Jan 15, 2026

**Contributors:** [Contributor 1], [Contributor 2], [Contributor 3]

**Jira Epic Tickets:**
- [PRJ-1002 - Discovery](https://{{ATLASSIAN_SITE}}/browse/PRJ-1002) (Status: Done ✅)
- [PRJ-1003 - Documentation](https://{{ATLASSIAN_SITE}}/browse/PRJ-1003) (Status: Done ✅)

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
