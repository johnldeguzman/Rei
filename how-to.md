# Project Management Workflows - How To Guide

**Confluence Reference:** [Cursor Workflow](https://everlong.atlassian.net/wiki/spaces/~5fff5e574d2179006e08ac02/pages/6485869260/Cursor+Workflow) (Page ID: `6485869260`)

This document outlines all the automated workflows available in the Project Management system. These workflows help you manage projects, track weekly progress, and maintain synchronization between Jira, Confluence, and local tracking files.

## Table of Contents

1. [Overview](#overview)
2. [Start Week Workflow](#start-week-workflow)
3. [End Week Workflow](#end-week-workflow)
4. [Create Jira Project Workflow](#create-jira-project-workflow)
5. [Done Projects Management](#done-projects-management)
6. [Memory Management](#memory-management)
7. [Atlassian/MCP Error Handling](#atlassianmcp-error-handling)
8. [File Structure](#file-structure)
9. [Best Practices](#best-practices)

---

## Overview

The Project Management system consists of six main workflows:

1. **Start Week** - Initialize a new weekly tracking file with priorities
2. **End Week** - Summarize the week, create executive summaries, and sync with project tracking
3. **Create Jira Project** - Generate a complete Jira project hierarchy with tickets
4. **Done Projects Management** - Automatically move completed projects to a Done Projects section with contributor information
5. **Memory Management** - Persist preferences, people context, and key decisions across sessions
6. **Atlassian/MCP Error Handling** - Automatically handles authentication errors from Atlassian or MCP tools

All workflows are triggered by natural language commands and guide you through each step.

---

## Start Week Workflow

**Trigger:** Say "start my week" or "start week"

### Purpose
Creates a new weekly tracking file for the current week with your top 4 priorities.

### Step-by-Step Process

#### Step 1: Calculate Dates
- System calculates today's date and the upcoming Friday
- Format: `MMMDDYYYY-MMMDDYYYY` (e.g., `Jan152025-Jan192025.md`)

#### Step 2: Check Previous Week Status
- **MANDATORY**: System checks if the previous week was properly ended
- If previous week wasn't ended, you'll see a warning:
  ```
  WARNING: The previous week's file has not been marked as ended.
  
  Do you want to:
  1. End the previous week first (recommended) - type "end previous week"
  2. Start a new week anyway - type "start anyway"
  3. Cancel - type "cancel"
  ```
- If previous week had priorities, they'll be displayed to help you remember

#### Step 3: Create weekly-updates Folder
- System ensures the "weekly-updates" folder exists
- Creates backup of previous week's file if it exists

#### Step 4: Set Priorities (MANDATORY)
- **System asks:** "What are your top priorities for this week? Please list them (I'll help you narrow to 4 if needed):"
- If you provide more than 4 priorities:
  - System displays them numbered
  - **System asks:** "You've provided [X] priorities. Please select the top 4 by telling me which numbers to keep (e.g., '1, 2, 3, 4'):"
- System confirms: "Are these your top 4 priorities for the week? (yes/no)"

#### Step 5: Copy Projects from Previous Week (MANDATORY)
- **MANDATORY**: System copies all existing projects from the previous week's file
- For each project:
  - Projects are copied with their project names and links
  - **MANDATORY**: For projects that are NOT done (check Jira status), add a small summary written from the perspective of what should be done THIS week
  - Format: `Last week: [What was completed last week]. **This week, [what is expected this week] is expected.**` (placed BEFORE "Progress this week:" section)
  - Bold the "This week" portion for better readability
  - Transform phrases like "are next" or "queued next" to "This week, [action] is expected"
  - Then add "Progress this week:" section below (leave empty for new entries)
  - If project is done, only include "Progress this week:" section (leave empty)
- This ensures continuity - you don't lose track of ongoing projects and provides context for the new week
- If no previous week exists, file starts with template project entries

#### Step 6: Create Weekly File
- Creates new file: `weekly-updates/[today]-[friday].md`
- File structure includes:
  - Top 4 priorities at the top (formatted as checkboxes)
  - Adhoc work section (formatted as checkboxes)
  - All projects from previous week (with empty progress sections)
  - "Any fires" and "Any highlights" sections at the end
  - Note at the bottom explaining how to mark tasks as complete with strikethrough
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
  
  [Previous Week Project 1] (with link)
  
  Last week: [What was completed]. **This week, [what is expected] is expected.**
  
  Progress this week:
  
  [Previous Week Project 2] (with link)
  
  Progress this week:
  
  Any fires
  
  Any highlights
  
  ---
  
  **Note:** To mark a task as complete, change `- [ ]` to `- [x]` and wrap the task text with `~~strikethrough~~` syntax. Example: `- [x] ~~Completed task~~`
  ```

### Example

**You:** "start my week"

**System:** 
- Checks previous week
- "What are your top priorities for this week?"

**You:** "1. Performance reviews, 2. QBR doc, 3. Medibank EAP, 4. CACT IAM, 5. Another task"

**System:** "You've provided 5 priorities. Please select the top 4..."

**You:** "1, 2, 3, 4"

**System:** Creates file with your 4 priorities at the top

---

## End Week Workflow

**Trigger:** Say "end my week" or "end week"

### Purpose
Summarizes the week, creates executive summaries for stakeholders, syncs with project tracking files, and optionally syncs to Confluence.

### Step-by-Step Process

#### Step 1: Identify Current Week File
- System locates the weekly file for the current week
- Format: `weekly-updates/[monday]-[friday].md`

#### Step 2: Read and Parse weekly-updates
- System reads the entire weekly file
- Extracts:
  - Project identifiers (e.g., ID-5024)
  - Progress notes
  - Fires (urgent issues)
  - Accomplishments

#### Step 3: Create Executive Summaries
- For each project, creates an executive summary:
  ```markdown
  ## [Project Name] - Week of [Monday] - [Friday]
  
  **Current Status:** [Summary of progress]
  
  **Fires:**
  - [List fires - ONLY if fires exist in weekly update]
  
  **Accomplishments:**
  - [List achievements - ONLY if accomplishments exist]
  ```
- **Important:** Fires and Accomplishments sections are only included if they have actual content
- Summaries are added to the weekly file under "Weekly Executive Summary" section

#### Step 4: Ask About Next Week Priorities (NEW)
- **System asks:** "Do you want to add priorities for next week? (yes/no)"
- If yes:
  - **System asks:** "Please list the priorities for next week (you can provide them as a list):"
  - Adds "Priorities for Next Week" section to the weekly file
  - Section is placed before the "Week Ended" marker

#### Step 5: Sync with all-projects.md
- Updates `all-projects.md` with project status updates
- **Important:** Only syncs status/project updates - NOT fires or accomplishments
- Includes full technical detail from weekly updates
- Format:
  ```markdown
  **Week of [Monday] - [Friday]:**
  - **Status:** [Executive summary - project update/status only]
  ```

#### Step 6: Verify Jira Ticket Statuses
- System checks current Jira statuses for:
  - Project tickets (if they exist)
  - Epic tickets
- Updates `all-projects.md` with current statuses
- **Status Calculation Rules:**
  - If project ticket exists: Use its status from Jira
  - If no project ticket: Calculate from epics
    - Project is "Done" ONLY if ALL epics are "Done"
    - If ANY epic is "In Progress" or "Backlog", project status is "In Progress"
    - Status progression: Backlog → In Progress → Done

#### Step 7: Sync to Confluence (Optional)
- **System asks:** "Do you want to sync all-projects.md to the Confluence page? (yes/no)"
- Confluence page: `https://everlong.atlassian.net/wiki/spaces/~5fff5e574d2179006e08ac02/pages/6476038314/All+Projects`
- If yes, updates Confluence page with content from `all-projects.md`

#### Step 7.6: Working Memory Review + Compaction
- Reads SOUL.md Working Memory section
- Flags entries older than 3 months or contradicted by current workspace state
- Flags entries now covered by rules or SOUL core sections
- If Working Memory exceeds 100 lines, recommends aggressive trimming
- Proposes removals, updates, or consolidations
- Applies only changes you approve
- Skipped if Working Memory is empty

#### Step 8: Mark Week as Ended
- Adds metadata marker to weekly file:
  ```markdown
  ---
  **Week Ended:** [Current Date/Time]
  **Week Status:** Completed
  ---
  ```

#### Step 9: Generate Final Report
- Presents final summary
- Confirms all updates are complete

### Example

**You:** "end my week"

**System:**
- Reads weekly file
- Creates executive summaries
- "Do you want to add priorities for next week? (yes/no)"

**You:** "yes"

**System:** "Please list the priorities for next week:"

**You:** "1. Draft performance reviews, 2. QBR doc, 3. Medibank EAP scoping, 4. CACT IAM planning"

**System:**
- Adds priorities to weekly file
- Updates all-projects.md
- Verifies Jira statuses
- "Do you want to sync all-projects.md to Confluence? (yes/no)"

**You:** "yes"

**System:** Syncs to Confluence and marks week as ended

---

## Create Jira Project Workflow

**Trigger:** Say "create a project" or request project creation in Jira

### Purpose
Creates a complete Jira project hierarchy: Project → Milestones → Epics → Technical Tasks, with optional Confluence page creation and all-projects.md updates.

### Hierarchy Structure

```
Project (Level 3)
  └── Milestone (Level 2)
      └── Epic (Level 1)
          └── Technical Task (Level 0)
```

### Step-by-Step Process

#### Step 0: Project ID & Milestone Selection (MANDATORY)

**1. Ask for Project ID**
- **System asks:** "What is the Project ID where this Project Ticket type should be created?"
- System can search for Project ID if you provide a project name or key
- **MANDATORY:** Cannot proceed without a valid Project ID

**2. Ask for Milestone Selection**
- **System asks:** "Do you need to create Discovery, Delivery, and Rollout milestones? (yes/no)"
- If "yes": Plans all three milestone types
- If "no": Asks which specific milestones you want

#### Step 1: Visual Planning & Approval (MANDATORY)

**1. Generate Visual Diagram**
- System creates a visual diagram showing the complete project structure
- Format: ASCII tree showing Project → Milestones → Epics → Technical Tasks
- Example:
  ```
  Project: Wiz Platform Evaluation (WIZPLAT)
  
  ├── Discovery Phase (Discovery Milestone)
  │     ├── Epic: Current State & Requirements Analysis
  │     │     ├── Technical Task: Inventory all current scanners
  │     │     └── Technical Task: Review documentation
  │     └── Epic: Wiz Platform Feature Assessment
  │           └── Technical Task: Assess Wiz UVM features
  
  ├── Integration & Testing (Delivery Milestone)
  │     └── Epic: Wiz UVM Proof of Concept
  │           └── Technical Task: Set up test environment
  
  └── Rollout & Recommendations (Rollout Milestone)
        └── Epic: Final Evaluation & Handover
              └── Technical Task: Finalize report
  ```

**2. Wait for Approval**
- **System says:** "Review the planned structure above. Type 'create' to generate tickets, or provide feedback for changes."
- **MANDATORY:** System will NOT create tickets until you type "create" or "yes"
- If you provide feedback, system revises the diagram and waits again

#### Step 2: Generate Tickets After Approval

**1. Create Project Ticket**
- Issue Type: `Project` (hierarchyLevel: 3)
- Naming Format: `[Project Name] ([ABBREVIATION])`
- Example: `Wiz Platform Evaluation (WIZPLAT)`
- Uses Project ID from Step 0

**2. Create Milestone Tickets**
- Issue Types:
  - `Discovery Milestone` - For research, analysis, requirements gathering
  - `Delivery Milestone` - For implementation, proof of concept, integration
  - `Rollout Milestone` - For final evaluation, handover, recommendations
- Naming Format: `[Phase Name] ([Milestone Type])`
- Examples:
  - `Discovery Phase (Discovery Milestone)`
  - `Integration & Testing (Delivery Milestone)`
  - `Rollout & Recommendations (Rollout Milestone)`
- Only creates milestones you approved in Step 0

**3. Create Epic Tickets**
- Issue Type: `Epic` (hierarchyLevel: 1)
- Naming Format: `[Epic Name]` (descriptive theme/work area)
- Examples:
  - `Current State & Requirements Analysis`
  - `Wiz UVM Proof of Concept`
- **MANDATORY RULE:** If Discovery Milestone exists, always creates at least one "Discovery" epic
- 2-4 Epics per Milestone

**4. Create Technical Task Tickets**
- Issue Type: `Technical Task` (hierarchyLevel: 0)
- Naming Format: `[Action Verb] [Subject/Object] [Context/Goal]`
- Examples:
  - `Inventory all current scanners and document integration capabilities`
  - `Review Wiz UVM documentation for scanner integration`
  - `Set up Wiz UVM test environment`
- 3-10 Tasks per Epic
- Action verbs: Inventory, Review, Identify, Document, Assess, Evaluate, Analyze, Set up, Integrate, Configure, Test, Validate, Compare, Collect, Finalize, Present, Plan

#### Step 3: Confluence Page Creation (MANDATORY)

**After Project ticket is created:**
- **System asks:** "Do you need to create a Confluence page for this project? (yes/no)"

**If yes:**
- **System asks:** "Which section should this page be in: 'Technical Discovery', 'Active', or 'Done'?"
- Maps to parent page IDs:
  - Technical Discovery → Page ID `3646914716`
  - Active → Page ID `3388866587`
  - Done → User specifies or uses default
- Creates Confluence page with:
  - Title: Same as Project ticket title
  - Parent: Selected parent page
  - Space: `ID` (Identity space)
  - Body: Project details + JQL query for finding epics

#### Step 4: Update all-projects.md (MANDATORY)

**After Project ticket is created:**
- System adds project entry to `/Users/jdeguzman/Desktop/Project Management/all-projects.md`
- Format:
  ```markdown
  ## [Project Title]
  
  **Confluence URL:** [URL]
  [ONLY included if Confluence page was created]
  
  **Jira Project Ticket:** [[PROJECT-KEY] - [Project Name]](https://everlong.atlassian.net/browse/[PROJECT-KEY])
  
  **Status:** [Status]
  [Fetched from Jira]
  
  **Jira Epic Tickets:**
  - [[EPIC-KEY-1] - [Epic Name]](https://everlong.atlassian.net/browse/[EPIC-KEY-1]) (Status: [Status])
  - [[EPIC-KEY-2] - [Epic Name]](https://everlong.atlassian.net/browse/[EPIC-KEY-2]) (Status: [Status])
  
  ---
  ```

### Naming Conventions

#### Project
- Format: `[Project Name] ([ABBREVIATION])`
- Example: `Wiz Platform Evaluation (WIZPLAT)`

#### Milestones
- Format: `[Phase Name] ([Milestone Type])`
- Examples:
  - `Discovery Phase (Discovery Milestone)`
  - `Integration & Testing (Delivery Milestone)`
  - `Rollout & Recommendations (Rollout Milestone)`

#### Epics
- Format: `[Epic Name]`
- Examples:
  - `Current State & Requirements Analysis`
  - `Wiz Platform Feature Assessment`
  - `Wiz UVM Proof of Concept`

#### Technical Tasks
- Format: `[Action Verb] [Subject/Object] [Context/Goal]`
- Examples:
  - `Inventory all current scanners and document integration capabilities`
  - `Review Wiz UVM documentation for scanner integration`
  - `Set up Wiz UVM test environment`

### Action Verb Reference

**Discovery/Research Tasks:**
- `Inventory` - List and catalog existing items
- `Review` - Examine existing documentation or systems
- `Identify` - Find and name specific items
- `Document` - Record information in written form
- `Assess` - Evaluate or judge a situation
- `Evaluate` - Determine the value or quality of something
- `Analyze` - Examine in detail

**Implementation/Integration Tasks:**
- `Set up` - Configure and prepare environment
- `Integrate` - Connect systems together
- `Configure` - Set up settings or parameters
- `Test` - Verify functionality
- `Validate` - Confirm correctness
- `Compare` - Examine differences between items

**Analysis/Reporting Tasks:**
- `Collect` - Gather information
- `Finalize` - Complete and finalize documents
- `Present` - Share findings with stakeholders
- `Plan` - Create strategy or roadmap

### Example

**You:** "create a project for Wiz Platform Evaluation"

**System:** "What is the Project ID where this Project Ticket type should be created?"

**You:** "PRD"

**System:** "Do you need to create Discovery, Delivery, and Rollout milestones? (yes/no)"

**You:** "yes"

**System:** [Creates visual diagram]

**System:** "Review the planned structure above. Type 'create' to generate tickets, or provide feedback for changes."

**You:** "create"

**System:**
- Creates Project ticket: `Wiz Platform Evaluation (WIZPLAT)`
- Creates Milestone tickets
- Creates Epic tickets
- Creates Technical Task tickets
- "Do you need to create a Confluence page for this project? (yes/no)"

**You:** "yes"

**System:** "Which section should this page be in: 'Technical Discovery', 'Active', or 'Done'?"

**You:** "Technical Discovery"

**System:**
- Creates Confluence page
- Updates all-projects.md
- Confirms completion

---

## Memory Management

**Trigger:** Say "remember that", "save that", "note that for next time", or Rei will proactively offer to log decisions during project conversations.

### Purpose
Persists durable knowledge across sessions — preferences, people context, and key decisions — so Rei doesn't start from zero every conversation.

### Where Memory Lives

| Location | What it stores | When to use |
|---|---|---|
| **SOUL.md Working Memory → Preferences** | Recurring preferences you've explicitly stated | "I prefer async standups", "Don't schedule calls before 11am" |
| **SOUL.md Working Memory → People & Context** | Key people, their roles, working preferences | "Sarah handles EAP integration, prefers Slack" |
| **SOUL.md Working Memory → Key Decisions** | Durable behavioral decisions | "We sequence projects, not parallelize" |
| **IMPROVEMENTS.md Decisions Log** | Project/technical decisions with detailed reasoning | "Chose Redis over Memcached because..." |

### How It Works

#### Explicit saves
Say "remember that", "save that", or "note that for next time" and Rei will write the information to Working Memory with proper attribution and date.

#### Proactive decision capture
During project conversations, Rei actively recognizes when decisions are being made — choosing between options, resolving open questions, agreeing on scope changes. At a natural pause, Rei offers to log them:
- "That's a decision worth logging — [summary]. Want me to save it?"
- "A few decisions came out of this conversation: [list]. Want me to log them?"

You can decline — no repeated prompting.

#### Confirmation for preferences
If Rei notices what seems like a durable preference, it confirms before writing: "That sounds like a lasting preference — want me to save it to Working Memory?"

### What Does NOT Go in Memory

- **Project status, priorities, or dates** → `all-projects.md` and weekly updates own these
- **Actionable rule/skill fixes** → IMPROVEMENTS.md Proposed section
- **Temporary context** — debugging sessions, one-off questions, transient tasks
- **Anything already captured elsewhere** — no duplication

### Precedence

When sources conflict:
1. **Your current statement** — always wins
2. **Workspace state files** — all-projects.md, weekly updates, project-timeline.md
3. **SOUL.md Working Memory** — for preferences, people, behavioral decisions
4. **IMPROVEMENTS.md Decisions Log** — for past reasoning

### Compaction

Working Memory is reviewed during **end-week** (Step 7.6):
- Entries older than 3 months are flagged for review
- Entries contradicted by current workspace state are flagged
- Entries now covered by rules or SOUL core sections are flagged
- You approve all removals/updates before Rei writes

Working Memory is capped at ~100 lines to keep SOUL.md lean.

### Example

**You:** "Let's go with the phased rollout approach for the migration instead of big-bang"

**Rei:** *[continues working on the current topic]*

**Rei:** "A decision came out of this conversation: Migration uses phased rollout, not big-bang. Want me to log it?"

**You:** "yes"

**Rei:** Logs to IMPROVEMENTS.md Decisions Log with date, reasoning, and project context.

---

## Atlassian/MCP Error Handling

**Trigger:** Automatically runs when authentication errors occur from Atlassian or MCP tools

### Purpose
Handles authentication errors (401 Unauthorized, 403 Forbidden, etc.) by guiding you to re-enable authentication before retrying operations.

### When It Triggers
- Any authentication-related error from Atlassian API calls
- Any authentication-related error from MCP tool calls
- Errors containing: `401 Unauthorized`, `403 Forbidden`, `Authentication failed`, `Unauthorized`, or similar authentication keywords

### What Happens

1. **System Detects Error**
   - Identifies authentication-related errors
   - Checks if error is from Atlassian or MCP tools

2. **Displays Instructions (IN ALL CAPS)**
   - System displays:
     ```
     ⚠️ AUTHENTICATION ERROR DETECTED ⚠️
     
     THE ATLASSIAN/MCP TOOL AUTHENTICATION HAS EXPIRED OR FAILED.
     
     PLEASE FOLLOW THESE STEPS:
     1. TOGGLE THE ATLASSIAN/MCP TOOL OFF AND THEN BACK ON TO RE-ENABLE AUTHENTICATION
     2. WAIT FOR AUTHENTICATION TO COMPLETE
     3. LET ME KNOW WHEN YOU ARE READY TO TRY AGAIN
     
     ONCE YOU'VE TOGGLED THE TOOL AND RE-AUTHENTICATED, TYPE "ready" OR "try again" AND I WILL RETRY THE OPERATION.
     ```

3. **Waits for Confirmation**
   - System does NOT automatically retry
   - System waits for you to type "ready", "try again", "retry", or similar confirmation

4. **Retries After Confirmation**
   - Once you confirm, system retries the original operation
   - If error persists, process repeats

### Important Notes

- **MANDATORY**: Instructions are always displayed in ALL CAPS for visibility
- **MANDATORY**: System never automatically retries without your confirmation
- **MANDATORY**: You must manually toggle the tool to refresh authentication tokens
- This prevents repeated failed attempts and ensures proper authentication

### Example

**Scenario:** Atlassian API call fails with 401 Unauthorized

**System:**
```
⚠️ AUTHENTICATION ERROR DETECTED ⚠️

THE ATLASSIAN/MCP TOOL AUTHENTICATION HAS EXPIRED OR FAILED.

PLEASE FOLLOW THESE STEPS:
1. TOGGLE THE ATLASSIAN/MCP TOOL OFF AND THEN BACK ON TO RE-ENABLE AUTHENTICATION
2. WAIT FOR AUTHENTICATION TO COMPLETE
3. LET ME KNOW WHEN YOU ARE READY TO TRY AGAIN

ONCE YOU'VE TOGGLED THE TOOL AND RE-AUTHENTICATED, TYPE "ready" OR "try again" AND I WILL RETRY THE OPERATION.
```

**You:** "ready"

**System:** Retries the original operation

---

## Done Projects Management

**Trigger:** Automatically runs when a project status changes to "Done" or when reviewing/updating all-projects.md

### Purpose
Manages completed projects by moving them from the active projects section to a dedicated "Done Projects" section in all-projects.md, preserving historical information and tracking contributors.

### Step-by-Step Process

#### Step 1: Identify Done Projects
- System checks all-projects.md for projects with status "Done ✅"
- Reviews recent weekly updates for completed projects
- Notes completion dates if available

#### Step 2: Gather Contributor Information
- System checks Jira tickets for assignee information
- Reviews weekly updates and project notes for contributor mentions
- **If contributor information is missing or incomplete:**
  - **System asks:** "Who worked on [Project Name]? Please provide the names of the people who contributed to this project."
  - Waits for your response before proceeding
- Formats contributor list (e.g., "Emmanuel Ezeh, John De Guzman")

#### Step 3: Create/Update Done Projects Section
- Checks if "Done Projects" section exists in all-projects.md
- If it doesn't exist, creates it at the end of the file
- Moves done projects from active section to Done Projects section

#### Step 4: Update Active Projects Section
- Removes done projects from the active projects section
- Maintains proper formatting and section order

#### Step 5: Sync to Confluence (MANDATORY)
- **MANDATORY**: Automatically syncs updated all-projects.md to Confluence
- Confluence page: `https://everlong.atlassian.net/wiki/spaces/~5fff5e574d2179006e08ac02/pages/6476038314/All+Projects`
- Page ID: `6476038314`
- Ensures the Confluence reference page stays up-to-date

### Done Projects Format

```markdown
## Done Projects

### [Project Name]

**Confluence URL:** [URL if exists]
[ONLY included if Confluence page exists]

**Jira Project Ticket:** [[PROJECT-KEY] - [Project Name]](https://everlong.atlassian.net/browse/[PROJECT-KEY])

**Completed:** [MMM DD, YYYY]

**Contributors:** [Name1, Name2, Name3]

**Jira Epic Tickets:**
- [[EPIC-KEY] - [Epic Name]](https://everlong.atlassian.net/browse/[EPIC-KEY]) (Status: Done ✅)

**Key Identity Tickets (if applicable):**
- [[TICKET-KEY] - [Ticket Name]](https://everlong.atlassian.net/browse/[TICKET-KEY]) (Status: Done ✅)

**Final Status Summary:**
[Final project status and summary]

---
```

### Example

**Scenario:** BCBSWY project is marked as Done ✅

**System:**
1. Identifies BCBSWY as done with completion date "Dec 01, 2025"
2. Checks Jira: Finds Emmanuel Ezeh assigned to key tickets
3. **Asks:** "Who worked on BCBSWY (BCBS Wyoming)? Please provide the names of the people who contributed to this project."

**You:** "Emmanuel Ezeh"

**System:**
- Moves BCBSWY from active projects to Done Projects section
- Adds completion date, contributor, and final status summary
- Syncs updated all-projects.md to Confluence
- Confirms successful completion

### Important Notes

- **MANDATORY**: Always asks for contributor information if not clear from Jira tickets
- **MANDATORY**: Never moves a project to Done Projects without completion date and contributor information
- **MANDATORY**: Always syncs all-projects.md to Confluence after moving a project to Done Projects
- Done projects preserve all historical information and ticket links
- The Done Projects section serves as an archive of completed work

---

## File Structure

### weekly-updates Folder
**Location:** `/Users/jdeguzman/Desktop/Project Management/weekly-updates/`

**File Format:** `[MMMDDYYYY]-[MMMDDYYYY].md`
- Example: `Jan152025-Jan192025.md`
- Format: Month abbreviation (3 letters, capitalized) + Day (no leading zero) + Year (4 digits)

**File Structure:**
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

[Project Name/Identifier]

Last week: [What was completed]. **This week, [what is expected] is expected.**

Progress this week:
[Weekly progress notes]

[Another Project]

Progress this week:
[Weekly progress notes]

Any fires
[Urgent issues/problems]

Any highlights
[Notable achievements]

---

**Note:** To mark a task as complete, change `- [ ]` to `- [x]` and wrap the task text with `~~strikethrough~~` syntax. Example: `- [x] ~~Completed task~~`
```

### all-projects.md
**Location:** `/Users/jdeguzman/Desktop/Project Management/all-projects.md`

**Purpose:** Central tracking file for all projects (active and completed)

**Active Projects Format:**
```markdown
## [Project Title]

**Confluence URL:** [URL]
[ONLY included if Confluence page exists]

**Jira Project Ticket:** [[PROJECT-KEY] - [Project Name]](https://everlong.atlassian.net/browse/[PROJECT-KEY])

**Status:** [Status]

**Jira Epic Tickets:**
- [[EPIC-KEY-1] - [Epic Name]](https://everlong.atlassian.net/browse/[EPIC-KEY-1]) (Status: [Status])
- [[EPIC-KEY-2] - [Epic Name]](https://everlong.atlassian.net/browse/[EPIC-KEY-2]) (Status: [Status])

**Week of [Monday] - [Friday]:**
- **Status:** [Executive summary - project update/status only]

---
```

**Done Projects Format:**
```markdown
## Done Projects

### [Project Name]

**Confluence URL:** [URL if exists]

**Jira Project Ticket:** [[PROJECT-KEY] - [Project Name]](https://everlong.atlassian.net/browse/[PROJECT-KEY])

**Completed:** [MMM DD, YYYY]

**Contributors:** [Name1, Name2, Name3]

**Jira Epic Tickets:**
- [[EPIC-KEY] - [Epic Name]](https://everlong.atlassian.net/browse/[EPIC-KEY]) (Status: Done ✅)

**Final Status Summary:**
[Final project status and summary]

---
```

### Confluence Pages

**all-projects.md Confluence Page:**
- **Location:** `https://everlong.atlassian.net/wiki/spaces/~5fff5e574d2179006e08ac02/pages/6476038314/All+Projects`
- **Page ID:** `6476038314`
- **Purpose:** Synced version of all-projects.md for team visibility

**how-to.md Confluence Page:**
- **Location:** `https://everlong.atlassian.net/wiki/spaces/~5fff5e574d2179006e08ac02/pages/6485869260/Cursor+Workflow`
- **Page ID:** `6485869260`
- **Purpose:** Synced version of how-to.md for team reference

---

## Best Practices

### Weekly Workflow

1. **Start your week early:** Run "start my week" on Monday morning to set priorities
2. **Update throughout the week:** Add project progress, fires, and highlights as they happen
3. **End your week properly:** Always run "end my week" on Friday to:
   - Create executive summaries
   - Sync with all-projects.md
   - Set priorities for next week
   - Mark the week as completed

### Project Creation

1. **Plan before creating:** Think about the project structure before triggering the workflow
2. **Review the visual diagram:** Always review the proposed structure before approving
3. **Use descriptive names:** Follow naming conventions for clarity
4. **Create Confluence pages:** Always create Confluence pages for better documentation
5. **Verify statuses:** Check that all-projects.md reflects current Jira statuses

### File Management

1. **Don't skip weeks:** Always end your week before starting a new one
2. **Keep priorities focused:** Limit to 4 top priorities per week
3. **Include technical details:** When updating weekly files, include full technical context
4. **Sync regularly:** Sync all-projects.md to Confluence regularly for team visibility

### Status Management

1. **Status calculation rules:**
   - If project ticket exists: Use its status from Jira
   - If no project ticket: Calculate from epics
     - Project is "Done" ONLY if ALL epics are "Done"
     - If ANY epic is "In Progress" or "Backlog", project status is "In Progress"
2. **Always verify:** System verifies Jira statuses when ending the week
3. **Update discrepancies:** System automatically updates status discrepancies
4. **Done projects:** When a project is marked as Done, system automatically moves it to Done Projects section with contributor information and syncs to Confluence

---

## Quick Reference

### Commands

- **Start Week:** "start my week" or "start week"
- **End Week:** "end my week" or "end week"
- **Create Project:** "create a project" or "create project"
- **Save to Memory:** "remember that", "save that", "note that for next time"
- **Log Decision:** "log that decision" (or Rei offers proactively during project conversations)

### File Locations

- **weekly-updates:** `/Users/jdeguzman/Desktop/Project Management/weekly-updates/`
- **all-projects.md:** `/Users/jdeguzman/Desktop/Project Management/all-projects.md`
- **Confluence:** `https://everlong.atlassian.net/wiki/spaces/~5fff5e574d2179006e08ac02/pages/6476038314/All+Projects`

### Important Rules

1. **MANDATORY:** Always end your week before starting a new one
2. **MANDATORY:** Set exactly 4 priorities when starting your week
3. **MANDATORY:** Review visual diagram before creating Jira tickets
4. **MANDATORY:** Provide Project ID before creating Jira projects
5. **MANDATORY:** Only include Fires/Accomplishments in executive summaries if they have content
6. **MANDATORY:** When a project is marked as Done, provide contributor information when asked
7. **MANDATORY:** Done projects are automatically synced to Confluence
8. **MANDATORY:** When authentication errors occur, toggle the Atlassian/MCP tool off and back on before retrying

---

## Troubleshooting

### Previous Week Not Ended
**Problem:** System warns that previous week wasn't ended

**Solution:** 
- Choose "end previous week" to run the end week workflow first
- Or choose "start anyway" if you want to proceed (not recommended)

### Too Many Priorities
**Problem:** You provided more than 4 priorities

**Solution:** System will ask you to select the top 4 by number

### Missing Project ID
**Problem:** System asks for Project ID when creating a project

**Solution:** Provide the Project ID or project name/key (system can search)

### Status Discrepancies
**Problem:** System finds status discrepancies between all-projects.md and Jira

**Solution:** System automatically updates all-projects.md with current Jira statuses

---

## Summary

These workflows automate:
- ✅ Weekly project tracking with priorities (checkbox format)
- ✅ Executive summary generation
- ✅ Jira project hierarchy creation
- ✅ Confluence page creation
- ✅ Status synchronization
- ✅ Project tracking across multiple systems
- ✅ Done projects management with contributor tracking
- ✅ Automatic Confluence synchronization for completed projects
- ✅ Authentication error handling for Atlassian/MCP tools

All workflows are designed to be interactive and guide you through each step. Simply use natural language commands to trigger them, and the system will handle the rest!

