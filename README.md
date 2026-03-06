# GitHub Agentic Workflows — Design Patterns Reference

> Examples for all 13 design patterns from the [GitHub Agentic Workflows documentation](https://github.github.com/gh-aw/introduction/architecture/).

---

## Table of Contents

1. [ChatOps](#1-chatops)
2. [DailyOps](#2-dailyops)
3. [IssueOps](#3-issueops)
4. [LabelOps](#4-labelops)
5. [ProjectOps](#5-projectops)
6. [DataOps](#6-dataops)
7. [TaskOps](#7-taskops)
8. [MultiRepoOps](#8-multirepoops)
9. [SideRepoOps](#9-siderepoops)
10. [SpecOps](#10-specops)
11. [TrialOps](#11-trialops)
12. [Monitoring](#12-monitoring)
13. [Orchestration](#13-orchestration)

---

## 1. ChatOps

**Pattern:** Trigger AI workflows via slash commands typed in issue or PR comments.

### Use Case: Automated Code Review on `/review`

```markdown
---
on:
  issue-comment:
    types: [created]

permissions:
  contents: read
  pull-requests: read

roles: [admin, maintainer, write]

commands:
  - /review

safe-outputs:
  add-comment: {}

tools:
  github: {}
---

# Code Review Bot

When someone types `/review` in a pull request comment, analyze the diff thoroughly.

Check for:
- Potential bugs or logic errors
- Security vulnerabilities (injection, auth issues, exposed secrets)
- Performance implications
- Missing tests or documentation
- Style and maintainability concerns

Post inline review comments on specific lines where issues are found.
End with a summary comment including an overall assessment and priority fixes.
```

### Key Points

- Commands like `/review`, `/deploy`, `/plan` activate the workflow.
- Only users with the specified `roles` can trigger it — never use `roles: all` in public repos.
- Use `needs.activation.outputs.text` (sanitized) instead of raw `github.event` fields.
- The main agent job runs **read-only**; `add-comment` (write) executes in a separate safe-output job.

---

## 2. DailyOps

**Pattern:** Schedule recurring AI workflows to post reports, updates, or digests automatically.

### Use Case: Daily Repository Status Report

```markdown
---
on:
  schedule: daily

permissions:
  contents: read
  issues: read
  pull-requests: read

safe-outputs:
  create-issue:
    title-prefix: "[Daily Report] "
    labels: [report, automated]

tools:
  github: {}

cache-memory: true
---

# Daily Repository Status Report

Generate a daily status report for maintainers. Include:

- **Activity Summary**: New issues opened/closed, PRs merged, commits pushed (last 24h)
- **Health Metrics**: Open bug count, stale PRs (> 7 days without activity)
- **Progress Tracking**: Milestones nearing deadline, recently completed tasks
- **Recommended Actions**: Top 3 items needing attention today

Keep the tone concise and actionable. Reference specific issue/PR numbers.
Use the cache at `/tmp/gh-aw/cache-memory/` to track trends across runs.
```

### Key Points

- Runs on a `schedule` — no human trigger required.
- `cache-memory: true` persists state across runs (metrics history, last run data).
- Ideal for daily standups, release notes drafts, or community engagement posts.
- Combine with `create-discussion` safe-output to post to a pinned thread instead of creating issues.

---

## 3. IssueOps

**Pattern:** Automatically triage, categorize, and respond to issues the moment they are opened.

### Use Case: Smart Issue Triage

```markdown
---
on:
  issues:
    types: [opened]

permissions:
  contents: read

safe-outputs:
  add-comment: {}
  add-labels:
    allowed: [bug, enhancement, question, needs-info, duplicate, documentation]

tools:
  github: {}
---

# Issue Triage Agent

Analyze this new issue and help the team respond quickly:

"${{ needs.activation.outputs.text }}"

Steps:
1. **Classify** the issue: bug report, feature request, question, or documentation gap.
2. **Check for duplicates** by searching existing open issues for similar topics.
3. **Assess completeness**: does a bug report include reproduction steps, environment
   info, and expected vs actual behavior?
4. **Respond** with a friendly comment:
   - If duplicate: link to the existing issue and suggest closing.
   - If incomplete: ask for the missing information.
   - If valid bug: acknowledge and suggest next steps.
5. **Apply labels**: choose at most 2 labels from the allowed list based on classification.
```

### Key Points

- Triggers on `issues: opened` — fully automated, zero human involvement.
- `needs.activation.outputs.text` combines title + body with all injections sanitized.
- `add-labels` uses an `allowed` list to prevent the agent from creating arbitrary labels.
- Write operations (`add-comment`, `add-labels`) happen in separate jobs with scoped permissions.

---

## 4. LabelOps

**Pattern:** Use labels as workflow triggers to route issues and PRs to specialized AI handlers.

### Use Case: Security Review Triggered by Label

```markdown
---
on:
  issues:
    types: [labeled]

permissions:
  contents: read

activate-if:
  label: needs-security-review

safe-outputs:
  add-comment: {}

tools:
  github: {}
---

# Security Review Agent

This issue has been labeled `needs-security-review`. Perform a focused security audit.

Issue context:
"${{ needs.activation.outputs.text }}"

Analyze for:
- Authentication or authorization flaws described in the issue
- Potential data exposure or privacy concerns
- Known vulnerability patterns (OWASP Top 10 relevance)
- Supply chain risks if third-party packages are mentioned

Respond with a structured security assessment comment. Tag severity as:
`[LOW]`, `[MEDIUM]`, `[HIGH]`, or `[CRITICAL]`.

Recommend whether this should be treated as a private security disclosure.
```

### Key Points

- Activates when a specific label (e.g., `needs-security-review`) is applied to an issue.
- Different label values can map to completely different workflows in the same repo.
- Enables human-in-the-loop routing: a human applies the label, AI does the specialized work.
- Combine with IssueOps: let the triage bot apply labels, then LabelOps kicks off deeper analysis.

---

## 5. ProjectOps

**Pattern:** Automate GitHub Projects (v2) management — sorting, prioritizing, and updating cards with AI.

### Use Case: Auto-Populate Project Board on Issue Creation

```markdown
---
on:
  issues:
    types: [opened]

permissions:
  contents: read

safe-outputs:
  update-project-item:
    project-url: https://github.com/orgs/my-org/projects/5
    token: ${{ secrets.PROJECT_TOKEN }}

tools:
  github: {}
---

# Project Board Automation

A new issue was created. Add it to our project board and set the appropriate fields.

Issue details:
"${{ needs.activation.outputs.text }}"

Determine and set the following project fields:
- **Status**: "Triage" for new items, "Backlog" if clearly understood
- **Priority**: P0 (critical/blocking), P1 (high), P2 (medium), P3 (low)
- **Effort**: XS (<1h), S (1-4h), M (4-8h), L (1-3d), XL (>3d)
- **Team**: Frontend, Backend, Infrastructure, or Docs based on content
- **Sprint**: Leave empty unless this is clearly urgent for the current sprint

Base your decisions on the issue content. When in doubt, default to Triage/P2/M.
```

### Key Points

- Requires a **pre-created** GitHub Project (create it in the UI first, then reference its URL).
- The `PROJECT_TOKEN` must have Projects read/write scope — never store it in the workflow file.
- The agent job never sees the Projects token directly; it flows only to the safe-output job.
- Supports user-owned and org-owned projects with different token types.

---

## 6. DataOps

**Pattern:** Deterministic data extraction in scripted steps, followed by agentic analysis and reporting.

### Use Case: Weekly Dependency Vulnerability Report

```markdown
---
on:
  schedule: weekly

permissions:
  contents: read
  security-events: read

safe-outputs:
  create-issue:
    title-prefix: "[Security Audit] "
    labels: [security, automated]

tools:
  github: {}

network:
  allowed:
    - defaults
    - node
    - python
---

# Dependency Vulnerability Analysis

Raw vulnerability scan data has been extracted in earlier steps and written to
`/tmp/gh-aw/scan-results.json`. Historical data is at
`/tmp/gh-aw/cache-memory/history.json`.

## Analysis Tasks

1. Parse the scan results and group vulnerabilities by severity (Critical, High, Medium, Low).
2. Identify any **new** vulnerabilities compared to the previous run.
3. For Critical and High findings, research the CVE details and suggest a fix or
   version upgrade.
4. Calculate a trend: is the overall vulnerability count improving or worsening?
5. Produce a structured report as an issue with:
   - Executive summary (2-3 sentences)
   - Critical/High table (CVE, package, current version, fixed version)
   - Recommended action plan ordered by priority
```

### Key Points

- Follows a two-phase model: **deterministic scripts first** (npm audit, pip-audit) → **agentic analysis second**.
- Data is written to `/tmp/gh-aw/` by setup steps and read by the agent.
- Keeps AI costs predictable by not letting the agent do raw data collection.
- Combine `cache-memory` with DataOps to track trends across scheduled runs.

---

## 7. TaskOps

**Pattern:** Break large goals into discrete sub-tasks and let the agent execute them one by one.

### Use Case: Automated Documentation Gap Filler

```markdown
---
on:
  workflow-dispatch:
    inputs:
      target-module:
        description: "Which module to document (e.g., src/auth)"
        required: true

permissions:
  contents: read

safe-outputs:
  create-pull-request:
    branch-prefix: "docs/auto-"
    title-prefix: "[TaskOps] "
    draft: true

tools:
  github: {}
  filesystem: {}
---

# Documentation Task Agent

You are filling documentation gaps for the module: `${{ inputs.target-module }}`

Work through each task in order. Complete one fully before moving to the next.

### Task 1 — Audit
Scan all `.ts` / `.js` / `.py` files in the target module.
List every exported function, class, and type that lacks a JSDoc/docstring.
Output a checklist to `/tmp/gh-aw/audit.md`.

### Task 2 — Generate Docs
For each undocumented item in the audit:
- Write a concise, accurate docstring/JSDoc comment
- Include `@param`, `@returns`, and `@throws` where applicable
- Use the existing code logic — do NOT hallucinate behavior

### Task 3 — Validate
Re-scan after writing. Confirm coverage improved. Note any items intentionally skipped.

### Task 4 — Create PR
Bundle all doc changes into a draft PR with a summary of what was documented.
```

### Key Points

- Driven by explicit numbered tasks — gives the agent a clear sequence to follow.
- `workflow-dispatch` with inputs allows humans to scope the task before it runs.
- `draft: true` on the PR ensures human review before merging.
- TaskOps is often combined with ChatOps (e.g., `/document src/auth` triggers this workflow).

---

## 8. MultiRepoOps

**Pattern:** Extend agentic automation across multiple repositories — sync, enforce, and coordinate.

### Use Case: Organization-Wide Security Policy Enforcement

```markdown
---
on:
  schedule: weekly

permissions:
  contents: read

safe-outputs:
  create-issue:
    target-repos:
      - my-org/repo-frontend
      - my-org/repo-backend
      - my-org/repo-infrastructure
    title-prefix: "[Policy Check] "
    labels: [compliance, automated]

tools:
  github:
    token: ${{ secrets.ORG_READ_TOKEN }}
---

# Cross-Repository Policy Enforcement

Audit the following repositories for compliance with org security policy:
- my-org/repo-frontend
- my-org/repo-backend
- my-org/repo-infrastructure

For each repository, check:
1. **Branch Protection**: Is `main` protected? Are reviews required?
2. **Dependency Scanning**: Is Dependabot enabled?
3. **Secret Scanning**: Is secret scanning active?
4. **Actions Pinning**: Are third-party actions pinned to SHAs?
5. **CODEOWNERS**: Does a CODEOWNERS file exist?

For each violation found, create a targeted issue in the offending repository
describing the compliance gap and steps to remediate it.
Do not create issues for repositories that are fully compliant.
```

### Key Points

- `target-repos` in safe-outputs allows creating issues/PRs in **external repositories**.
- The `ORG_READ_TOKEN` must be a token with access to all target repos — kept as a secret.
- The agent job uses a scoped read token; write tokens only flow to safe-output jobs.
- Great for org-wide security patches, dependency bumps, or policy rollouts.

---

## 9. SideRepoOps

**Pattern:** Host agentic workflows in a private "sidecar" repository that acts on other repos.

### Use Case: Private Automation Repo Managing a Public OSS Project

```markdown
---
# This workflow lives in: my-org/internal-automation (private)
# It acts on:           my-org/awesome-oss-project (public)

on:
  schedule: daily

permissions:
  contents: read

safe-outputs:
  add-comment:
    target-repo: my-org/awesome-oss-project
  add-labels:
    target-repo: my-org/awesome-oss-project
    allowed: [stale, needs-triage, good-first-issue]

tools:
  github:
    token: ${{ secrets.OSS_REPO_TOKEN }}
---

# OSS Issue Staleness Manager (SideRepo)

Manage stale issues in my-org/awesome-oss-project from this private automation repo.

## Workflow

1. **Find stale issues**: Search for open issues with no activity in the last 30 days
   and no `pinned` or `roadmap` label.

2. **Evaluate each stale issue**:
   - Is it still relevant given recent project direction?
   - Is it a good candidate for a first-time contributor? (`good-first-issue`)
   - Should it be flagged as stale?

3. **Take action**:
   - Apply `stale` label and post a polite "Is this still relevant?" comment.
   - Apply `good-first-issue` if appropriate and add a helpful getting-started comment.
   - Do NOT close issues — humans should make that final call.

Keep comments friendly and community-oriented in tone.
```

### Key Points

- The workflow YAML lives in a **private repo** — your automation logic stays confidential.
- `target-repo` points all safe-outputs to the public repo being managed.
- `OSS_REPO_TOKEN` grants the side-repo permission to comment on the target repo.
- Excellent for separating proprietary automation logic from the open-source project it manages.

---

## 10. SpecOps

**Pattern:** Use AI to generate, validate, or evolve specifications and structured documentation from code or requirements.

### Use Case: Auto-Generate OpenAPI Spec from Route Handlers

```markdown
---
on:
  push:
    paths:
      - "src/routes/**"
    branches:
      - main

permissions:
  contents: read

safe-outputs:
  create-pull-request:
    branch-prefix: "spec/openapi-update-"
    title-prefix: "[SpecOps] Update OpenAPI spec — "
    draft: false

tools:
  filesystem: {}
  github: {}
---

# OpenAPI Specification Updater

A push to `src/routes/` was detected. Update the OpenAPI specification to match.

## Instructions

1. **Read current spec**: Load `docs/openapi.yaml` as the baseline.

2. **Scan changed route files**: Examine the diff for new, modified, or deleted routes.
   Look for:
   - HTTP method and path
   - Request body schema (TypeScript types, Zod schemas, Pydantic models)
   - Response types and status codes
   - Authentication requirements (Bearer, API key, none)
   - Query parameters and path variables

3. **Update the spec**: Modify `docs/openapi.yaml` to reflect changes.
   - Follow existing naming conventions and style.
   - Use `$ref` for shared schemas where possible.
   - Include `summary` and `description` for all new endpoints.
   - Mark deprecated endpoints with `deprecated: true`.

4. **Validate**: Ensure the resulting YAML is valid OpenAPI 3.1.

5. **Open a PR** with the spec changes. Title should mention which routes changed.
```

### Key Points

- Triggers on path-filtered pushes — only runs when route files actually change.
- The agent reads both code and existing spec to produce an accurate delta update.
- Keeps documentation in sync with code automatically, reducing drift.
- Works equally well for AsyncAPI, GraphQL schema files, or Protobuf definitions.

---

## 11. TrialOps

**Pattern:** Test and iterate on workflows safely in an isolated trial repository before deploying to production.

### Use Case: Validating a New IssueOps Workflow Before Going Live

```markdown
---
# trial-issueops-triage.md
# Run in: my-org/aw-trial-sandbox (isolated test repo)

on:
  issues:
    types: [opened]

permissions:
  contents: read

safe-outputs:
  add-comment: {}
  add-labels:
    allowed: [bug, enhancement, question, needs-info]

tools:
  github: {}

trial:
  enabled: true
  log-outputs: true
  dry-run-external: true
---

# IssueOps Triage — TRIAL MODE

This workflow is under validation. Treat every issue as a test case.

"${{ needs.activation.outputs.text }}"

Triage this issue as you would in production:
1. Classify: bug, enhancement, question, or needs-info.
2. Draft a response comment — be helpful and specific.
3. Suggest appropriate labels (max 2).

After completing the triage, append a **[TRIAL EVALUATION]** section to your comment:
- Was the issue text realistic? (yes/no)
- Were your label choices appropriate? (yes/no)
- Confidence in classification: (low/medium/high)
- Any edge cases or prompt improvements needed?

This self-evaluation helps us refine the production workflow.
```

### Key Points

- All testing happens in a dedicated **sandbox repo** — no impact on real projects.
- `trial.log-outputs: true` captures everything to artifacts for post-run review.
- Add a self-evaluation section to the agent's instructions to surface prompt weaknesses.
- Once validated, copy the workflow to the target repo and remove trial configuration.

---

## 12. Monitoring

**Pattern:** Use GitHub Projects and AI to track and surface the health of agentic workflows themselves.

### Use Case: Workflow Health Dashboard

```markdown
---
on:
  schedule: daily

permissions:
  contents: read
  actions: read

safe-outputs:
  update-project-item:
    project-url: https://github.com/orgs/my-org/projects/12
    token: ${{ secrets.PROJECT_TOKEN }}
  create-issue:
    title-prefix: "[Alert] "
    labels: [workflow-alert, automated]

tools:
  github: {}
---

# Agentic Workflow Monitor

Audit the health of all agentic workflows in this organization.

## Monitoring Targets

Check recent run history for:
- `daily-report.md` — should succeed daily
- `issue-triage.md` — should succeed on every new issue
- `openapi-updater.md` — should succeed on every route push

## Health Checks

For each workflow:
1. **Success Rate**: Calculate success rate over the last 7 days (pass if > 95%).
2. **Avg Duration**: Flag if average duration increased > 50% vs the prior week.
3. **Cost Trend**: Note if token usage increased significantly.
4. **Last Failure**: If the workflow failed in the last 24h, capture the error reason.

## Actions

- **Update project board**: Set each workflow's Status (Healthy/Degraded/Failing)
  and update the "Last Checked" custom field to today's date.
- **Create an alert issue** only if a workflow has been Failing for 2+ consecutive days.
  Include the error message and a link to the failed run.

Do not create duplicate alerts if an issue for the same workflow is already open.
```

### Key Points

- Monitors the health of **other workflows** — meta-automation.
- The project board provides a persistent, visual health dashboard.
- Alert issues are deduplicated by checking for existing open issues before creating.
- Combine with DailyOps scheduling to get a morning health digest.

---

## 13. Orchestration

**Pattern:** Use a coordinator workflow to dispatch and sequence multiple specialized worker workflows.

### Use Case: Full Release Automation Pipeline

```markdown
---
# orchestrator.md — Coordinator
on:
  push:
    tags:
      - "v*"

permissions:
  contents: read
  actions: write

safe-outputs:
  create-issue:
    title-prefix: "[Release] "
    labels: [release, automated]

tools:
  github: {}
---

# Release Orchestrator

A new version tag was pushed: `${{ github.ref_name }}`.
Coordinate the full release process by dispatching worker workflows in sequence.

## Orchestration Steps

### Phase 1 — Validate
Dispatch `validate-release.md` with input `version: ${{ github.ref_name }}`.
Wait for it to complete. If it fails, stop and create a blocking issue.

### Phase 2 — Documentation
Dispatch `generate-release-notes.md` with input `version: ${{ github.ref_name }}`.
Wait for completion and confirm the output artifact path.

### Phase 3 — Notifications (run in parallel)
Dispatch all of the following simultaneously:
- `notify-discord.md` — Post release announcement to Discord
- `update-changelog.md` — Commit CHANGELOG.md update to main
- `close-milestone.md` — Close the matching GitHub milestone

### Phase 4 — Summary
Once all Phase 3 workers complete (regardless of individual success/failure),
create a summary issue listing:
- Which steps succeeded ✅
- Which steps failed ❌ (with links to the failed run)
- The final release status: COMPLETE, PARTIAL, or FAILED

If any critical step failed (Phase 1 or 2), mark the summary issue as `[BLOCKED]`.
```

### Worker Example: `generate-release-notes.md`

```markdown
---
# generate-release-notes.md — Worker
on:
  workflow-dispatch:
    inputs:
      version:
        description: "Version tag to generate notes for"
        required: true

permissions:
  contents: read

safe-outputs:
  create-pull-request:
    branch-prefix: "release/notes-"
    title-prefix: "[Release Notes] "

tools:
  github: {}
---

# Release Notes Generator

Generate release notes for version `${{ inputs.version }}`.

1. Fetch all PRs merged since the previous tag.
2. Group by type: Features, Bug Fixes, Breaking Changes, Docs, Chores.
3. Write a user-facing CHANGELOG entry in Keep a Changelog format.
4. Open a PR to merge the CHANGELOG update into main.
```

### Key Points

- The orchestrator uses `workflow-dispatch` to trigger named worker workflows.
- Workers are independent — they can be triggered manually or by the orchestrator.
- The orchestrator handles sequencing, parallelism, and failure aggregation.
- Phased execution (validate → document → notify) mirrors a CI/CD pipeline, in natural language.
- Each worker has minimal, scoped permissions — only the orchestrator needs `actions: write`.

---

## Quick Reference

| Pattern | Trigger | Key Safe Output | Primary Use |
|---|---|---|---|
| **ChatOps** | Comment (`/command`) | `add-comment` | Human-in-the-loop automation |
| **DailyOps** | Schedule | `create-issue`, `create-discussion` | Recurring reports & digests |
| **IssueOps** | Issue opened | `add-comment`, `add-labels` | Triage & auto-response |
| **LabelOps** | Label applied | `add-comment` | Label-based routing |
| **ProjectOps** | Issue/PR events | `update-project-item` | Project board automation |
| **DataOps** | Schedule or push | `create-issue`, `create-pr` | Extract → Analyze → Report |
| **TaskOps** | Dispatch or command | `create-pull-request` | Sequential task execution |
| **MultiRepoOps** | Any + `target-repos` | All (cross-repo) | Org-wide coordination |
| **SideRepoOps** | Any (private repo) | All (cross-repo) | Private automation logic |
| **SpecOps** | Push to spec paths | `create-pull-request` | Spec generation & sync |
| **TrialOps** | Any (sandbox repo) | All (isolated) | Safe workflow testing |
| **Monitoring** | Schedule | `update-project-item`, `create-issue` | Workflow health tracking |
| **Orchestration** | Tags or dispatch | `create-issue` + dispatch | Multi-workflow pipelines |

---

*All examples follow the GH-AW security model: agent jobs run read-only, all writes are delegated to scoped safe-output jobs, and user-provided content is accessed via `needs.activation.outputs.text` (sanitized).*