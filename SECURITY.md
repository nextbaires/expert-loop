# Security

## Reporting a vulnerability

Use GitHub's private vulnerability reporting on this repository
(Security → Report a vulnerability). Please do not open a public issue for anything that
could be exploited before it is fixed.

Expect a first response within a week.

## What counts as a vulnerability here

This project's job is to sit between untrusted text and an agent with write access, so the
interesting failures are ones that cross that boundary:

- A crafted issue body that causes the drafting run to write outside `allowed-paths`, edit
  a workflow, edit `CODEOWNERS`, or touch a dependency manifest.
- Any path to executing the loop without a maintainer having applied the trigger label.
- Exfiltration of `ANTHROPIC_API_KEY`, the `GITHUB_TOKEN`, or any other secret through
  drafted code, tool output, run logs, or pull request content.
- A drafted change that merges without human approval.
- Privilege escalation from proposer to committer through any route.

## What is known and accepted

**Prompt injection can influence drafted content.** The assistant reads attacker-authored
text. The permission deny-list and the label gate bound what it can *do*; they do not
guarantee that everything it *writes* is faithful to the honest reading of a proposal. A
crafted proposal could argue for a subtly wrong change persuasively enough that a hurried
reviewer approves it.

This is mitigated by review, not by configuration. The loop is built to make that review
tractable — the original proposal is quoted verbatim in the pull request so intent and
implementation sit side by side, every domain claim carries a source, and the assistant is
required to publish what it was unsure about. If you are reviewing a drafted pull request,
those three sections are the ones to read first.

Reports that demonstrate a *new* way to defeat the review step — rather than restating that
review is required — are in scope and welcome.

## Deploying this safely

- Keep the trigger on `issues: [labeled]` with a maintainer-only label. This is the primary
  control; everything else is defence in depth.
- Require `CODEOWNERS` review and enable branch protection on the default branch. Without
  this the second human gate does not exist.
- Scope `allowed-paths` as narrowly as the work allows.
- Set `max-turns` to a real ceiling. It bounds cost and blast radius together.
- Never grant the workflow more permissions than `contents: write`, `pull-requests: write`,
  `issues: write`.
- Review drafted dependency changes as if they were dependency changes, because they are.
  The deny-list blocks manifest edits precisely so this cannot happen silently — if a run
  asks for a new dependency, that request is a human decision.
