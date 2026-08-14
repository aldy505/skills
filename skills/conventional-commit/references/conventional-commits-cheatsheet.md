# Conventional Commits Cheatsheet

Expanded from the [Conventional Commits 1.0.0](https://www.conventionalcommits.org/) specification and the qoomon commit-message cheatsheet.

## Commit message structure

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

- Type and description are required; scope, body, and footers are optional.
- Subject line must not exceed 72 characters; the whole commit message aims to avoid wide lines for tooling.

## Types and SemVer mapping

| Type | Release impact | Notes |
|------|----------------|-------|
| `feat` | MINOR | New feature visible to the end user. |
| `fix` | PATCH | Bug fix visible to the end user. |
| `refactor` | — | Neither fix nor feature; behavior should be preserved. |
| `perf` | — | Performance improvement; often pairs with `refactor`, may warrant a patch release. |
| `style` | — | Formatting/minor cosmetic changes; no production behavior change. |
| `test` | — | Add, fix, or improve tests. |
| `docs` | — | Documentation changes. |
| `build` | — | Build system, dependencies, or packaging. |
| `ops` | — | Operational changes: deployment configs, runbooks, infra, scripts. |
| `ci` | — | CI configuration and scripts. |
| `chore` | — | Default/catch-all maintenance; no other type fits. |
| `revert` | — | Reverts a prior commit; reference it in the body (e.g. `This reverts commit <hash>.`) |

**SemVer rule:** a commit with a `BREAKING CHANGE:` footer or a `!` after type/scope triggers a MAJOR version bump, regardless of type. `feat` → MINOR, `fix` → PATCH. Types without a defined impact (`refactor`, `perf`, etc.) do not force a release by themselves; teams decide their cutover policy.

## Type/scope usage

- Scope: a lowercase noun naming the affected component (`feat(api)`, `fix(payments)`). Multi-word: hyphenate (`fix(checkout-flow)`). No issue IDs in the scope; put ticket references in the footer (`Refs: #123`).
- A commit MAY carry multiple footers of the same type, and the `BREAKING CHANGE` scope (e.g. `BREAKING-CHANGE:`) is treated as matching the `BREAKING CHANGE:` token.

## Description guidelines

- Imperative mood, concise: "add", "fix", "remove" — not "adds", "fixed", "removing".
- Lowercase first character unless a proper noun/tool name otherwise.
- No trailing period.
- Keep under 72 characters; move detail into the body.

## Body

- Explain the *why* and the *what* beyond the subject.
- Wrap lines at roughly 72 columns.
- Distinguish intentional change from incidental churn.

## Breaking change indicators

Two equivalent signals; use one or both:

1. `!` after type/scope — subjects with `!` draw attention: `feat(api)!: remove legacy endpoint`.
2. Footer `BREAKING CHANGE:` followed by a description of the change and migration path:

```
feat(api)!: require API key on all endpoints

BREAKING CHANGE: clients must now send an Authorization header.
```

For example usage, `BREAKING CHANGE`, `BREAKING-CHANGE`, or `BREAKING CHANGE` (any capitalization) with a trailing `:` are recognized as the breaking-change footer, with breaking change content that need not start on the token's own line.

## Footers

- One token per line, `Token: value` form: `Refs: #123`, `Closes: #456`, `Reviewed-by: Z`, `BREAKING CHANGE: ...`.
- Token can use `-` in place of spaces (`Reviewed-by`, `Acked-by`).

## PR / squash-merge titles

- One commit in the branch: reuse the commit subject.
- Multiple commits: pick the highest-impact type (`feat`/`fix` before `chore`/`style`) plus a concise summary of the total change; keep the list of types and key changes in the PR body.
- Match the project's merge strategy (squash vs rebase) so the final commit message is Conventional.

## Validation checklist

- [ ] Type is one of `feat`, `fix`, `refactor`, `perf`, `style`, `test`, `docs`, `build`, `ops`, `ci`, `chore`, `revert`.
- [ ] Format `<type>(scope): <description>` or `<type>: <description>`.
- [ ] Description imperative, lowercase-first, no trailing period, ≤72 chars.
- [ ] Scope lowercase, no issue IDs.
- [ ] Breaking changes carry `!` and/or `BREAKING CHANGE:` footer.
- [ ] Body/footer lines wrapped at ~72 chars.