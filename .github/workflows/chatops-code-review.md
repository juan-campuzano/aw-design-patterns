---
on:
  issue_comment:
    types: [created]
  roles: [admin, maintainer, write]

permissions:
  contents: read
  pull-requests: read
  issues: read

command: /review

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
