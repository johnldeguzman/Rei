# Start Week Templates

## Weekly File Template

```markdown
## Top Priorities for This Week

- [ ] [Priority 1]
- [ ] [Priority 2]
- [ ] [Priority 3]
- [ ] [Priority 4]

## Adhoc work to do:

- [ ] [Adhoc task 1]
- [ ] [Adhoc task 2]

---

[Project Name] (https://{{ATLASSIAN_SITE}}/browse/ID-XXXX)

Last week: [What was completed]. **This week, [what is expected] is expected.**

Progress this week:

[Another Project] (https://{{ATLASSIAN_SITE}}/browse/ID-YYYY)

Last week: [Summary from executive summary]. **This week, [expected work] is expected.**

Progress this week:

Any fires

Any highlights

---

**Note:** To mark a task as complete, change `- [ ]` to `- [x]` and wrap the task text with `~~strikethrough~~` syntax. Example: `- [x] ~~Completed task~~`
```

## Project Entry Format

```markdown
[Project Name] (Jira Link)

Last week: [Summary of last week's progress]. **This week, [expected deliverables] is expected.**

Progress this week:
```

## Example Output

For Monday, January 26, 2026:

**Filename:** `Jan262026-Jan302026.md`

```markdown
## Top Priorities for This Week

- [ ] Performance reviews
- [ ] QBR doc
- [ ] Project Alpha
- [ ] Project Beta

## Adhoc work to do:

- [ ] Review [team member]'s time off request

---

PRJ-1001 - Project Beta (https://{{ATLASSIAN_SITE}}/browse/PRJ-1001)

Last week: Team addressed ADR action items with Security Team alignment conversations. **This week, continuing Phase 1 implementation and finalizing rollout plan is expected.**

Progress this week:

Any fires

Any highlights

---

**Note:** To mark a task as complete, change `- [ ]` to `- [x]` and wrap the task text with `~~strikethrough~~` syntax. Example: `- [x] ~~Completed task~~`
```

## File Locations

- **Weekly Updates Folder:** `weekly-updates/` (organized by half-year subfolders: `1h2026`, `2h2025`, etc. H1 = Jan–Jun, H2 = Jul–Dec) — resolve from shared/config.md
- **all-projects.md:** `all-projects.md` — resolve from shared/config.md
