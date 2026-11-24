---
name: pull-request
description: Open high-quality, review-ready pull requests — never sloppy AI-generated ones. Covers branch naming, PR bodies with real evidence (mermaid, snippets, page screenshots for frontend, before/after, benchmarks), self-review, and stacked PRs. Use when opening or drafting a PR, naming a branch, writing or reviewing a PR description, or splitting a large change into a stack.
---

# Pull Request

A PR is the artifact a reviewer judges. AI may write it; it must not read like one. Every line must earn the reviewer's time.

## Quality bar

A sloppy PR wastes review cycles. Before opening one:

- **Self-review the diff.** No debug logs, commented-out code, stray files, dead code, or TODOs without an owner.
- **One concern per PR.** If it spans concerns, split it into a stack instead of shipping a grab bag.
- **Every claim is verifiable.** Link the file/symbol, command, benchmark, or screenshot behind it. Never state what you did not observe.
- **Cut filler.** No "This PR aims to…", no restating the diff, no effort narration, no apologies.
- **The reviewer can answer three questions:** what changed, why, and how do I verify it.

## Branching

Never edit or commit directly on `master`. Check out a branch first.

Name branches `username/{feat,fix,ref,chore,docs,test}/scope/description`, where `username` is the GitHub handle of the human running the agent:

```
alice/feat/customer-registration/passport-expiry
bob/fix/event-queue/duplicate-ack
carol/docs/authentication/auth-v3-flow
```

| Valid | Invalid |
|-------|---------|
| `alice/fix/api/timeout` | `fix/api-timeout` (no owner) |
| `bob/feat/billing/invoice-pdf` | `bob/feature/invoice` (wrong type) |
| `carol/chore/ci/pin-node` | `carol/fix/api/timeout-v2-final` (noise suffix) |

Types: `feat`, `fix`, `ref`, `chore`, `docs`, `test`. One scope, one short description.

## Writing the body

Include:

- **Bullets, not essays.** A few lines of context beat a wall of prose.
- **Mermaid** for anything sequential or structural: pipeline, request flow, state machine, dependency order. Prefer a diagram over describing the flow in prose.
- **Code samples** — internals worth reviewing (key function, schema, config) or sample usage of the new API.
- **Page screenshots for frontend PRs** — any change touching UI must show the rendered page in its real context, even when the diff looks non-visual. Before/after tables when visuals change; a single screenshot of the resulting page otherwise.
- **Before/after tables** for visual changes (direct or indirect), with uploaded images or videos.
- **Benchmark tables** for performance work: baseline from the target branch, candidate from the PR.
- **Code refs** — files, symbols, line ranges — so reviewers can jump straight in.

Exclude:

- Claims that tests ran or validation passed — CI shows that.
- Intermediate history: dropped line counts, commit-by-commit refactors, abandoned approaches. Reviewers only see the final squashed commit.
- Screenshots that show nothing changed.
- Placeholder text ("TBD", "will update later") — a draft PR may be incomplete, but its description must be true.

### Size the body to the change

A typo or one-line fix gets one or two sentences and nothing else — no diagram, screenshot, or table: `Fixes "recieve" → "receive" in the login error shown to users.`

Evidence demands scale with risk. Reserve long-form (context → storytelling → samples → before/after → diagrams → images) for genuinely difficult, high-risk, or wide-scoped changes.

### Evidence formats

- **Diagram** — a fenced `mermaid` block. Sequence, flow, state, or dependency; no decorative diagrams.
- **Visual / frontend** — before/after table with uploaded images or videos when visuals change; for frontend, a screenshot of the rendered page (not a mock) even when the diff looks non-visual.
- **Benchmark** — table of scenario, target branch, this PR, and delta. State the command and environment below it.

## Stacked PRs

Large or multi-concern changes ship as a stack of dependent branches, each reviewed on its own. Use the `gh stack` extension:

```bash
gh stack init 1-refactor     # first branch, based on master
gh stack add 2-rename        # child of 1-refactor
gh stack submit --auto       # push and open PRs for the whole stack
gh stack sync                # fetch, rebase, push
```

Rules:

- **Plan layers before writing code.** Foundational changes go in lower branches; dependent changes stack above.
- **Fix low layers at the source.** Navigate down, commit there, then rebase the branches above — never patch around it from the top branch.
- **Ship oldest-first** (closest to `master`) and re-root the stack after each merge.

## Review checklist

- [ ] Branch name matches `username/{type}/scope/description`; not `master`; title follows Conventional Commits.
- [ ] Body opens with the why, then bullets; trivial fixes stay one or two sentences.
- [ ] Flow, sequence, or structure conveyed by mermaid, not prose.
- [ ] Screenshot of the rendered page for frontend PRs; before/after table for visual changes; benchmark table for perf.
- [ ] Every claim backed by a ref, command, or screenshot.
- [ ] No filler, no test/validation narration, no intermediate PR history.
- [ ] Stacked PRs target their parent branch, not `master`.

## References

- [references/pr-body-examples.md](references/pr-body-examples.md) — full bodies for a feature, a UI change, a benchmark, and a stacked PR, plus a sloppy-vs-review-ready comparison.
