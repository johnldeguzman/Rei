# Agent Team Orchestration — Claude Code

Detailed workflow reference for the agent-team system. `CLAUDE.md` contains the trigger rules and specialist selection tables; this file contains the full procedures.

---

## Three Operating Modes

### 1. Full Team Review
Multi-specialist analysis for PRDs, architecture docs, launch readiness, etc. Triggered by CLAUDE.md auto-activate rules or manual request.

### 2. Embedded Specialist
A single specialist woven into an existing workflow (e.g., PM during end-week, engineer during technical planning). Triggered by CLAUDE.md embed tables.

### 3. Layered Research
Generalist-first investigation for cross-domain problems. Maps the full landscape before going deep. Triggered by research asks that span multiple systems.

---

## Agency — Think, Recommend, Act

Specialists don't just analyze and report. They have **agency**.

| Level | Description | Example |
|-------|-------------|---------|
| **Observe** | Identify what's happening | "3 of 5 projects slipped this week" |
| **Assess** | Explain why it matters | "Two slips are on the same dependency" |
| **Recommend** | Propose a specific action | "Escalate the shared dependency to [owner]" |
| **Act** | Execute the recommendation (with confirmation) | Update the file, draft the message |

Every specialist output must reach at least **Recommend**. Observations without recommendations are incomplete.

---

## Spawning Specialists in Claude Code

Use the **Agent tool** to spawn each specialist. Key parameters:

```
Agent tool call:
  description: "{specialist_type} specialist review"
  subagent_type: "general-purpose"
  model: "{see model table below}"
  prompt: "{constructed from profile + source material + instructions}"
  run_in_background: true  (for parallel spawning)
```

### Model Configuration

| Specialist | Model Parameter | Rationale |
|---|---|---|
| Engineering | `opus` | Deep technical reasoning, architecture analysis |
| Security | `opus` | Thorough threat modeling, compliance review |
| AI | `sonnet` | Prompt engineering, directive analysis |
| Product | `sonnet` | Product strategy, scope and delivery review |
| Ops | `sonnet` | Operational readiness, deployment assessment |

### Agent Prompt Structure (Full Team Review)

```
You are a {specialist_type} specialist conducting a review.

## Your Lens
{full content of .cursor/skills/agent-team/specialists/{type}.md}

## Source Material to Review
{full content of .agent-team/input.md}

## Tool Access — Verify, Don't Guess
You have full tool access: read files, run shell commands (read-only), search the codebase, and browse the workspace. USE THEM.

You also have GitHub CLI access (gh). The user is authenticated and has repo scope across the org. Use gh to verify claims against actual source code, configuration files, and repo structure.

Your profile includes a "Verification" section with specific checks for your domain. For every Critical or Warning finding, verify your claim using tools and include the evidence in the "Evidence" field.

## Your Task
Analyze the source material through your specialist lens. Be specific — reference exact sections, quotes, or gaps. Flag severity levels. Verify findings with tools wherever possible. If you identify an unknown that you have the expertise and tools to resolve, investigate it — don't leave it as an open question when you can answer it.

## Output
Write your complete findings to {absolute_path}/.agent-team/{type}-findings.md

Use this exact format:
{template from specialists/templates.md — Specialist Findings Format section}
```

---

## Full Team Review Workflow

### Step 0: Prepare Input

1. Identify the source material (file, Confluence page, Jira ticket, or pasted content)
2. **Clean slate check (MANDATORY):** If `.agent-team/` already exists, delete all files inside it before proceeding. Stale findings from a previous review will contaminate the resolution loop. Never mix findings from separate reviews.
3. Create `.agent-team/` directory at workspace root (or reuse the now-empty one)
4. Write source material to `.agent-team/input.md`
5. If input includes guiding questions, run the **framing quality check**:
   - **Architectural** concepts require architectural questions
   - **Tactical** concepts can use tactical questions directly
   - If mismatch, rewrite the questions
6. Determine which specialists are relevant (see Selection Table in CLAUDE.md)
7. Tell the user which specialists are being activated and why

### Step 1: Round 1 — Independent Analysis (Parallel)

Spawn specialist sub-agents in parallel via the Agent tool (max 4 at a time). **Run in background** — don't block waiting.

**Prompt construction (MANDATORY):** Before spawning, resolve all placeholders in the agent prompt template:
- Replace `{absolute_path}` with the actual workspace absolute path
- Replace `{type}` with the specialist type (e.g., `engineering`, `security`)
- Read `specialists/templates.md`, extract the "Specialist Findings Format" section, and substitute it into the prompt where the template placeholder appears
- Read `.cursor/skills/agent-team/specialists/{type}.md` and inline the full content into the prompt

Each agent receives:
- The source material content **inline in the prompt** (not a file reference)
- Their specialist profile (read from `.cursor/skills/agent-team/specialists/{type}.md` and included inline)
- Instructions to write findings to the **absolute path** `.agent-team/{type}-findings.md`
- The output format from `specialists/templates.md` (inlined, not referenced)

### Step 2: Resolution Loop — Cross-Specialist Conversation

After Round 1, specialists may have raised questions for each other.

#### 2a: Extract Unresolved Questions

1. Read all specialist findings files from `.agent-team/`
2. Collect every "Questions for Other Specialists" entry
3. Build a question map: `{target_specialist: [{question, asked_by, context}]}`
4. If the question map is empty → skip to Step 3
5. **STOP gate (MANDATORY):** Before proceeding, explicitly list every cross-specialist question found. Confirm:
   - Listed all questions from all findings
   - If non-empty, proceed to 2b — do not rationalize skipping

#### 2b: Resolution Round

For each specialist that received questions, spawn a targeted follow-up agent:

```
You are a {specialist_type} specialist in a multi-specialist review.
Other specialists have raised questions for you.

## Your Lens
{specialist profile content}

## Your Round 1 Findings
{their findings file content}

## Questions Directed at You

### From {asking_specialist}:
> {question text}

Context from their findings:
{relevant excerpt}

## Tool Access — Verify, Don't Guess
You have full tool access. If a question can be answered by checking the filesystem, running a command, or reading a file — do it.

## Your Task
1. Answer each question specifically and concisely — verify with tools where possible
2. If your answers change any Round 1 findings, note the updates
3. If answering reveals NEW questions for other specialists, include them
4. Write responses to {absolute_path}/.agent-team/{type}-resolution-r{round}.md
```

#### 2c: Check for New Questions

After resolution agents complete:
1. Read all resolution files
2. Check for NEW cross-specialist questions
3. If questions target an existing specialist → run another resolution round
4. If questions target a NEW specialist → ask the user before bringing them in
5. If no new questions → proceed to synthesis

#### 2d: Resolution Limits

- **Maximum 4 total rounds** (1 initial + up to 3 resolution rounds)
- Each round should have fewer agents than the previous (converging)
- If a round produces MORE questions than the previous, stop and surface to user
- New specialists added during resolution count toward round limit but not convergence check

### Step 3: Synthesis and Presentation

#### 3a: Write Synthesis File

Write `.agent-team/synthesis.md` using the format from `specialists/templates.md`:
1. **Verdict** — 1-2 sentences
2. **Top 3 Risks** — with blockquotes for cross-specialist agreement
3. **Cross-Cutting Concerns Table** — compact
4. **Per-Specialist Summaries** — bullet-only
5. **Recommended Actions Table** — compact
6. **Open Questions** — one line each

#### 3b: Present Chat Summary

In the chat, present ONLY:
1. Verdict (1-2 sentences)
2. Top 3 risks (3 short bullets)
3. Top action (what to do first)
4. Link to synthesis file

### Step 4: Execution Planning

#### 4a: Categorize Each Action

| Category | Criteria | Who Executes |
|----------|----------|--------------|
| **Delegate** | Isolated, well-scoped, fully describable | Worker agent (background) |
| **Do myself** | Requires session context, judgment, interdependent | Rei directly |
| **Needs input** | Can't proceed without user decision | User (ask first) |

#### 4b: Present Execution Plan

```
Execution plan:
- **I'll handle:** [list]
- **Delegating to workers:** [list]
- **Needs your input:** [list]

Proceed?
```

#### 4c: Dispatch and Execute

1. Dispatch worker agents as background tasks (max 4)
2. Execute own tasks while workers run
3. Check worker results after

#### 4d: Review Worker Output

1. Read each modified file and verify correctness
2. Check for unintended side effects
3. If worker output needs correction, fix directly
4. If worker failed, do the task myself

#### 4e: Present Results

```
Completed:
- [x] [action] — [who did it]
- [ ] [action] — needs your input: [question]
```

### Step 5: Cleanup

- **After synthesis:** Offer cleanup
- **Auto-clean:** When user moves on or new review starts
- **Don't clean:** Mid-conversation while findings are referenced, or user said to keep them

---

## Embedded Specialist Mode

### When to Use

| Workflow | Specialist | What they contribute |
|---------|-----------|---------------------|
| **end-week** | Product Manager | Assess progress, flag risks, recommend priority adjustments |
| **start-week** | Product Manager | Review priorities, flag sequencing issues, suggest focus areas |
| **timeline-sync** | Product Manager | Analyze timeline for conflicts, recommend adjustments |
| **jira-health-check** | Product Manager | Prioritize which hygiene issues matter vs. noise |
| **jira-project** (create) | Engineering | Review project structure, flag missing epics or phasing issues |
| **Technical implementation** | Engineering | Pre-flight: catch path issues, env assumptions, missing error handling |
| **Technical discussion** | Engineering | Feasibility, trade-offs, implementation approach |
| **Rule/skill/prompt changes** | AI | Review prompt structure, directive clarity, compliance patterns |

### Embedded Prompt Structure

```
You are a {specialist_type} specialist embedded in a {skill_name} workflow.

## Your Lens
{specialist profile content}

## Context
{relevant data from the skill}

## Tool Access — Verify, Don't Guess
You have full tool access: read files, run shell commands (read-only), search the codebase.

## Your Task
Review this context through your specialist lens. Provide:
1. 3-5 specific, actionable recommendations (not observations — actions)
2. For each: what to do, why it matters, and suggested priority (high/medium/low)
3. Flag anything that needs immediate attention vs. next-week items
4. For high-priority items, include evidence from tool verification
5. If you identify an unknown you can resolve, investigate it

Keep it concise. This feeds directly into the user's workflow output.
```

### Embedded Mode Rules

- **One specialist per skill invocation.** Multiple → escalate to full team review.
- **Lightweight.** 3-5 focused recommendations, not a comprehensive review.
- **Integrated output.** Recommendations appear inside the skill's normal output.
- **No `.agent-team/` directory.** Output goes directly into the skill's flow.
- **Escalation path.** If something needs multi-perspective analysis, suggest full team review.

---

## Layered Research Mode

### Architecture

```
Phase 1: Landscape    Phase 2: Deep Dives    Phase 3: Synthesis
                      ┌─ Domain A agent ─┐
Generalist agent ────►├─ Domain B agent ─┤────► Synthesize findings
                      └─ Domain C agent ─┘
```

### Phase 1: Landscape Mapping

Spawn a single generalist agent to go **wide, not deep**:

```
You are a generalist research analyst. Your job is NOT to solve the problem — it's to map the full landscape of where solutions might live.

## Problem Statement
{user's research question}

## Your Task — Map, Don't Solve
1. Identify all systems, platforms, and tools involved
2. For each, list solution categories with at least 2 specific named approaches
3. Assess each: Quick answer / Needs deep dive / Unknown
4. Recommend investigation areas with specific questions

## Rules
- MANDATORY: Perform at least one web search per system/domain
- Cast a WIDE net — avoid tunnel vision
- DO NOT go deep on any single area — that's Phase 2's job

## Output Format
### Problem Landscape
### Solution Map (per system/domain)
### Recommended Investigation Plan
### Explicitly Considered and Ruled Out (REQUIRED)
```

### Phase 2: Focused Deep Dives

Based on the landscape, spawn targeted research agents (max 4, parallel). Use specialist profiles when domain matches, general researchers otherwise.

**Prioritization when >4 areas:** (1) Unknown areas first, (2) Problem-statement areas, (3) Areas with dependencies. Defer the rest.

### Phase 3: Cross-Domain Synthesis

1. Read all deep-dive outputs
2. Cross-reference across domains
3. Build comparison matrix
4. Present top finding + options comparison + recommendation

### Layered Research Rules

- Generalist goes wide, specialists go deep
- Phase 2 agents don't overlap — clear domain boundaries
- Skip unnecessary depth for "quick answer" areas
- Web search is mandatory in Phase 1
- Findings don't write to `.agent-team/` — synthesized in conversation

---

## Specialist Review of Execution

For non-trivial actions, the specialist reviews execution before it's finalized.

**When to loop back:**

| Action type | Review needed? |
|-------------|---------------|
| Mechanical / data entry | No |
| Status write-up | Yes |
| Prioritization or sequencing | Yes |
| Architecture or technical decision | Yes |
| Risk assessment or communication | Yes |

**Review flow:**
1. Execute the recommendation and capture the result
2. Spawn the same specialist: "Here's how I executed your recommendation. Does this correctly reflect your intent?"
3. If corrections → apply and present corrected result
4. If approved → present final result to user

**Rules:**
- Review is a single pass — one check, not iterative
- Skip review for batches of mechanical actions

---

## Error Handling

- **Sub-agent failure**: Note which specialist couldn't complete, proceed with available findings
- **File write failure**: Capture findings from agent return value, write manually
- **Timeout**: Proceed with available findings, note the gap
- **Resolution loop stall**: If a round produces no new answers, stop and synthesize

---

## Validation Checklist

- [ ] Source material collected and written to `.agent-team/input.md`
- [ ] Appropriate specialists selected based on content type
- [ ] User informed which specialists are running
- [ ] Round 1 agents spawned in parallel and findings collected
- [ ] Cross-specialist questions extracted and resolution loop run (or no questions)
- [ ] Resolution converged or hit round limit
- [ ] Synthesis file written to `.agent-team/synthesis.md`
- [ ] Chat summary presented (verdict, top 3, top action, link to file)
- [ ] Execution plan presented if actions exist
- [ ] Worker output reviewed before presenting to user
- [ ] Cleanup offered after conversation moves on
