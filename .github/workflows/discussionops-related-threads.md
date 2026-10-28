---
on:
  discussion:
    types: [created]

permissions:
  contents: read
  discussions: read

safe-outputs:
  add-comment:
    max: 1

tools:
  github:
    toolsets: [default, discussions]

engine:
  id: copilot
  model: claude-sonnet-4.5
---

# Discussion Related-Threads Agent

A new discussion was just opened. Your job is to find older discussions in this
repository that already covered the same topic and point the author to them.

"${{ needs.activation.outputs.text }}"

Steps:
1. **Extract the topic**: summarize the new discussion in one sentence and list
   3-5 key terms (technologies, error messages, feature names, patterns).
2. **Scan existing discussions**: use the GitHub discussions tools to list
   discussions in this repository, paginating through the full history
   (open, closed and answered). Exclude the current discussion. Compare titles
   and bodies against the key terms first; only fetch the full thread and its
   comments for promising candidates.
3. **Rank candidates**: keep at most 3 discussions that truly cover the same
   topic or answer the same question. Prefer answered discussions, and ignore
   superficial keyword overlaps. For each, note the answer or conclusion
   reached (read the accepted answer or the final comments).
4. **Respond** with a single comment:
   - If there are relevant matches: greet the author, then list each one as
     `- #<number> <title> — <one-line summary of the conclusion>` and ask them
     to check whether it resolves their question or what is different about
     their case.
   - If a discussion is a clear duplicate: say so explicitly and suggest
     continuing the conversation there.
   - If nothing relevant exists: do **not** comment (use the noop output).

Rules:
- Never invent discussion numbers or links; only reference threads you read.
- Be concise and friendly; do not repeat the full content of old threads.
