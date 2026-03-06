---
# orchestrator.md — Coordinator
on:
  push:
    tags:
      - "v*"

permissions:
  contents: read
  issues: read
  pull-requests: read

safe-outputs:
  create-issue:
    title-prefix: "[Release] "
    labels: [release, automated]

tools:
  github: {}
---

# Release Orchestrator

A new version tag was pushed: `${{ github.event.release.tag_name }}`.
Coordinate the full release process by dispatching worker workflows in sequence.

## Orchestration Steps

### Phase 1 — Validate
Dispatch `validate-release.md` with input `version: ${{ github.event.release.tag_name }}`.
Wait for it to complete. If it fails, stop and create a blocking issue.

### Phase 2 — Documentation
Dispatch `generate-release-notes.md` with input `version: ${{ github.event.release.tag_name }}`.
Wait for completion and confirm the output artifact path.

### Phase 3 — Notifications (run in parallel)
Dispatch all of the following simultaneously:
- `notify-discord.md` — Post release announcement to Discord
- `update-changelog.md` — Commit CHANGELOG.md update to main
- `close-milestone.md` — Close the matching GitHub milestone

### Phase 4 — Summary
Once all Phase 3 workers complete (regardless of individual success/failure),
create a summary issue listing:
- Which steps succeeded ✅
- Which steps failed ❌ (with links to the failed run)
- The final release status: COMPLETE, PARTIAL, or FAILED

If any critical step failed (Phase 1 or 2), mark the summary issue as `[BLOCKED]`.
