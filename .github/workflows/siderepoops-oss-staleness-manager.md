---
# This workflow lives in: my-org/internal-automation (private)
# It acts on:           my-org/awesome-oss-project (public)

on:
  schedule: daily

permissions:
  contents: read
  issues: read
  pull-requests: read

safe-outputs:
  add-comment:
    target-repo: my-org/awesome-oss-project
  add-labels:
    target-repo: my-org/awesome-oss-project
    allowed: [stale, needs-triage, good-first-issue]

tools:
  github: {}
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
