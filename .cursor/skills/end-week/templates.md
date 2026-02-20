# End Week Templates

## Executive Summary Template

```markdown
## Weekly Executive Summary

### Fires - Week of [Monday] - [Friday]

**[Fire Name] ([Date])** - [Ticket Link]: [Brief description]. [Investigation/context if relevant]. **Resolution:** [How it was resolved or current status].

**[Fire Name 2] ([Date])** - [Ticket Link]: [Brief description]. **Resolution:** [Status].

---

### [Project Name] - Week of [Monday] - [Friday]

**Current Status:** [2-3 sentence summary of progress and current state]

**Accomplishments:**
- [Accomplishment 1]
- [Accomplishment 2]
[ONLY include this section if there are accomplishments listed in the weekly update]

---

[Additional project summaries...]
```

## all-projects.md Sync Format

```markdown
## [Project Name]

**Confluence URL:** [URL]
[ONLY include this line if a Confluence URL actually exists]

**Jira Project Ticket:** [[PROJECT-KEY] - [Project Name]](https://{{ATLASSIAN_SITE}}/browse/[PROJECT-KEY])

**Status:** [Status]

**Jira Epic Tickets:**
- [[EPIC-KEY] - [Epic Name]](https://{{ATLASSIAN_SITE}}/browse/[EPIC-KEY]) (Status: [Status])

**Week of [Monday] - [Friday]:**
- **Status:** [Executive summary - project update/status only, with full technical detail]

---
```

## Week Ended Marker

```markdown
---
**Week Ended:** Jan 23, 2026 17:00
**Week Status:** Completed
---
```

## Priorities for Next Week Section

```markdown
## Priorities for Next Week

1. [Priority 1]
2. [Priority 2]
3. [Priority 3]
4. [Priority 4]
```

## Status Calculation Rules

See [shared/config.md](../shared/config.md) for status calculation rules and Confluence sync details (Cloud ID, page IDs).

## Example Executive Summary

```markdown
## Weekly Executive Summary

### Fires - Week of Jan 19 - Jan 24

**Database Connection Issue (Jan 20)** - [PRJ-2001](https://{{ATLASSIAN_SITE}}/browse/PRJ-2001): Production database connections were exhausted during peak hours. Identified connection pool misconfiguration. **Resolution:** Increased connection pool size and added monitoring alerts.

---

### Project Alpha - Week of Jan 19 - Jan 24

**Current Status:** Team has begun addressing architecture decision action items from the recent Architecture Review, with alignment conversations ongoing with the Security team and other engineers. Multiple documentation pieces were created mid-week and are currently under review. Implementation work has started on the identity provider side.

**Timeline Update:** {{TEAM_NAME}} team's work is projected to complete by end of February/early March. Full rollout support expected in March.

---
```
