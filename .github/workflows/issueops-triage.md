---
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
