---
on:
  schedule: weekly

permissions:
  contents: read
  issues: read
  pull-requests: read

safe-outputs:
  create-issue:
    title-prefix: "[Policy Check] "
    labels: [compliance, automated]

tools:
  github: {}
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
