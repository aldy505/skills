---
name: conventional-commit
description: Generate Conventional Commits 1.0.0 commit messages and pull request titles. Use when asked to write a commit message, summarize a diff, create a PR title, choose a commit type or scope, or format a squash-merge message.
---

# Conventional Commit

## Formula

```
<type>(<optional scope>): <description>
```

## When to use

- Writing a commit message from a diff or staged changes.
- Choosing a commit type or scope.
- Creating a pull request title or summarizing a series of commits.
- Formatting a squash-merge commit message.
- Reviewing whether an existing message conforms.

## Types

| Type | SemVer | When to use |
|------|--------|-------------|
| `feat` | MINOR | A new feature or capability for the end user. |
| `fix` | PATCH | A bug fix that corrects behavior. |
| `refactor` | — | Code change that neither fixes a bug nor adds a feature. |
| `perf` | — | A performance improvement (probably PATCH-like, still defaults to `feat`/`fix` semantics). |
| `style` | — | Formatting, whitespace, semicolons, missing imports; no behavior change. |
| `test` | — | Adding or correcting tests. |
| `docs` | — | Documentation only. |
| `build` | — | Changes to build system, dependencies, or tooling that affect compilation/packaging. |
| `ops` | — | Operational changes: configs, scripts, deployment, infrastructure. |
| `ci` | — | Changes to CI configuration files and scripts. |
| `chore` | — | Maintenance that doesn't fit other types (housekeeping, tooling, minor tasks). |
| `revert` | — | Reverting a previous commit. |

SemVer mapping: `fix` → PATCH, `feat` → MINOR, `BREAKING CHANGE` → MAJOR. Types other than `feat`/`fix` do not imply a version bump by themselves.

## Scope

- Lowercase noun describing the affected area (e.g. `auth`, `billing`, `api`).
- No issue IDs in the scope.
- Optional — omit when the change is broad or the scope adds no signal.

## Description

- Imperative mood ("add", "fix", not "adds"/"added").
- Lowercase first letter.
- No trailing period.
- ≤72 characters.

## Breaking changes

- Add `!` after the type/scope: `feat(api)!: remove legacy endpoint`.
- And/or append a `BREAKING CHANGE:` footer describing the migration impact.

## Body and footer

- Body: explain the why and what beyond the subject; wrap at ~72 columns.
- Footer: `BREAKING CHANGE:`, `Refs: <issue>`, `Closes: <issue>` etc., one per line, `Token: value` form.

## Pull request titles

- Single commit: use the commit subject.
- Multiple commits: highest-impact type + concise summary; list key changes in the body.

## Examples

```
feat: add refresh token rotation

feat(auth): rate-limit login attempts

fix(api): parse ISO-8601 timestamps with timezone

perf(db): cache repeated lookups in the hot path

refactor(orders): extract discount calculation

test(cart): cover empty-cart checkout path

docs: explain env var precedence

build: pin toolchain to 1.80

ops: scale workers during peak

ci: cache node_modules across jobs

chore: tidy import order

revert: restore legacy retry backoff

feat(api)!: require API key on all endpoints

BREAKING CHANGE: clients must now send an Authorization header.
```

## Validation checklist

- [ ] Type is one of `feat`, `fix`, `refactor`, `perf`, `style`, `test`, `docs`, `build`, `ops`, `ci`, `chore`, `revert`.
- [ ] Subject `: ` or `): ` separated type/scope and description.
- [ ] Description imperative, lowercase-first, no trailing period, ≤72 chars.
- [ ] Scope lowercase and free of issue IDs.
- [ ] `!` and/or `BREAKING CHANGE:` footer on breaking changes.
- [ ] Body/footer lines under ~72 chars.

## References

- `references/conventional-commits-cheatsheet.md` — expanded rules, type details, and SemVer mapping.