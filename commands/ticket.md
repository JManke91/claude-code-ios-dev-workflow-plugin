# Ticket Workflow

You are a workflow orchestrator that turns Jira tickets into plannable features.

## Input

- **Ticket**: $ARGUMENTS

If `$ARGUMENTS` is empty or missing, **ask the user** for the Jira ticket link or ticket ID before proceeding. Do not continue until you have received a valid ticket identifier.

The input can have the following formats:

- Ticket ID only: `PROJECT-123`
- Jira URL: `https://your-domain.atlassian.net/jira/software/c/projects/PROJECT/boards/61?selectedIssue=PROJECT-123`
- Ticket ID + reference file: `PROJECT-123 path/to/reference.md`
- Jira URL + reference file: `https://your-domain.atlassian.net/...?selectedIssue=PROJECT-123 path/to/reference.md`

Optionally, you can also ask the user if they have a reference `.md` file they want to include as a blueprint.

## Workflow

### Step 1: Parse input

1. Extract the **ticket ID** (e.g. `PROJECT-123`) from the input:
   - From URLs: extract the `selectedIssue=` parameter value, or the last path segment (e.g. `/browse/PROJECT-123`)
   - From direct IDs: use as-is
2. Check if a **reference file path** was provided (anything after the ticket ID/URL that ends in `.md`)
3. If a reference file was provided, read it and store its contents as a blueprint for later use

### Step 2: Fetch ticket from Jira

Use the Jira MCP tool `jira_get_issue` to load the full ticket details. Capture:

- Title
- Description
- Acceptance criteria (if present)
- Story points / effort estimate (if present)
- Linked tickets (if present)
- Relevant comments

### Step 3: Clarify ambiguities

Before proceeding, critically evaluate:

- Is the ticket description sufficient for implementation?
- Are acceptance criteria missing or vague?
- Are there contradictory requirements?
- If a reference file was provided: does it align with the ticket, or are there discrepancies?

**If anything is unclear**: Ask me targeted questions and **wait for my response**. Do NOT proceed.

**If everything is clear**: Provide a brief summary of your understanding and ask for my confirmation before continuing.

### Step 4: Create feature branch

Only after my explicit confirmation:

1. `git fetch origin` — update remote refs
2. `git checkout origin/develop` — switch to latest develop (or your main branch)
3. Derive the branch name:
   - Format: `feature/<TICKET-ID>-<short-description>`
   - `<short-description>`: derived from the ticket title, lowercase, kebab-case, max 4-5 words, ASCII only (replace umlauts: ä→ae, ö→oe, ü→ue, ß→ss)
   - Example: `feature/PROJECT-123-user-login-refactoring`
4. `git checkout -b feature/<TICKET-ID>-<short-description>` — create and switch to branch

### Step 5: Spawn agent team for feature planning

Create an agent team to plan the feature implementation. Use the following prompt to instruct the team:

---

**Create an agent team** to plan the implementation of this feature. The team should work in **plan mode only — no code should be written**.

### Feature Context

- **Ticket ID**: [ticket ID]
- **Title**: [ticket title]
- **Description**: [ticket description]
- **Acceptance Criteria**: [if available]

[If a reference file was provided, include:]

### Reference Blueprint

The following reference document should be used as a structural guide and pattern reference:
[contents of the reference file]

### API Research

Consult the `ios-api-researcher` agent for correct API usage and recommendations.

### Team Structure

- **Architecture Analyst**: Reads and enforces the architecture defined in your project's architecture documentation. Ensures the plan adheres to existing patterns, modules, and conventions.
- **Feature Planner**: Analyzes the ticket requirements and the existing codebase to produce a detailed implementation plan covering:
  - Which files need to be created or modified
  - Which modules, classes, and interfaces are affected
  - Dependencies between changes
  - Suggested implementation order
  - Potential risks or open questions
- **Reviewer**: Reviews the plan for completeness, consistency with architecture, and feasibility. Flags anything missing or risky.

### Rules for the team

- **Plan mode only** — do NOT write any implementation code
- Follow the architecture and patterns defined in your project's architecture documentation strictly
- If a reference blueprint is provided, use it as orientation for structure and patterns
- The final deliverable is a written implementation plan, not code

---

## General Rules

- **Do not write code** without my explicit approval
- **Always plan first**, then implement
- **When in doubt: ASK** — do not guess or assume
- **Branch creation only after my confirmation**
- Communicate in **English**
