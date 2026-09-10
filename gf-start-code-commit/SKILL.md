---
name: gf-start-code-commit
description: Git workflow rules for shipping code — creating a dedicated branch before the first commit of any new work, branch naming, Conventional Commits messages (single-line header only, no body), PR size limits, PR description content, merge requirements, and how to review a PR. Consult this before creating a branch, staging/committing anything, or opening or reviewing a pull request. Coding standards and design principles live in the separate gf-start-coding skill.
---

# Start Code Commit

Git-level conventions for turning finished, verified work (see `gf-start-coding` → worklog) into a branch, a commit, and a PR.

## Before the first commit: branch check

Never commit directly on `main`/`master`, and never keep committing on a branch that belongs to a different, already-shipped or unrelated piece of work.

- Before staging anything, check the current branch. If it is `main`/`master`, or an existing branch for a different logical change, **create a new branch first** using the naming convention below — do this automatically as part of the workflow, do not wait to be asked.
- One branch = one logical change. Unrelated work gets a new branch cut from the base, never appended to an open one.
- Only ask first when the *branch action itself* is destructive — force-push, history rewrite, branch deletion (see Boundaries below). Creating a new branch to hold new work is not one of those and does not need a separate go-ahead beyond the go-ahead for the task itself.

## Branch naming

```
feat/     fix/     hotfix/     ref/
perf/     test/    docs/       chore/
```

Examples: `feat/listing-search-filter`, `fix/auth-token-expiry`

- Slug is kebab-case and describes **the work**, not the ticket. `fix/auth-token-expiry`, not `fix/JIRA-2841`.
- `hotfix/` is a *process* distinction — commits inside it are still `fix`.

## Commit messages (Conventional Commits)

```
<type>(<scope>): <short description>
```

`feat` · `fix` · `refactor` · `perf` · `test` · `docs` · `build` · `ci` · `chore`

Example: `feat(listing): add price filter to search API`

- **Header line only. No body, no explanation paragraph, no bullet list under the header.** The git log is not a changelog and not a design doc. If a rejected alternative, trade-off, or non-obvious "why" needs recording, it goes in the PR description or the worklog — never in the commit body.
- Use only the types listed above. Inventing one such as `hotfix` means semantic-release and changelog tooling silently skips the commit — precisely when visibility matters most. Urgency belongs in the branch name and the PR, not the type.
- There is deliberately no `style` type. Formatting is the formatter's job. A commit that only reformats means the formatter is not wired up — fix that instead.
- `scope` is the affected module, domain, or package: one lowercase word. Not being able to name a single scope usually means the commit carries too much.
- Description is imperative, lowercase, no trailing period, under 72 characters.
- Breaking change: `!` after the scope, plus a `BREAKING CHANGE: <description>` footer — this is the one case where a trailing footer line is allowed; it is still not a body/explanation paragraph.
- One commit = one coherent change that builds. No WIP or "fix typo" commits on a shared branch; squash before opening the PR.

## PR size

Review quality drops sharply past a few hundred lines — the reviewer shifts from reading to skimming and approving.

- **Target ≤400 changed lines, ideally under 200.** Lockfiles, generated files, and snapshots don't count, but say in the description what was excluded.
- **One PR = one logical change.** Never mix refactoring with a feature: the reviewer cannot separate structural from behavioural changes, so both get waved through.
- **Split large work sequentially**, one PR per step, each merged before the next opens: preparatory refactor (no behaviour change) → feature → cleanup.
- **When it genuinely cannot be split** — a large migration, an API change rippling through call sites — give the reason plus a **suggested file reading order**.

If the size target and "one logical change" conflict, **one logical change wins**. Arbitrary 400-line slices cannot be reviewed independently or reverted safely. Exceed the target and explain why.

## PR description

Cover only what a reviewer needs. Where worklogs exist, link to the task's worklog instead of restating it.

- Link to `docs/worklogs/<file>.md` — goal, success criteria, and verification results already live there. Without a worklog, state the goal and how the change was verified, briefly.
- Suggested file reading order, if large or spanning several layers.
- Before/after screenshots, if the UI changed.
- What to look at closely: anything you are unsure about, alternatives considered and rejected, technical debt taken on deliberately.

## Merge requirements

- Verification complete and recorded — worklog **DONE**, every criterion PASS with pasted evidence.
- CI green: test, lint, type-check.
- At least one approval, every blocking comment resolved rather than dismissed.
- No commented-out code, no leftover debug logging, no TODO without a ticket.
- Squash merge, final commit title following Conventional Commits above (header only).

## Boundaries: stop and ask

- Force-push, history rewrite, branch deletion.
- Pushing to a remote, or any action visible to others (opening/closing/commenting on PRs or issues) beyond what the task already authorized.
- Editing CI/CD config as part of a commit — that is a separate boundary under `gf-start-coding`, not a git-mechanics decision to make here.

## Reviewing a PR

The branch/commit workflow above is for authoring. Reviewing follows a different procedure:

- Read the description and linked worklog **before** the diff. A diff without stated intent cannot be reviewed, only proofread.
- Separate blocking from non-blocking and say which is which. An unlabelled pile of comments stalls the PR while the author guesses.
- Review against the stated goal, not against the code you would have written. Style preferences no rule or config backs are non-blocking at most.
- A PR that mixes concerns, or exceeds the size guidance without explanation, gets sent back to be split before any line-level review.
