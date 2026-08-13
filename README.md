# expert-loop

**A GitHub workflow that lets domain experts contribute code without writing code.**

An expert files an issue describing what they know. A maintainer reads it and opens the
gate. An assistant reads the repository, implements the proposal, writes a test, cites the
evidence, and opens a pull request. A human reviews and merges.

The expert never opens an editor. Their name is on the contribution.

---

## Why this exists

In seismology, in clinical medicine, in every field where the people who understand the
problem are not the people who write the software, the same thing happens: knowledge that
would improve a system never reaches it. The expert has the insight and no way to land it.
The maintainer has commit access and not enough domain knowledge to know what is missing.
The gap between them is a programming skill neither of them should need to have in common.

Translating a well-described idea into a small, tested, cited code change is now something
a language model does reliably. That is the entire thesis: **the bottleneck was never the
expertise, it was the translation** — and the translation step is the part that got cheap.

What the model does *not* do is decide whether the idea is right. That stays with the
people who know the field. The loop is built so the human judgement sits at both ends —
a maintainer decides what gets drafted, a reviewer decides what gets merged — and the
machine only does the part in the middle.

## How it works

```
Expert files an issue
  (structured form, no code, evidence required)
        │
        ▼
Maintainer reads it, applies `approved-for-drafting`   ← human gate #1
        │
        ▼
Workflow runs Claude against the repository
  reads conventions → implements → writes a test → cites every claim
        │
        ├── ambiguous or out of scope? → asks on the issue, opens nothing
        │
        ▼
Pull request, labelled `needs-domain-review`
        │
        ▼
Human review, CODEOWNERS approval, merge              ← human gate #2
```

Built on [`anthropics/claude-code-action`](https://github.com/anthropics/claude-code-action),
which supplies the agent, the branch, and the pull request. This repository supplies the
issue form, the drafting prompt, the permission boundary, and the review discipline.

## Use it in your repository

1. Copy [`examples/consumer-workflow.yml`](examples/consumer-workflow.yml) to
   `.github/workflows/expert-loop.yml` and set `domain`, `allowed-paths`, and
   `test-command`.
2. Copy [`.github/ISSUE_TEMPLATE/proposal.yml`](.github/ISSUE_TEMPLATE/proposal.yml) into
   your repository and rewrite the placeholder examples for your field. Do rewrite them —
   a form whose examples come from someone else's discipline reads as boilerplate and gets
   filled in badly.
3. Add an `ANTHROPIC_API_KEY` repository secret.
4. Create the labels `proposal`, `approved-for-drafting`, and `needs-domain-review`.
5. Add a `CODEOWNERS` file and turn on branch protection requiring review. Without this,
   gate #2 does not exist.

Optionally add `.github/expert-loop/CONTRIBUTING-DRAFT.md` with repository-specific
drafting instructions; it takes precedence over the built-in prompt.

### Inputs

| Input | Default | Notes |
|---|---|---|
| `domain` | *required* | Sets the assistant's frame, e.g. `seismology` |
| `allowed-paths` | *required* | Space-separated. Everything else is off limits |
| `test-command` | `""` | Must pass before a pull request opens |
| `model` | `claude-opus-5` | See cost, below |
| `max-turns` | `40` | The main cost ceiling |
| `contribution-guide` | `.github/expert-loop/CONTRIBUTING-DRAFT.md` | Optional, repo-specific |

Secret: `anthropic-api-key`.

### Cost

| Model | Input / output per MTok | Use for |
|---|---|---|
| `claude-opus-5` | $5 / $25 | Proposals needing real judgement — the default |
| `claude-sonnet-5` | $3 / $15 ($2 / $10 through 2026-08-31) | Mechanical, well-specified changes |

Cost per proposal is driven by how much of the repository the assistant has to read, so it
scales with repository size more than with proposal length. `max-turns` is the ceiling that
actually bounds a runaway run; set it deliberately rather than raising it reflexively.

## Security

The issue body is text written by a stranger, and it is fed to an agent that can write to a
branch. That is a prompt-injection surface, and it is treated as one:

- **Nothing runs until a maintainer applies a label.** Anyone can file an issue; only
  someone with write access can label one. Do not change the trigger to `issues: [opened]`.
- **The prompt frames issue content as evidence, never as instruction**, and directs the
  assistant to report injection attempts in the pull request rather than act on them.
- **Writes are denied** to workflows, `CODEOWNERS`, and dependency manifests, and confined
  to `allowed-paths`. Network fetches and `git push` are denied.
- **Merging requires human approval** via `CODEOWNERS` and branch protection.

The residual risk is real and worth stating plainly: a sufficiently careful injection could
still influence the *content* of a drafted change in a way a hurried reviewer misses. The
mitigation is the reviewer, which is why the loop is designed to make review easy — the
proposal is quoted verbatim in the pull request body so intent and implementation can be
compared side by side, and the assistant is required to list what it was unsure about.

Report vulnerabilities per [SECURITY.md](SECURITY.md).

## Attribution

Every drafted pull request credits the proposer by name and links the issue. The domain
judgement is theirs. If the implementation is wrong, that is the loop's failure and the
maintainer's to fix — not the contributor's.

## Used by

- [`nextbaires/seismic-bench`](https://github.com/nextbaires/seismic-bench) — open
  earthquake-forecasting benchmark
- [`nextbaires/open-eviden`](https://github.com/nextbaires/open-eviden) — clinical
  evidence-synthesis tooling

## License

MIT. See [LICENSE](LICENSE).
