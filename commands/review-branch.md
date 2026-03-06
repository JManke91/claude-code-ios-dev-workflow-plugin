# Code Review: Current Branch

Review the code changes in the current branch against the Jira ticket requirements and project standards. Produce a comprehensive review document suitable for GitHub PR feedback.

## Step 1: Gather Context

1. **Ask the user** for the Jira ticket URL or ticket number (e.g. `PROJECT-123` or a full Jira URL).
2. Once provided, use the **Jira MCP** to fetch the ticket details — specifically:
   - Summary / title
   - Description
   - Acceptance criteria (check the description, acceptance criteria field, or subtasks)
   - **Comments** — fetch all comments on the ticket (use `get_comments` or equivalent). These may contain QA feedback, reviewer notes, or clarifications from the team that are relevant to the review.
   - Any linked tickets or context
3. **Fetch the latest base branch** before computing the diff:
   ```
   git fetch origin develop
   ```
   (Adjust `develop` to your main branch if different)
4. Identify the **current branch** and the **diff against the base branch**:
   ```
   git diff $(git merge-base HEAD origin/develop)..HEAD
   ```
   Also collect the list of changed files:
   ```
   git diff --name-only $(git merge-base HEAD origin/develop)..HEAD
   ```

## Step 2: Spawn Agent Team for Parallel Review

Create an **agent team** with the following dedicated agents. Each agent works in its own context window to keep analysis focused. Pass each agent only the relevant subset of information.

**⚠️ CRITICAL SCOPE RULE — applies to ALL agents:** Review agents must **ONLY** comment on code that was actually introduced or changed in this branch (i.e. lines present in the diff against the base branch). Do NOT flag issues in pre-existing code that was not touched by this branch. If an agent needs to read surrounding code for context, that is fine, but all findings must reference lines from the diff only. Existing code patterns — even if imperfect — are out of scope for this review.

### Agent 1 — "Acceptance Criteria Reviewer"

**Input:** Jira ticket details (summary, description, acceptance criteria, **comments**) + full diff of changed files.

**Task:**

- Map each acceptance criterion from the ticket to concrete code changes **in the diff**.
- For every criterion, determine: ✅ Met, ⚠️ Partially Met, or ❌ Not Met.
- Provide evidence (file + snippet reference from the diff) for each determination.
- Flag any acceptance criteria that have **no corresponding code changes**.
- Flag any code changes that don't map to **any** acceptance criterion (scope creep).
- **Review ticket comments** for QA feedback, reviewer requests, or team clarifications. Check whether any actionable feedback from comments has been addressed in the code changes. Flag any unresolved comment feedback.

**Output:** A structured list of acceptance criteria with status and evidence, plus a summary of comment feedback and whether it has been addressed.

---

### Agent 2 — "Architecture & Best Practices Reviewer"

**Input:** Full diff of changed files + contents of your project's architecture documentation + relevant existing source files for pattern context.

**Task:**

- Read your architecture documentation thoroughly to understand the project's architecture, patterns, and conventions.
- Review every **changed line/file in the diff** against:
  - **iOS best practices**: proper use of Swift conventions, memory management (retain cycles, weak/unowned), concurrency patterns (async/await, actors, MainActor), error handling, SOLID principles.
  - **Project architecture alignment**: Does the code follow the layers, naming conventions, dependency rules, and patterns defined in your architecture docs?
  - **Code quality**: readability, testability, duplication, proper separation of concerns.
- For each finding, categorize as: 🔴 Blocker, 🟡 Suggestion, 🟢 Nitpick.
- If everything looks good for a file, explicitly state that.

**Output:** A categorized list of findings per file with severity, description, and suggested fix.

---

### Agent 3 — "Build Verification"

**Input:** The current project workspace on the current branch.

**Task:**

- Build the app using `xcodebuild`. Determine the correct scheme and destination from the project/workspace configuration (e.g. `xcodebuild -list` to discover schemes).
- Run a clean build:
  ```
  xcodebuild clean build -scheme <Scheme> -destination 'generic/platform=iOS' -quiet
  ```
- Capture the **full build output**, especially any errors or warnings.
- Determine: ✅ Build Succeeded or ❌ Build Failed.
- If the build fails, extract and list all **build errors** with file, line, and error message.
- Also collect any **build warnings that appear in files changed by this branch** (filter warnings by the changed file list).

**Output:** Build status (pass/fail), list of errors if any, and list of new warnings in changed files.

---

### Agent 4 — "API Correctness Reviewer"

**Input:** Diff of changed files (especially any **new or modified** API usage, framework imports, system calls introduced in this branch).

**Task:**

- Use the **`ios-api-researcher`** agent to validate only APIs that were **added or changed in the diff**:
  - Are the iOS/system APIs used correctly (correct parameters, availability, deprecation status)?
  - Are there newer or more appropriate APIs available for the target deployment version?
  - Are any APIs used in a way that could cause runtime issues (main thread violations, missing entitlements, etc.)?
- Document each API usage finding with the relevant file, line context, and recommendation.

**Output:** A list of API-related findings or an explicit "all clear" if no issues found.

---

## Step 3: Plan Mode — Synthesize & Present Draft

Once all four agents have reported back, **synthesize their outputs** into a unified review plan. Present this to the user in **plan mode** (do NOT create the file yet). The plan should include:

1. **Build Status** — pass/fail. If failed, this is an automatic 🔴 Blocker and the review cannot pass.
2. **Executive Summary** — overall health of the branch (e.g., "Ready to merge", "Needs minor changes", "Needs significant rework").
3. **Acceptance Criteria Status** — table of criteria and their status.
4. **Ticket Comment Feedback** — any unresolved QA/reviewer feedback from Jira comments.
5. **Architecture & Best Practices Findings** — grouped by severity.
6. **API Correctness Findings** — grouped by severity.
7. **Recommended Actions** — prioritized list of what the author should address before merge.

**A failed build automatically means the review does not pass**, regardless of all other findings.

Ask the user: _"Here is the review plan. Should I generate the final review document, or would you like to adjust anything first?"_

## Step 4: Generate Final Review Document

Only after the user confirms, create a markdown file at:

```
reviews/review-<branch-name>-<YYYY-MM-DD>.md
```

The file should follow this structure:

```markdown
# Code Review: <Branch Name>

**Ticket:** <TICKET-ID> — <Ticket Title>
**Branch:** `<branch-name>`
**Base:** `origin/develop`
**Reviewer:** Claude Code (automated)
**Date:** <YYYY-MM-DD>

---

## Executive Summary

<1-3 sentence overall assessment with clear recommendation: approve / request changes>

---

## Build Status

<✅ Build Succeeded — or — ❌ Build Failed>

<If failed: list all build errors with file, line, and message>
<If succeeded with new warnings in changed files: list warnings>

---

## Acceptance Criteria

| #   | Criterion | Status   | Evidence       |
| --- | --------- | -------- | -------------- |
| 1   | ...       | ✅/⚠️/❌ | File + context |

<Additional notes on criteria coverage>

---

## Ticket Comment Feedback

<Summary of actionable feedback found in Jira ticket comments (e.g. QA findings, reviewer requests, team clarifications) and whether each item has been addressed in the code changes. If no actionable comments: "No actionable feedback found in ticket comments.">

---

## Architecture & Code Quality

### 🔴 Blockers

<findings or "None">

### 🟡 Suggestions

<findings or "None">

### 🟢 Nitpicks

<findings or "None">

---

## API Correctness

<findings or "All APIs verified — no issues found.">

---

## Summary

<If no issues: "This branch looks good. The app builds successfully, all acceptance criteria are met, the code follows project architecture and iOS best practices, and API usage is correct. Ready for merge.">

<If issues exist: Prioritized action items for the author.>
```

## Important Notes

- **Scope: only review code from this branch's diff.** Never flag issues in pre-existing code that was not changed. Surrounding code may be read for context, but all findings must point to lines introduced or modified in this branch.
- **Base branch is configurable.** The examples use `origin/develop` but adjust to your project's main branch.
- **A successful build is a hard requirement.** If the build fails, the review automatically does not pass — flag it as the top-priority blocker.
- If the review finds **no issues at all**, explicitly state this clearly — a clean review is valuable information.
- Keep findings **actionable**: always suggest what to do, not just what's wrong.
- Reference specific files and code context so findings are easy to locate.
- The tone should be constructive and professional — this is meant to be shared on a GitHub PR.
