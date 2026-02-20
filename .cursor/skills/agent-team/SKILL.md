---
name: agent-team
description: Multi-agent review system that spawns specialist sub-agents for multi-perspective analysis. Agents share findings via filesystem, enabling cross-pollinated follow-up. Triggered naturally by the agentTeam.mdc rule when content warrants multi-perspective review.
---

# Agent Team Review

Orchestrates parallel specialist sub-agents for multi-perspective analysis. Agents communicate through shared findings on the filesystem, enabling cross-pollinated follow-up rounds.

## When This Activates

The agent team activates based on the `agentTeam.mdc` rule, which detects when content naturally warrants multi-perspective review. See that rule for trigger conditions, or the user can request it manually.

## Architecture

```
Workspace Root
└── .agent-team/
    ├── input.md                ← Source material being reviewed
    ├── security-findings.md    ← Security specialist output
    ├── engineering-findings.md ← Engineering specialist output
    ├── ops-findings.md         ← Ops specialist output
    ├── product-findings.md     ← Product manager specialist output
    └── synthesis.md            ← Final synthesized output
```

Agents communicate indirectly: each writes structured findings to their designated file. I read all findings between rounds and inject cross-references into follow-up prompts.

## Specialist Selection

Not every review needs all specialists. Select based on content:

| Content Type | Specialists |
|---|---|
| PRD / Product Spec | Product + Security + Engineering + Ops |
| Architecture / Design Doc | Engineering + Security |
| Launch Readiness | Ops + Security + Engineering + Product |
| Vendor / Tool Evaluation | Product + Security + Engineering |
| API Design | Security + Engineering |
| Incident / Post-mortem | Engineering + Ops |
| Infrastructure Change | Ops + Security |
| Feature Proposal / RFC | Product + Engineering + Security |
| Scope / Timeline Review | Product + Engineering |

Default to all four if unsure. For purely technical docs with no user-facing impact, skip Product.

## Workflow

### Step 0: Prepare Input

1. Identify the source material:
   - If the user points to a file → read it
   - If the user shares a Confluence page → fetch it
   - If the user shares a Jira ticket → fetch it
   - If the user pastes content → use directly
2. Create `.agent-team/` directory at workspace root
3. Write the source material to `.agent-team/input.md`
4. Determine which specialists are relevant (see Specialist Selection table)
5. Tell the user which specialists are being activated and why

### Step 1: Round 1 — Independent Analysis (Parallel)

Spawn specialist sub-agents in parallel via the Task tool (max 4, one per specialist). Each agent receives:

- The source material content (inline in the prompt, NOT a file reference — agents must have the full content)
- Their specialist profile (read from `specialists/{type}.md` and included inline)
- Instructions to write findings to the absolute path of `.agent-team/{type}-findings.md`
- The output format (from `templates.md`)

**Agent prompt structure:**

```
You are a {specialist_type} specialist conducting a review.

## Your Lens
{full content of specialists/{type}.md}

## Source Material to Review
{full content of .agent-team/input.md}

## Your Task
Analyze the source material through your specialist lens. Be specific — reference exact sections, quotes, or gaps. Flag severity levels.

## Output
Write your complete findings to {absolute_path}/.agent-team/{type}-findings.md

Use this exact format:
{template from templates.md — Specialist Findings Format section}
```

**Use `subagent_type: "generalPurpose"` for each specialist agent.** Do NOT use "explore" — specialists need full write access.

### Step 2: Resolution Loop — Cross-Specialist Conversation

After Round 1, specialists may have raised questions for each other (in their "Questions for Other Specialists" section). This step resolves those conversations.

#### 2a: Extract Unresolved Questions

1. Read all specialist findings files from `.agent-team/`
2. Collect every "Questions for Other Specialists" entry across all findings
3. Build a question map: `{target_specialist: [{question, asked_by, context}]}`
4. If the question map is empty → skip to Step 3 (synthesis)

#### 2b: Resolution Round

For each specialist that received questions, spawn a targeted follow-up agent. Only re-spawn specialists who were asked something — don't re-run everyone.

Each resolution agent receives:
- The questions directed at them (with full context from the asker's findings)
- Their own Round 1 findings (so they can refine)
- Other specialists' findings (for full picture)

**Resolution agent prompt structure:**

```
You are a {specialist_type} specialist in a multi-specialist review.
Other specialists have raised questions for you based on their review.

## Your Round 1 Findings
{their findings file content}

## Questions Directed at You

{For each question:}
### From {asking_specialist}:
> {question text}

Context from their findings:
{relevant excerpt from asker's findings}

## Your Task
1. Answer each question specifically and concisely
2. If your answers change any of your Round 1 findings, note the updates
3. If answering reveals NEW questions for other specialists, include them
4. Write your responses to {absolute_path}/.agent-team/{type}-resolution-r{round}.md
```

**Use `subagent_type: "generalPurpose"`.**

#### 2c: Check for New Questions

After resolution agents complete:
1. Read all resolution files
2. Check if any responses raised NEW cross-specialist questions
3. If yes → run another resolution round (2b) with only the newly-questioned specialists
4. If no → proceed to synthesis

#### 2d: Resolution Limits

- **Maximum 4 total rounds** (1 initial + up to 3 resolution rounds)
- If questions remain unresolved after 4 rounds, surface them as open questions in the synthesis
- Each resolution round should have fewer agents than the previous (converging, not expanding)
- If a resolution round produces MORE questions than the previous round, stop and surface — the review needs human input

### Step 3: Synthesis and Presentation

After all rounds complete, synthesize and present results. This step has two outputs: a **synthesis file** (detailed, for reference) and a **chat summary** (short, for immediate consumption).

#### 3a: Write Synthesis File

Write `.agent-team/synthesis.md` using the Synthesis File Format from `templates.md`. Key structure:

1. **Verdict** — 1-2 sentences: is the approach sound? What's the main concern?
2. **Top 3 Risks** — The highest-signal items, each with a short paragraph of context and a concrete action. Use blockquotes for emphasis (e.g., "> All three specialists flagged this independently").
3. **Cross-Cutting Concerns Table** — Compact table: #, Concern, Who Flagged, Severity. No long prose in table cells.
4. **Per-Specialist Summaries** — 4-5 bullet points per specialist. Headlines only — no paragraphs.
5. **Recommended Actions Table** — Compact table: #, Action, Owner.
6. **Open Questions** — Numbered list, one line each.

#### 3b: Present Chat Summary

In the chat message to the user, present ONLY:

1. **Verdict** (1-2 sentences)
2. **Top 3 risks** (3 short bullets with the core issue — no full explanations)
3. **Top action** (what to do first)
4. **Link to synthesis file** ("Full review written to `.agent-team/synthesis.md`")

**Readability rules (MANDATORY):**
- Never dump the full synthesis into the chat message
- Chat summary should be scannable in under 30 seconds
- Use the file for detail, use the chat for the headline
- If the user wants more detail, they can open the file or ask

### Step 5: Cleanup

`.agent-team/` contains working files, not permanent artifacts. Follow this cleanup policy:

#### After synthesis is presented
- Offer cleanup: "Want me to keep the detailed findings, or clean up?"
- If the user says keep → leave files in place
- If the user says clean → delete `.agent-team/` directory

#### Auto-clean (no need to ask)
- **The user acts on findings and moves on** — once the user has responded to the review (taken action, acknowledged, or shifted to a different topic), clean up silently on the next interaction. The synthesis captured everything worth keeping.
- **New team review starts** — always clean up the previous run's `.agent-team/` before writing new files. Never mix findings from separate reviews.

#### Do NOT clean up
- **Mid-conversation while findings are still being referenced** — if the user is drilling into findings, asking follow-up questions, or acting on specific items (e.g., "tell me more about finding #2", "update those Jira tickets"), keep the files until the conversation moves on.
- **The user explicitly said to keep them** — respect the override until the next review or until the user says otherwise.

## Critical Rules

- **Never skip the synthesis step.** Raw specialist output isn't useful on its own — the value is in cross-referencing.
- **Always tell the user which specialists are running and why.** No silent multi-agent spawning.
- **Include full content in agent prompts.** Agents cannot reliably read workspace files on their own — inline everything they need.
- **Run the resolution loop.** Don't skip straight to synthesis if specialists raised questions for each other. The cross-specialist conversation is where the real insight emerges.
- **Converge, don't expand.** Each resolution round should involve fewer agents and fewer questions than the previous. If it's expanding, stop and surface to the user.
- **Respect the round limit.** Maximum 4 total rounds (1 initial + 3 resolution). Surface unresolved items as open questions.
- **Use the right specialists.** Don't spawn ops for a pure API design review. Don't skip security for anything touching auth or data.
- **Chat is the headline, file is the detail.** Never dump the full synthesis into a chat message. Write `.agent-team/synthesis.md` for the full review, present a short scannable summary in chat.
- **Findings are working files, not permanent artifacts.** Offer cleanup after synthesis. Auto-clean when the user moves on or a new review starts. Never clean up mid-conversation while findings are still being referenced.

## Error Handling

- **Sub-agent failure**: If a specialist agent fails or returns empty, note which specialist couldn't complete and proceed with available findings. Don't block the whole review.
- **File write failure**: If an agent can't write to its findings file, capture the findings from the agent's return value and write the file manually.
- **Timeout**: If a specialist takes too long, proceed with available findings and note the gap in the synthesis.
- **Resolution loop stall**: If a resolution round produces no new answers (agents return empty or repeat prior content), stop the loop and proceed to synthesis.

## Validation Checklist

- [ ] Source material collected and written to `.agent-team/input.md`
- [ ] Appropriate specialists selected based on content type
- [ ] User informed which specialists are running
- [ ] Round 1 agents spawned in parallel and findings collected
- [ ] Cross-specialist questions extracted and resolution loop run (or no questions to resolve)
- [ ] Resolution converged (fewer questions each round) or hit round limit
- [ ] Synthesis file written to `.agent-team/synthesis.md` (readable, structured)
- [ ] Chat summary presented (verdict, top 3, top action, link to file)
- [ ] Cleanup offered (or auto-cleaned after conversation moved on)

See [templates.md](templates.md) for output format templates.
See [specialists/](specialists/) for individual specialist profiles.
