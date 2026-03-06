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
