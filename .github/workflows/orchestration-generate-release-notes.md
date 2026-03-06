---
# generate-release-notes.md — Worker
on:
  workflow_dispatch:
    inputs:
      version:
        description: "Version tag to generate notes for"
        required: true

permissions:
  contents: read
  issues: read
  pull-requests: read

safe-outputs:
  create-pull-request:
    title-prefix: "[Release Notes] "

tools:
  github: {}
---

# Release Notes Generator

Generate release notes for version `${{ inputs.version }}`.

1. Fetch all PRs merged since the previous tag.
2. Group by type: Features, Bug Fixes, Breaking Changes, Docs, Chores.
3. Write a user-facing CHANGELOG entry in Keep a Changelog format.
4. Open a PR to merge the CHANGELOG update into main.
