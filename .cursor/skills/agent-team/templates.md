# Agent Team Templates

## Specialist Findings Format

Each specialist writes findings to `.agent-team/{type}-findings.md` using this format:

```markdown
# {Type} Review — {Subject}

**Reviewed:** {YYYY-MM-DD}
**Specialist:** {Security | Engineering | Ops}
**Input:** {brief description of what was reviewed}

---

## Executive Summary

{2-3 sentences: overall assessment from this specialist's perspective}

## Findings

### 🔴 Critical

- **Finding**: {what the issue is}
- **Where**: {section/component/area of the source material}
- **Evidence**: {what you checked to verify — command output, file contents, or "Unverified: [reason]"}
- **Why it matters**: {impact if not addressed}
- **Recommendation**: {specific action}

{repeat for each critical finding, or "None" if none}

### 🟡 Warning

- **Finding**: {what the issue is}
- **Where**: {section/component/area of the source material}
- **Evidence**: {what you checked to verify — command output, file contents, or "Unverified: [reason]"}
- **Why it matters**: {impact if not addressed}
- **Recommendation**: {specific action}

{repeat for each warning, or "None" if none}

### 🔵 Notes

- **Finding**: {what the issue is}
- **Where**: {section/component/area of the source material}
- **Why it matters**: {impact if not addressed}
- **Recommendation**: {specific action}

{repeat for each note, or "None" if none. Evidence field optional for Notes — include if you verified something.}

## Questions for Other Specialists

- **For Engineering:** {question or concern — or "None"}
- **For Ops:** {question or concern — or "None"}
- **For Security:** {question or concern — or "None"}

## Open Questions

{Things that need human input or further investigation — or "None"}
```

---

## Resolution Response Format

When a specialist is re-spawned to answer questions from other specialists, they write to `.agent-team/{type}-resolution-r{round}.md`:

```markdown
# {Type} Resolution — Round {N}

**Date:** {YYYY-MM-DD}
**Specialist:** {Security | Engineering | Ops}
**Responding to:** {list of specialists who asked questions}

---

## Responses

### To {Asking Specialist}: {question summary}

**Answer:** {direct, specific answer}

**Impact on my findings:** {did this change any of my Round 1 findings? If yes, what changed and why. If no, say "No changes."}

{repeat for each question}

---

## Updated Findings (if any)

{Only include if answers changed previous findings. Reference the original finding and state the update.}

## New Questions for Other Specialists

{Only include if answering revealed new questions. If none, say "None — all cross-references resolved."}

- **For {Specialist}:** {new question}
```

---

## Synthesis File Format

Written to `.agent-team/synthesis.md`. Optimized for scannability — short sections, compact tables, no walls of text.

```markdown
# Team Review: {Subject}

**Specialists:** {list} | **Rounds:** {N} | **Date:** {YYYY-MM-DD}

---

## Verdict

{1-2 sentences. Is the approach sound? What's the main concern?}

---

## Top 3 Risks

### 1. {Risk title}

{2-3 sentences of context — what it is, why it matters.}

> {Quote highlighting cross-specialist agreement, e.g., "All three specialists flagged this independently."}

**Action:** {One concrete sentence — what to do.}

---

### 2. {Risk title}

{Same structure}

---

### 3. {Risk title}

{Same structure}

---

## Cross-Cutting Concerns

Items flagged by 2+ specialists.

| # | Concern | Who Flagged | Severity |
|---|---------|------------|----------|
| 1 | {short description} | {specialists} | {emoji} |

{Keep table cells SHORT — one line each. No paragraphs in tables.}

---

## By Specialist

### {Specialist 1} — {X critical, Y warnings}

Key concerns:
- **{Headline}** — {one sentence}
- **{Headline}** — {one sentence}
- **{Headline}** — {one sentence}
- **{Headline}** — {one sentence}

### {Specialist 2} — {X critical, Y warnings}

{Same structure}

### {Specialist 3} — {X critical, Y warnings}

{Same structure}

---

## Recommended Actions

| # | Action | Owner |
|---|--------|-------|
| 1 | **{action}** — {why, one phrase} | {role} |
| 2 | **{action}** — {why, one phrase} | {role} |

---

## Open Questions

1. {One-line question}
2. {One-line question}

---

*Full specialist findings: [security-findings.md](security-findings.md) | [engineering-findings.md](engineering-findings.md) | [ops-findings.md](ops-findings.md)*
```

### Formatting Rules (MANDATORY)

1. **Verdict first.** Lead with the bottom line — don't make the user read 3 pages to get the answer.
2. **Top 3 Risks use blockquotes** for cross-specialist agreement signals.
3. **Tables are compact.** One line per cell. No multi-sentence explanations inside table cells.
4. **Per-specialist sections are bullet-only.** Bold headline + one sentence. No paragraphs.
5. **Horizontal rules (`---`) between major sections** for visual separation.
6. **Recommended Actions is a table**, not a numbered prose list.
7. **Open Questions are one line each.** If it needs more context, it belongs in the specialist findings, not here.

---

## Chat Summary Format

Presented in the chat message (NOT in a file). Must be scannable in under 30 seconds.

```
**Verdict:** {1-2 sentences}

**Top 3 risks:**
1. **{Risk}** — {one sentence}
2. **{Risk}** — {one sentence}
3. **{Risk}** — {one sentence}

**Do first:** {one concrete action}

Full review written to `.agent-team/synthesis.md` — specialist deep-dives in the same directory.
```

**Rules:**
- Never paste the full synthesis into chat
- No tables in the chat summary
- No per-specialist breakdowns in chat
- If the user wants detail, they can open the file or ask

---

## Example: Cross-Cutting Concern Table

| # | Concern | Who Flagged | Severity |
|---|---------|------------|----------|
| 1 | No rate limiting on token endpoint | Security, Engineering | 🔴 |
| 2 | No rollback plan for DB migration | Ops, Engineering | 🟡 |
| 3 | Token storage location unspecified | Security, Ops | 🟡 |
