---
# trial-issueops-triage.md
# Run in: my-org/aw-trial-sandbox (isolated test repo)

on:
  issues:
    types: [opened]

permissions:
  contents: read
  issues: read
  pull-requests: read

safe-outputs:
  add-comment: {}
  add-labels:
    allowed: [bug, enhancement, question, needs-info]

tools:
  github: {}

strict: true
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
