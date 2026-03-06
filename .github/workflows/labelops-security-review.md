---
on:
  issues:
    types: [labeled]

permissions:
  contents: read
  issues: read
  pull-requests: read

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
