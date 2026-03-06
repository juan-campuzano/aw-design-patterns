---
on:
  issue_comment:
    types: [created]
  roles: [admin, maintainer, write]

command: /review-joke

permissions:
  contents: read
  issues: read
  pull-requests: read

safe-outputs:
  add-comment: {}

tools:
  github: {}
---

# Joke Review Bot

Someone has triggered `/review-joke` in this issue. Your job is to read all the
comments in the issue, identify the jokes, judge them, and crown a winner.

Issue context:
"${{ needs.activation.outputs.text }}"

## Instructions

1. **Collect all jokes**: Fetch every comment in this issue. Extract all jokes or
   humorous remarks — ignore non-joke comments (e.g., meta-discussion, questions,
   administrative messages).

2. **Evaluate each joke** using these criteria:
   - **Originality**: Is it fresh or a recycled classic?
   - **Delivery**: Is the timing/phrasing sharp?
   - **Relevance**: Does it fit the context of the issue?
   - **Reaction**: Did anyone react with 👍, 😄, 🎉, or similar emoji?

3. **Pick the winner**: Choose the single best joke based on your evaluation.
   In case of a tie, favour the one with more emoji reactions.

4. **Announce the winner** with a comment in this format:

   ---
   ## 🏆 Joke Review Results

   After carefully reviewing all submissions, the winner is...

   **@{author}** with:

   > {winning joke}

   **Why it won:** {1-2 sentence explanation of what made it stand out}

   **Honourable mentions:**
   - @{author2}: "{brief quote}" — {one-line reason}
   - @{author3}: "{brief quote}" — {one-line reason}

   *Judged by the Joke Review Bot. Results are final and non-negotiable.* 🎤⬇️
   ---

Keep the tone fun and celebratory. Be generous with praise for all participants.
