---
on:
  schedule: daily

permissions:
  contents: read
  actions: read
  issues: read
  pull-requests: read

safe-outputs:
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
