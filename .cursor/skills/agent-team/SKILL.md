---
name: agent-team
description: Multi-agent review system that spawns specialist sub-agents for multi-perspective analysis. Agents share findings via filesystem, enabling cross-pollinated follow-up. Triggered naturally by the agentTeam.mdc rule when content warrants multi-perspective review.
---

# Agent Team Review

Orchestrates parallel specialist sub-agents for multi-perspective analysis. Agents communicate through shared findings on the filesystem, enabling cross-pollinated follow-up rounds.

## When This Activates

The agent team operates in three modes:

1. **Full team review** — Multi-specialist analysis for PRDs, architecture docs, launch readiness, etc. Triggered by the `agentTeam.mdc` rule or manual request.
2. **Embedded specialist** — A single specialist is woven into an existing skill workflow (e.g., PM during end-week, engineer during technical planning). Triggered by the `agentTeam.mdc` skill-integration mapping.
3. **Layered research** — Generalist-first investigation for cross-domain problems. Maps the full landscape before going deep. Triggered by research asks that span multiple systems or have an unknown solution space.

## Agency — Think, Recommend, Act

Specialists don't just analyze and report. They have **agency**: the ability to propose concrete actions and drive outcomes.

### What agency means

| Level | Description | Example |
|-------|-------------|---------|
| **Observe** | Identify what's happening | "3 of 5 projects slipped this week" |
| **Assess** | Explain why it matters | "Two slips are on the same dependency — if it's not unblocked, next week is worse" |
| **Recommend** | Propose a specific action | "Escalate the shared dependency to [owner]. Defer Project C's milestone by 1 week to absorb the slip." |
| **Act** | Execute the recommendation (with confirmation) | Update the weekly file, draft the escalation message, adjust the timeline |

Every specialist output must reach at least **Recommend**. Observations without recommendations are incomplete.

### How this works in practice

- **Full team reviews**: The synthesis includes a **Recommended Actions** table with specific, assignable actions — not vague suggestions like "consider improving monitoring." Each action should say what to do, who should do it, and by when.
- **Embedded specialist**: The specialist's output is folded directly into the skill's output as recommendations. For example, during end-week the PM specialist doesn't produce a separate findings file — their insights appear as actionable suggestions in the weekly summary itself.

### Who executes

After analysis, execution is split based on the work:

- **Rei directly** — judgment calls, interdependent changes, anything requiring session context or behavioral rule edits.
- **Worker agents** — isolated, well-scoped, mechanical tasks that can be fully described in a prompt. Dispatched in parallel with write access. Output is reviewed by Rei before presenting to the user.
- **User** — decisions that require human judgment or external context.

The key principle: **analysis is read-only, execution is planned and delegated.** See Step 4 (Execution Planning) for the full workflow.

### Specialist review of execution

I have the tools but not the specialist's lens. For non-trivial actions, the specialist reviews my execution before it's finalized.

**When to loop back for review:**

| Action type | Review needed? | Example |
|-------------|---------------|---------|
| Mechanical / data entry | No | Update a Jira field, mark a task done, write a date |
| Status write-up | Yes | Weekly summary, project status narrative, escalation framing |
| Prioritization or sequencing | Yes | Reordering priorities, deferring scope, adjusting timelines |
| Architecture or technical decision | Yes | Proposing a design change, scoping technical work, phasing milestones |
| Risk assessment or communication | Yes | Framing a risk to stakeholders, drafting a go/no-go recommendation |

**Review flow:**

1. I execute the recommendation and capture the result (e.g., the updated file, the drafted message, the new timeline)
2. I spawn the same specialist with the execution output and ask: "Here's how I executed your recommendation. Does this correctly reflect your intent? Flag anything I got wrong or missed."
3. If the specialist flags corrections → I apply them and present the corrected result
4. If the specialist approves → I present the final result to the user

**Rules:**
- Review is a single pass — one check, not an iterative loop. If the specialist raises significant concerns, surface them to the user rather than auto-correcting repeatedly.
- The review agent gets: the original recommendation, my execution output, and the relevant context. Keep it focused.
- Skip review for batches of mechanical actions (e.g., updating 5 Jira fields). Apply review to the judgment calls within the batch, not every line item.

### Guardrails

- **Recommendations require reasoning.** No "you should do X" without explaining why.
- **Destructive actions need confirmation.** Anything that changes Jira, modifies files, or sends communications — always confirm with the user before executing.
- **Prioritize.** If there are 10 possible actions, highlight the top 1-3 that matter most. Don't overwhelm with a laundry list.

## Embedded Specialist Mode

Some workflows benefit from a specialist perspective without the overhead of a full multi-agent review. In embedded mode, a single specialist runs as part of an existing skill and contributes directly to that skill's output.

### When to use embedded mode

| Skill | Specialist | What they contribute |
|-------|-----------|---------------------|
| **end-week** | Product Manager | Assess week's progress against goals. Flag scope creep, stalled projects, delivery risks. Recommend priority adjustments for next week. |
| **start-week** | Product Manager | Review incoming priorities. Flag sequencing issues, missing dependencies, unrealistic commitments. Suggest focus areas. |
| **timeline-sync** | Product Manager | Analyze timeline for scheduling conflicts, unrealistic durations, missing buffers. Recommend adjustments. |
| **jira-health-check** | Product Manager | Interpret hygiene findings in project context. Prioritize which issues actually matter vs. noise. |
| **jira-project** (create) | Engineering | Review project structure, milestone breakdown, and technical scoping. Flag missing epics or unrealistic phasing. |
| **Technical implementation** (scripts, automation, infra, system changes) | Engineering | Review the implementation plan before execution. Catch environment assumptions, path issues, missing error handling, dependency risks, and integration gaps. Runs between plan and execute. |
| **Any technical discussion** | Engineering | When the conversation involves code, architecture, or technical decisions — bring engineering perspective on feasibility, trade-offs, and implementation approach. |
| **Code review** | Engineering | Review completed code against the plan, coding standards, and best practices. Uses `subagent_type: "code-reviewer"` instead of `generalPurpose`. |
| **Rule, skill, or prompt changes** | AI | Review prompt structure, directive clarity, compliance patterns, token efficiency, and discoverability. Catch ambiguity, redundancy, and behavioral edge cases before deploying. |

### How embedded mode works

1. The skill reads this mapping (or the `agentTeam.mdc` rule triggers it)
2. Spawn **one** specialist sub-agent with:
   - Their specialist profile (from `specialists/{type}.md`)
   - The relevant context from the skill (e.g., this week's data, the timeline, the Jira results)
   - A focused prompt: "You're embedded in [skill]. Here's the context. Provide 3-5 actionable recommendations."
3. The specialist's output is **integrated into the skill's output** — not written to a separate findings file
4. No synthesis step, no resolution loop — this is a lightweight consultation

### Embedded specialist prompt structure

```
You are a {specialist_type} specialist embedded in a {skill_name} workflow.

## Your Lens
{specialist profile content}

## Context
{relevant data from the skill — e.g., weekly progress, timeline, Jira results}

## Pre-Computed Metrics
{deterministic facts extracted from the context data — e.g., "12/15 tickets missing story points", "3 projects slipped this week". Omit if no quantifiable data.}

## Tool Access — Verify, Don't Guess
You have full tool access: read files, run shell commands (read-only), search the codebase. You also have **GitHub CLI access** (`gh`) with `repo` scope across the org — use it to verify against actual source code, not just the local workspace. Your profile includes a "Verification" section — use it. When making a claim about the state of something, check it first.

## Your Task
Review this context through your specialist lens. Provide:
1. 3-5 specific, actionable recommendations (not observations — actions)
2. For each: what to do, why it matters, and suggested priority (high/medium/low)
3. Flag anything that needs immediate attention vs. next-week items
4. For high-priority items, include evidence from tool verification where applicable
5. If you identify an unknown you have the expertise and tools to resolve, investigate it — don't leave it as an open question when you can answer it

Keep it concise. This feeds directly into the user's workflow output.
```

### Rules for embedded mode

- **One specialist per skill invocation.** Don't embed multiple specialists — if the situation warrants more, escalate to a full team review.
- **Lightweight, not exhaustive.** Embedded mode produces 3-5 focused recommendations, not a comprehensive review document.
- **Integrated output.** The specialist's recommendations appear inside the skill's normal output (e.g., in the weekly summary, not in a separate file).
- **No `.agent-team/` directory.** Embedded mode doesn't create findings files — the output goes directly into the skill's flow.
- **Escalation path.** If the embedded specialist surfaces something that needs deeper multi-perspective analysis, recommend a full team review: "This warrants a deeper look — want me to run the full team on it?"

---

## Layered Research Mode

For cross-domain problems where the solution space isn't known upfront. Instead of jumping straight to a domain specialist, start with a generalist who maps the full landscape, then go deep per domain, then synthesize.

### When to use layered research (vs. other modes)

| Signal | Use this mode |
|---|---|
| Problem spans multiple systems (e.g., Auth0 → Splunk, Kong → backend → Auth0) | Yes |
| Solution space is open-ended ("what are our options for X?") | Yes |
| Research ask where you don't know which domains contain the answer | Yes |
| "Investigate this", "look into this", "research this" + cross-domain | Yes |
| Single-domain, known system ("how do Auth0 Actions work?") | No — use embedded specialist |
| Reviewing an existing document for quality | No — use full team review |
| Quick factual question | No — answer directly |

### Architecture

```
Phase 1: Landscape    Phase 2: Deep Dives    Phase 3: Synthesis
                      ┌─ Domain A agent ─┐
Generalist agent ────►├─ Domain B agent ─┤────► Synthesize findings
                      └─ Domain C agent ─┘
```

### Workflow

#### Phase 1: Landscape Mapping

Spawn a single generalist agent whose job is to go **wide, not deep**. They map the full problem space without solving anything.

**Generalist agent prompt:**

```
You are a generalist research analyst. Your job is NOT to solve the problem — it's to map the full landscape of where solutions might live.

## Problem Statement
{user's research question or problem description, with full context from the conversation}

## Your Task — Map, Don't Solve

1. **Identify all systems, platforms, and tools involved** in this problem — both the ones explicitly mentioned and adjacent ones that might contain solutions. Think about the full data/workflow chain from end to end.

2. **For each system/domain, list solution categories** — what kinds of approaches exist on that side? Name them specifically (e.g., "Splunk Ingest Actions" not just "Splunk-side processing"). Use web search to discover options you're not already aware of.

3. **Assess each area:**
   - Quick answer possible? (Can be resolved with a web search or doc lookup)
   - Needs deep investigation? (Requires domain expertise, trade-off analysis, or verification)
   - Unknown? (Not sure if solutions exist here — flag for exploration)

4. **Recommend investigation areas** — for each area that needs a deep dive:
   - What specific questions should be answered?
   - What domain expertise is needed?
   - What tools/sources should be checked?

## Rules
- **MANDATORY: Perform at least one web search per system/domain before finalizing your output.** You are mapping what EXISTS, not only what you already know. Failure to search is the exact failure mode this workflow exists to prevent.
- Cast a WIDE net. The whole point is to avoid tunnel vision.
- Think about the full chain: source → transport → processing → destination. Solutions can live at any layer.
- For each system, list **at least 2 specific named solutions or approaches** — not generic categories.
- DO NOT go deep on any single area. If you find yourself writing more than 2-3 sentences about one solution, stop — that's Phase 2's job.

## Output Format

### Problem Landscape

**Systems involved:** [list all systems in the chain]

### Solution Map

For each system/domain:

#### {System/Domain Name}
- **Solution categories:** [list at least 2 specific named solutions/approaches per system]
- **Assessment:** Quick answer / Needs deep dive / Unknown
- **If deep dive needed:**
  - Questions to answer: [specific questions]
  - Expertise needed: [domain type]
  - Sources to check: [docs, repos, tools]

### Recommended Investigation Plan

[Ordered list of deep dives to run, prioritized by: (1) Unknown areas first, (2) Areas needing deep dive, (3) Explicit mentions in the problem statement. Group by domain.]

### Explicitly Considered and Ruled Out (REQUIRED)

[Systems or approaches you looked at but don't think are relevant, and why. This prevents Phase 2 from re-covering dead ends. If nothing was ruled out, state "N/A — all identified systems were relevant."]
```

**Use `subagent_type: "generalPurpose"`.** For the model: use `model: "fast"` when the problem has clearly named systems and a narrow scope. Use the default model (no `model` parameter) when the problem is vague, spans many domains, or has an unknown solution space — broader problems need stronger reasoning to avoid shallow mapping.

#### Phase 2: Focused Deep Dives

Based on the generalist's landscape map, spawn targeted research agents for each domain area that needs investigation. Run in parallel (max 4).

**Prioritization (when >4 areas need deep dives):** If the generalist recommends more than 4 investigation areas, prioritize by: (1) "Unknown" areas first — these are the biggest blind spots, (2) "Needs deep dive" areas explicitly mentioned in the problem statement, (3) Areas with dependencies on other areas. Defer remaining areas and tell the user what was deferred.

**Selecting deep-dive agents:**

| Generalist recommends | Agent to spawn |
|---|---|
| Area maps to an existing specialist domain (Auth0, security, ops) | Use that specialist profile |
| Area is a general technology domain (Splunk, Kong, AWS) | Use a general researcher with focused scope |
| Area flagged as "quick answer" | Skip agent — answer directly from the generalist's findings or a quick web search |

**Deep-dive agent prompt:**

```
You are a focused researcher investigating one domain area of a larger cross-domain problem.

## Problem Context
{original problem statement}

## Your Investigation Area
{specific domain/system from the generalist's map}

## Questions to Answer
{specific questions from the generalist's investigation plan}

## Full Investigation Plan (from landscape mapping)
{the generalist's complete Recommended Investigation Plan — so you see where your area fits in the bigger picture}

## What Other Domains Are Being Investigated
{list of other parallel investigations — so you know the boundaries of YOUR scope}

## What's Already Known
{any quick answers or ruled-out approaches from the generalist — don't re-cover these}

## Your Task
1. Answer the specific questions for your domain with depth and evidence
2. For each solution you find: explain how it works, what it covers, what it doesn't cover, and any caveats
3. Use web search and documentation to verify — don't speculate
4. If you discover solutions that cross into another domain's territory, note them but don't deep-dive — flag for synthesis

## Output Format

### {Domain} — Deep Dive Findings

**Solutions found:**

#### {Solution 1 Name}
- **How it works:** [concise explanation]
- **What it covers:** [specific to the original problem]
- **What it doesn't cover:** [gaps, limitations]
- **Caveats:** [gotchas, prerequisites, cost]
- **Evidence:** [links, docs, verified claims]

[Repeat for each solution]

**Cross-domain notes:** [anything that touches other investigation areas]

**Recommendation:** [which solution in this domain looks strongest and why]
```

**Use `subagent_type: "generalPurpose"`.** For agents using an existing specialist profile, use that profile's designated model. For general researchers, use the default model.

#### Phase 3: Cross-Domain Synthesis

After all deep dives complete, synthesize findings across domains. This is where the real value lives — solutions in one domain may eliminate the need for complexity in another.

1. Read all deep-dive outputs
2. Cross-reference: do different domains surface the same solution? Does a simple solution in domain B make a complex solution in domain A unnecessary?
3. Build a **comparison matrix** of all viable solutions across all domains
4. Present to the user:

```
**Research: {topic}**

**Landscape:** {N} domains investigated — {list}

**Top finding:** {the best approach and why — often the simplest one that was only visible because we looked across all domains}

**Options comparison:**

| Approach | Domain | Complexity | Coverage | Infrastructure |
|---|---|---|---|---|
| {option} | {which system} | {Low/Med/High} | {what it handles} | {what's needed} |

**Recommendation:** {which option, with reasoning that references the cross-domain view}

**Detailed findings saved to:** {location if written to file, or "available on request"}
```

### Rules for layered research

- **Generalist goes wide, specialists go deep.** Never let the generalist solve the problem — their job is mapping, not analysis.
- **Phase 2 agents don't overlap.** Each deep-dive has a clear domain boundary. If something crosses domains, flag it for synthesis.
- **Skip unnecessary depth.** If the generalist flags an area as "quick answer," don't spawn an agent — resolve it directly.
- **2 phases of depth is usually enough.** Only go to a third level if a deep-dive agent flags a sub-area that needs further investigation and the user confirms.
- **Web search is mandatory in Phase 1.** The generalist MUST search — the whole point is discovering what you don't already know. Failure to search is the exact failure mode this mode exists to prevent.
- **Present the landscape to the user before deep dives** if the problem is ambiguous. For clear problems, proceed directly.
- **Escalation to full team review.** If research findings produce a proposal or architecture that needs multi-perspective evaluation, suggest running a full team review on the output.

### Cleanup

Layered research doesn't write to `.agent-team/` by default — findings are synthesized directly in the conversation. If the user asks to save findings, write to `notes/{topic}-research.md`.

---

## Full Team Review

The full team review is the original multi-agent flow for deep analysis. Everything below applies to full team reviews.

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

## Model Configuration

Model selection depends on **how** the specialist is being used, not which specialist it is. Specialist profiles contain all domain knowledge (checklists, domain context, verification steps) — the model applies that knowledge.

### By mode

| Mode | Model | Rationale |
|---|---|---|
| **Embedded** (in a skill with pre-computed metrics — jira-health-check, timeline-sync, weekly update) | `fast` | Pre-computed metrics do the heavy lifting (Step 2b). The specialist interprets numbers and recommends actions — a structured task. |
| **Full team review** (PRD, architecture doc, RFC, launch readiness) | default (high) | Reasoning from raw material about scope, trade-offs, security implications, and cross-domain interactions. |
| **Synthesis** (cross-specialist resolution and final synthesis) | default (high) | Cross-specialist reasoning, conflict resolution, priority ranking. |
| **Single specialist — structured content** (rule/skill review, code review) | `fast` | Applying a known checklist to well-structured input. |
| **Single specialist — open-ended content** (ad-hoc analysis, ambiguous scope) | default (high) | Needs reasoning about trade-offs without a pre-defined checklist. |

### Quick decision rule

**If the specialist has pre-computed metrics or is applying a checklist to structured data → `fast`.** If the specialist is reasoning from raw, ambiguous material → default (high).

When spawning specialist agents, set the `model` parameter based on the mode, not the specialist type. The specialist's profile declares `**Model:** context-dependent` — this table is the source of truth for which context gets which model.

### Subagent Type Routing

All specialists use `subagent_type: "generalPurpose"` by default. Engineering has a context-dependent override:

| Context | Subagent Type | When |
|---|---|---|
| Full team review, embedded analysis, technical discussion | `generalPurpose` | Default — needs write access for findings files and full tool access for verification |
| Code review | `code-reviewer` | When reviewing completed code against a plan or coding standards (e.g., "review this code", "code review", post-implementation review) |

The specialist's profile declares its available subagent types. When spawning, match the context to the correct type.

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
| Code Review | Engineering (`code-reviewer`) |
| Rule / Skill / Prompt Design | AI + Engineering |
| Team / Process / Staffing Change | EM + Product |
| Cross-Team Collaboration Model | EM + Engineering |

Default to all specialists if unsure. For purely technical docs with no user-facing impact, skip Product. For non-team/process topics, skip EM.

## Workflow

### Step 0: Prepare Input

1. Identify the source material:
   - If the user points to a file → read it
   - If the user shares a Confluence page → fetch it
   - If the user shares a Jira ticket → fetch it
   - If the user pastes content → use directly
2. Create `.agent-team/` directory at workspace root
3. Write the source material to `.agent-team/input.md`. If the input includes guiding questions for specialists, run the **framing quality check** before finalizing:
   - Do the questions match the ambition level of the source concept?
   - **Architectural** concepts (new structures, systems, patterns, approaches) require architectural questions — "what would this look like?", "design this system", "how should this work?" Do not ask tactical questions ("which existing flags to flip", "what to move where") until the architectural design is established.
   - **Tactical** concepts (refactors, config changes, flag toggles) can use tactical questions directly.
   - If the source proposes a new design but the questions only ask about tweaking existing mechanisms, rewrite the questions.
4. Determine which specialists are relevant (see Specialist Selection table)
5. **Pre-compute metrics** (see Step 0b below) — if the source material contains quantifiable data, extract key metrics before sending to specialists
6. Tell the user which specialists are being activated and why

### Step 0b: Pre-Compute Metrics (When Applicable)

For content types that contain quantifiable data, compute deterministic metrics before passing to specialists. This ensures specialists argue about *interpretation and priority*, not about *what the numbers say*.

**When to pre-compute:** If the source material or the context around it includes structured data (timelines, ticket counts, resource allocations, status distributions, health metrics), extract the key numbers.

**How:**
1. Read the relevant data sources (Jira query results, project files, timeline data, health check output)
2. Compute summary statistics — totals, distributions, ratios, thresholds exceeded
3. Add a `## Pre-Computed Metrics` section to the specialist prompt (between the source material and the task instructions)

**Pre-computed metrics prompt block:**

```
## Pre-Computed Metrics

These are deterministic facts extracted from the source data. Reference them in your analysis — don't re-derive them.

{metrics content — e.g.:}
- Total projects: 8 (4 active, 2 planned, 2 backlog)
- Tickets missing dates: 12/47 (25%) — 3 projects, 9 epics
- Overdue items: 5 (oldest: 23 days past due)
- Resource allocation: 3 devs at >90% utilization, 1 dev at 45%
- Timeline span: 8 months, 4 milestones, 2 with no end date
- Status misalignment: 2 projects show "In Progress" but all children are "Backlog"
```

**What to compute per content type:**

| Content Type | Metrics to Extract |
|---|---|
| Jira health check results | Ticket counts by check (stale, missing dates, overdue, empty epics, misaligned), severity distribution, days-overdue histogram |
| Project timeline | Total duration, milestone count, milestones without dates, resource count, dependency depth, projects by status |
| Weekly update / end-week | Tasks completed vs planned, projects that slipped, blockers count, carryover items |
| PRD / feature spec | Feature count, integration points, external dependencies count, scope items without acceptance criteria |
| Architecture doc | Components count, external systems touched, new infrastructure required, migration steps |

**Rules:**
- Only pre-compute what the data supports — don't fabricate metrics for qualitative content
- Keep it to 5-10 bullet points — this is a summary, not a report
- Use the same metrics for all specialists in a given review — consistency matters
- For full team reviews, include the metrics in every specialist's prompt. For embedded mode, include in the embedded specialist's prompt.

### Step 1: Round 1 — Independent Analysis (Non-Blocking, Parallel)

Spawn specialist sub-agents in parallel via the Task tool (max 4, one per specialist). **Analysis agents run in the background** — do not block waiting for them. Continue preparing for synthesis (e.g., reading related files, checking workspace state) while they work. Collect results when all agents complete.

Each agent receives:

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

## Pre-Computed Metrics
{metrics from Step 0b, or omit this section if no quantifiable data was available}

## Tool Access — Verify, Don't Guess
You have full tool access: read files, run shell commands (read-only), search the codebase, and browse the workspace. USE THEM.

You also have **GitHub CLI access** (`gh`). The user is authenticated and has `repo` scope across the org. Use `gh` to verify claims against actual source code, configuration files, and repo structure — don't limit yourself to the local workspace.

Your profile includes a "Verification" section with specific checks for your domain. For every Critical or Warning finding, verify your claim using tools and include the evidence in the "Evidence" field. Don't speculate when you can check.

Examples:
- Checking a path exists: run `ls -la /path/to/thing` and include the output
- Checking a binary is installed: run `which binary-name`
- Checking config correctness: read the actual file and quote the relevant section
- Checking project state: read `all-projects.md` or query workspace files
- Checking source code on GitHub: run `gh search code "pattern" --owner={{GITHUB_ORG}}` or `gh api repos/{owner}/{repo}/contents/{path} --jq '.content' | base64 -d`
- Listing repos or files: run `gh search repos "keyword" --owner={{GITHUB_ORG}}` or `gh api repos/{owner}/{repo}/git/trees/main --jq '.tree[].path'`

If you cannot verify something (no access, remote dependency, needs runtime testing), say "Unverified: [reason]" in the Evidence field.

## Your Task
Analyze the source material through your specialist lens. Be specific — reference exact sections, quotes, or gaps. Flag severity levels. Verify findings with tools wherever possible. If you identify an unknown that you have the expertise and tools to resolve, investigate it and include your findings — don't leave it as an open question when you can answer it.

## Output
Write your complete findings to {absolute_path}/.agent-team/{type}-findings.md

Use this exact format:
{template from templates.md — Specialist Findings Format section}
```

**Use `subagent_type: "generalPurpose"` for each specialist agent.** Do NOT use "explore" — specialists need full write access and tool access for verification. **Set the `model` parameter per the Model Configuration table** — full team reviews use the default (high) model; embedded specialists use `fast`.

### Step 2: Resolution Loop — Cross-Specialist Conversation

After Round 1, specialists may have raised questions for each other (in their "Questions for Other Specialists" section). This step resolves those conversations.

#### 2a: Extract Unresolved Questions

1. Read all specialist findings files from `.agent-team/`
2. Collect every "Questions for Other Specialists" entry across all findings
3. Build a question map: `{target_specialist: [{question, asked_by, context}]}`
4. If the question map is empty → skip to Step 3 (synthesis)
5. **STOP gate (MANDATORY):** Before proceeding to Step 3, explicitly list every cross-specialist question found. Confirm:
   - [ ] Listed all questions from all findings
   - [ ] If the list is non-empty, proceed to Step 2b — do not rationalize skipping ("the answer is obvious", "synthesis will cover it", "it's a minor question", "running 2b would be redundant"). Even a short "see my Round 1 finding" reply counts as running 2b.

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

## Your Lens
{full content of specialists/{type}.md}

## Your Round 1 Findings
{their findings file content}

## Questions Directed at You

{For each question:}
### From {asking_specialist}:
> {question text}

Context from their findings:
{relevant excerpt from asker's findings}

## Tool Access — Verify, Don't Guess
You have full tool access. If a question can be answered by checking the filesystem, running a command, or reading a file — do it. Include evidence in your answers.

## Your Task
1. Answer each question specifically and concisely — verify with tools where possible
2. If your answers change any of your Round 1 findings, note the updates
3. If answering reveals NEW questions for other specialists, include them
4. Write your responses to {absolute_path}/.agent-team/{type}-resolution-r{round}.md
```

**Use `subagent_type: "generalPurpose"`.**

#### 2c: Check for New Questions

After resolution agents complete:
1. Read all resolution files
2. Check if any responses raised NEW cross-specialist questions
3. **If questions target an existing specialist** → run another resolution round (2b) with only the newly-questioned specialists
4. **If questions target a NEW specialist not in the current review** → ask the user: "Resolution raised questions for [specialist] who wasn't part of this review. Want me to bring them in?" Only spawn the new specialist if the user approves.
5. If no new questions → proceed to synthesis

#### 2d: Resolution Limits

- **Maximum 4 total rounds** (1 initial + up to 3 resolution rounds)
- If questions remain unresolved after 4 rounds, surface them as open questions in the synthesis
- Each resolution round should have fewer agents than the previous (converging, not expanding)
- If a resolution round produces MORE questions than the previous round, stop and surface — the review needs human input
- **New specialists added during resolution** count toward the round limit but not the convergence check (they're additive by design)

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

### Step 4: Execution Planning

After synthesis, if the review produced actionable recommendations, plan who executes what before proceeding.

#### 4a: Categorize Each Action

For each recommended action from the synthesis, classify it:

| Category | Criteria | Who Executes |
|----------|----------|--------------|
| **Delegate** | Isolated, well-scoped, fully describable without session context. No dependencies on other actions. | Worker agent (write access) |
| **Do myself** | Requires session context, involves judgment calls, interdependent with other changes, or affects my own behavioral rules. | Rei directly |
| **Needs input** | Can't proceed without a user decision or clarification. | User (ask first) |

#### 4b: Present Execution Plan

Present the plan to the user before executing:

```
Execution plan:
- **I'll handle:** [list — judgment calls, interdependent changes]
- **Delegating to workers:** [list — isolated, mechanical fixes]
- **Needs your input:** [list — decisions only you can make]

Proceed?
```

#### 4c: Dispatch and Execute (Parallel)

1. **Dispatch worker agents as background tasks** (max 4) for delegated tasks. Each worker gets:
   - The specific file(s) to edit
   - The exact change to make (before/after or clear instructions)
   - Relevant context from the synthesis (not the full review — just what they need)
   - `readonly: false` — workers have write access
   - `subagent_type: "generalPurpose"`
   - Workers run in the background — do not block waiting for them
2. **Execute own tasks immediately** while workers run in parallel
3. **Check worker results** after own tasks complete — read modified files to verify changes

#### 4d: Review Worker Output

After workers complete:
1. Read each modified file and verify the change is correct
2. Check for unintended side effects (broken references, formatting issues)
3. If a worker's output needs correction, fix it directly — don't re-dispatch
4. If a worker failed, do the task myself and note it

#### 4e: Present Results

Summarize what was done:
```
Completed:
- [x] [action] — [who did it]
- [x] [action] — [who did it]
- [ ] [action] — needs your input: [question]
```

#### Execution planning rules

- **Always present the plan before executing.** No silent delegation.
- **Workers get minimal context.** Only what's needed for their specific task — not the full synthesis or session history.
- **Review is mandatory.** Never present worker output to the user without checking it first.
- **Fallback is me.** If a worker fails or produces bad output, I do the task myself. Don't re-dispatch.
- **Skip this step for embedded mode.** Embedded specialists are lightweight consultations — execution planning is for full team reviews only.

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

### Agency
- **Every specialist output must include actionable recommendations.** Observations without "here's what to do about it" are incomplete.
- **Recommendations must be specific and assignable.** "Improve monitoring" is not actionable. "Add latency alerts on the auth endpoint with a 500ms P99 threshold" is.
- **Confirm before acting.** Specialists can recommend destructive or external actions, but I always confirm with the user before executing.

### Full team reviews
- **Never skip the synthesis step.** Raw specialist output isn't useful on its own — the value is in cross-referencing.
- **Always tell the user which specialists are running and why.** No silent multi-agent spawning.
- **Include full content in agent prompts.** Agents cannot reliably read workspace files on their own — inline everything they need.
- **Run the resolution loop.** Don't skip straight to synthesis if specialists raised questions for each other. The cross-specialist conversation is where the real insight emerges.
- **Converge, don't expand.** Each resolution round should involve fewer agents and fewer questions than the previous. If it's expanding, stop and surface to the user.
- **Respect the round limit.** Maximum 4 total rounds (1 initial + 3 resolution). Surface unresolved items as open questions.
- **Use the right specialists.** Don't spawn ops for a pure API design review. Don't skip security for anything touching auth or data.
- **Chat is the headline, file is the detail.** Never dump the full synthesis into a chat message. Write `.agent-team/synthesis.md` for the full review, present a short scannable summary in chat.
- **Findings are working files, not permanent artifacts.** Offer cleanup after synthesis. Auto-clean when the user moves on or a new review starts. Never clean up mid-conversation while findings are still being referenced.

### Embedded mode
- **One specialist, focused output.** Embedded mode is a lightweight consultation — 3-5 recommendations, not a full review.
- **Integrated, not separate.** Output folds into the skill's normal flow. No findings files, no synthesis step.
- **Escalate when warranted.** If the embedded specialist finds something that needs multi-perspective analysis, suggest a full team review.

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
- [ ] Execution plan presented and approved by user (if actions exist)
- [ ] Worker agents dispatched for delegated tasks (readonly: false)
- [ ] Worker output reviewed before presenting to user
- [ ] Results summary presented with completion status
- [ ] Cleanup offered (or auto-cleaned after conversation moved on)

See [templates.md](templates.md) for output format templates.
See [specialists/](specialists/) for individual specialist profiles.
