---
on:
  workflow_dispatch:
    inputs:
      target-module:
        description: "Which module to document (e.g., src/auth)"
        required: true

permissions:
  contents: read
  issues: read
  pull-requests: read

safe-outputs:
  create-pull-request:
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
