---
description: Triage pull request review comments via the pr-feedback-responder persona
---

`/pr-feedback` is the mirror of `/review`: that command produces findings, this one answers them.

## Gather the context first

Before dispatching, collect and pass along:

- The PR number or branch and its review threads (`gh pr view <n> --comments`, or the GitHub MCP tools if configured)
- The PR description and the spec or task it implements — a comment asking for something out of scope is a scope question, not a code question
- The diff under review

## Run the triage

Spawn the `pr-feedback-responder` subagent using the Agent tool (`subagent_type: pr-feedback-responder`). It returns a report classifying every comment as Accept, Accept with variation, Defer, Decline, or Needs the author — each with the change applied or the rule cited — plus a draft reply per thread and the linter, type check and test results after patching.

## Output

Return the full triage to the user. This is a single-persona command; no merge step is needed.

## Rules

1. Every comment gets a verdict and a draft reply — silence on a thread reads as being ignored.
2. Every decline cites a specific rule from `references/`, a project convention, or the spec — not a preference.
3. **Draft the replies; do not post them.** Posting to GitHub is an outward-facing action the human approves and sends.
4. Patches address only what the comments raise — a review thread is not an invitation to refactor.
5. If a comment reveals the change is fundamentally wrong, say so plainly rather than patching around it.
