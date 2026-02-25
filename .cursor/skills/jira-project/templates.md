# Jira Project Templates

## all-projects.md Entry Format

```markdown
## [Project Title]

**Confluence URL:** [Confluence page URL]
[ONLY include this line if a Confluence URL actually exists]

**Jira Project Ticket:** [[PROJECT-KEY] - [Project Name]](https://{{ATLASSIAN_SITE}}/browse/[PROJECT-KEY])

**Status:** [Status]

**Jira Epic Tickets:**
- [[EPIC-KEY-1] - [Epic Name]](https://{{ATLASSIAN_SITE}}/browse/[EPIC-KEY-1]) (Status: [Status])
- [[EPIC-KEY-2] - [Epic Name]](https://{{ATLASSIAN_SITE}}/browse/[EPIC-KEY-2]) (Status: [Status])

---
```

## Example Entry

```markdown
## Project Alpha (ALPHA)

**Confluence URL:** https://{{ATLASSIAN_SITE}}/wiki/spaces/TEAM/pages/123456789/Project+Alpha

**Jira Project Ticket:** [PROJ-100 - Project Alpha (ALPHA)](https://{{ATLASSIAN_SITE}}/browse/PROJ-100)

**Status:** In Progress

**Jira Epic Tickets:**
- [PROJ-101 - Discovery](https://{{ATLASSIAN_SITE}}/browse/PROJ-101) (Status: In Progress)

---
```

## Confluence Page Body Template

```markdown
## Project Overview

[Include project description from Project ticket if available]

## Project Details

**Customer:** [Customer name if available]

**Project Key:** [PROJECT-KEY]

## Epics

Use the following JQL query to view all epics under this project:

project = {{JIRA_PROJECT_KEY}} AND issuetype = Epic AND parent in (project = {{JIRA_PROJECT_KEY}} AND parent = [PROJECT-KEY] AND issuetype in (Discovery Milestone, Delivery Milestone, Rollout Milestone)) ORDER BY created DESC

[View Epics in Jira](https://{{ATLASSIAN_SITE}}/issues/?jql=...)
```

## Example Visual Diagram

```
Project: Wiz Platform Evaluation (WIZPLAT)

├── Discovery Phase (Discovery Milestone)
│   ├── Epic: Current State & Requirements Analysis
│   │   ├── Technical Task: Inventory all current scanners
│   │   ├── Technical Task: Identify system owners
│   │   └── Technical Task: Document deduplication logic
│   └── Epic: Wiz Platform Feature Assessment
│       ├── Technical Task: Review Wiz UVM documentation
│       └── Technical Task: Assess Wiz UVM's compatibility

├── Integration & Testing (Delivery Milestone)
│   ├── Epic: Wiz UVM Proof of Concept
│   │   ├── Technical Task: Set up Wiz UVM test environment
│   │   └── Technical Task: Integrate with current scanner
│   └── Epic: Issue Remediation & Feedback
│       └── Technical Task: Collect feedback from stakeholders

└── Rollout & Recommendations (Rollout Milestone)
    └── Epic: Final Evaluation & Handover
        ├── Technical Task: Finalize evaluation report
        └── Technical Task: Present findings to stakeholders
```

## Action Verbs Reference

### Discovery/Research
- `Inventory` - List and catalog existing items
- `Review` - Examine existing documentation
- `Identify` - Find and name specific items
- `Document` - Record information
- `Assess` - Evaluate or judge
- `Analyze` - Examine in detail

### Implementation/Integration
- `Set up` - Configure environment
- `Integrate` - Connect systems
- `Configure` - Set up parameters
- `Test` - Verify functionality
- `Validate` - Confirm correctness

### Analysis/Reporting
- `Collect` - Gather information
- `Finalize` - Complete documents
- `Present` - Share findings
- `Plan` - Create strategy

## Status Calculation Rules & File Locations

See [shared/config.md](../shared/config.md) for status calculation rules, file locations, Confluence IDs, and all-projects.md entry format.
