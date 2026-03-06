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
