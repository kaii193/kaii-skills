---
name: kaii-coding-standards
description: The baseline quality floor and working rules for writing code — naming, self-documenting code over comments, guard-clause control flow, error handling, security, type safety, testing, scope discipline, which actions require asking the user first, and the required worklog workflow covering success criteria, verification, and what counts as done. Consult this whenever writing, refactoring, debugging, or reviewing code, including when the user does not mention standards at all. Git branch, commit, and PR conventions live in the separate kaii-git-workflow skill.
---

# Coding Standards (Baseline)

The shared quality floor for every implementation and review. Language-agnostic — framework and pattern specifics belong in separate skills layered on top.

**When implementing**, treat each rule as a guardrail while writing. **When reviewing**, treat each as a checklist item.

## Read next

This file covers how code should be written. The surrounding workflow lives elsewhere:

| Read | When |
|---|---|
| `references/worklog.md` | Starting any task that changes code. Covers success criteria, the worklog file, verification, and what counts as done. Read it **before writing code** — the criteria come first. |
| **`kaii-git-workflow` skill** | Naming a branch, writing a commit message, or opening and reviewing a PR. Separate skill, not part of this one. |

## Rule precedence

Conflicts are inevitable. Resolve them in this order:

1. **Repo tool config** — linter, formatter, type-checker. The config is the source of truth for formatting and lint; it wins over anything written here.
2. **Project documentation** — `{project-root}/**/project-context.md` and equivalent project conventions.
3. **This baseline.**
4. **Language and framework idiom** — where this baseline is silent, follow what the ecosystem does.

When two rules *inside* this baseline pull against each other, prefer in this order: **correctness and safety → readability → consistency → brevity**. Never trade the first for the last.

## Boundaries: stop and ask

Some changes are cheap to propose and expensive to undo. For anything on this list, describe what you intend to do and why, then wait for an explicit go-ahead. Do not bundle an unapproved change into an approved task.

- Deleting files, directories, or database records.
- Changing a database schema, or writing/running a migration.
- Adding, upgrading, or removing a dependency.
- Editing CI/CD, build, deployment, or infrastructure config.
- Changing a public contract — route paths, response shapes, exported function signatures, event payloads.
- Touching authentication, authorization, payments, or any code path handling money or personal data.
- Running any command with effects outside the working tree: network writes, deploys, or anything pointed at a non-local database.
- Rewriting or reformatting code the task did not require you to touch.

Finding a bug, a hardcoded secret, or bad code outside your task is not authorization to fix it. Report it, and record it under follow-ups.

## Scope discipline

The diff should contain the task and nothing else. Unrelated changes hide the real one from the reviewer and make the change impossible to revert cleanly.

- Change only what the requirement needs. Noticed something else worth fixing? Note it as a follow-up in the worklog — do not fix it here.
- No drive-by reformatting. A whitespace pass across a file destroys the diff and buries the two lines that matter.
- No opportunistic renames outside the code you already had to modify.

## Principles

- **Readability first** — code is read far more than written. Clear names and self-documenting structure beat cleverness or comments.
- **KISS** — the simplest solution that works. No premature optimization, no speculative abstraction.
- **DRY** — extract repeated logic into named functions/modules; avoid copy-paste. But do not abstract two things that are only coincidentally similar.
- **YAGNI** — build what the current requirement needs. Add complexity when it is actually required, not before.

## Naming

- Names state intent: `marketSearchQuery`, `isUserAuthenticated`, `totalRevenue` — not `q`, `flag`, `x`.
- Functions read as **verb + noun**: `fetchMarketData`, `calculateSimilarity`, `isValidEmail` — not `market`, `similarity`, `email`.
- Booleans read as predicates: `is…`, `has…`, `should…`.
- Magic numbers and strings become named constants: `MAX_RETRIES = 3`, never a bare `3`.

## Self-documenting code over comments

The name is the documentation. A comment restating the code is a maintenance liability — it goes stale, and it signals that the code failed to communicate. **Fix the name, don't add the comment.**

When tempted to write an explanatory comment, work down this list and stop at the first that applies:

1. **Rename** the variable or function so the intent is obvious.
2. **Extract** the confusing block into a well-named function — the name replaces the comment.
3. **Only then**, if real context remains that code cannot express, write the comment.

Two patterns this specifically replaces:

- A comment describing **what a condition means** — extract the condition into a named boolean instead. The name travels with the value.
- A comment used as a **section header inside a long function** — each header is a function waiting to be extracted. The body then reads as a list of named steps.

**Comments that earn their place** (they carry what code cannot):

- **Why**, not what — the reasoning behind a non-obvious choice, such as why a retry uses exponential backoff.
- Constraints and trade-offs — why a rule in this document is being knowingly broken here.
- Workarounds — what is being worked around, plus a version or issue reference so the comment can be deleted when it expires.
- Doc comments on public and exported APIs — purpose, parameters, return value, errors raised.

**Never**: commented-out code, changelog or authorship comments (dates, names, ticket history — that is the commit log's job), or a comment narrating the next line.

## Control flow: guard clauses over if/else

Handle invalid and edge cases with early exits at the top, then let the happy path run unindented below. Nesting hides the main logic and forces the reader to carry a mental stack of conditions.

- Give each failure case its own `if (!condition) return | throw | continue`.
- Delete `else` after a branch that always exits via `return`, `throw`, or `break`. It is dead weight.
- In loops, `continue` is the guard. Don't wrap the whole body in an `if`.
- Nested ternaries are nesting. Extract into a named function or an early return.

**Separate guards, or one named boolean?** If the conditions represent *distinct failures* that deserve different messages or different handling, give each its own guard. If they are facets of *one concept* with one outcome, combine them into a named boolean and guard on that. `isEligibleForRefund` is one concept; "input missing" and "email invalid" are two failures.

**On nesting depth:** count any construct that indents a body — `if`, `for`, `while`, `try`, `switch`, closure. Three levels is the signal to extract; a fourth is not a judgement call, extract it. This is about the reader's working memory, not about hitting a number, so a shallow function doing six unrelated things is still wrong.

## Immutability

- Do not mutate shared inputs in place. Build a new value rather than modifying an object or array the caller may still hold.
- Where in-place mutation is a deliberate performance decision, keep it local to the function and state why in a comment — one of the "why" comments that earns its place.

## Error handling

- Handle the failure paths, not just the happy path. Validate external input and check operation results before using them.
- Fail with a clear, contextual message. Never swallow an error silently. Preserve the original cause when re-raising.
- Don't catch what you can't handle — let it propagate to a layer that can.
- Errors surfaced to users carry no internals: no stack traces, SQL, file paths, or dependency names. Log the detail, return the summary.

## Security

Cheaper to get right while writing than to retrofit after a review.

- **Secrets never enter the repo.** Read them from environment or a secret manager. Finding a hardcoded secret is a stop-and-report event — rotating it is the owner's call, and quietly moving it to a config file fixes nothing.
- **Authorization is checked where the resource lives** — in the service or handler that owns it, not in the UI and not only at the route. A hidden button is not an access control.
- **Never build queries or commands by string concatenation** with outside data. Parameterized queries, argument arrays for subprocesses, no shell interpolation of user input.
- **Encode on output** according to the destination — HTML, SQL, shell, URL each need their own escaping. Sanitizing on input alone is not equivalent.
- **Logs carry no secrets, tokens, credentials, or personal data**, and no full request or response bodies from authenticated endpoints.
- **Every new dependency is a liability.** Prefer the standard library. A new package needs a reason, a look at its maintenance status, and — per Boundaries above — the user's agreement.

## Concurrency

- Run independent async work in parallel when there is no data dependency between the calls.
- Keep dependent work ordered. Don't parallelize operations that must observe each other's results.

## Type safety and contracts

- Give functions and data structures explicit, precise types. Avoid untyped escape hatches (`any` and equivalents) outside genuine boundaries, and constrain those.
- Validate data crossing a trust boundary — user input, network, storage — against a schema before trusting its shape.

## Code smells

- **Long functions** — one function doing many things splits into named steps.
- **Deep nesting** — see the guard clause rules above.
- **`else` after `return`/`throw`** — delete it and unindent.
- **Comment-as-crutch** — a comment explaining *what* means the name is wrong.
- **Magic numbers and strings** — name them.
- **Duplicated logic** — extract it, respecting the YAGNI/DRY balance.
- **Vague names** — rename to state intent.
- **Boolean parameters** — `render(true)` says nothing at the call site. Use a named option or two functions.
- **Unrelated changes in the diff** — see Scope discipline.

## Testing

- Structure tests **Arrange / Act / Assert**.
- Name tests by behaviour and condition: "returns empty list when no markets match query", not "works".
- A regression test fails before the fix and passes after. Write it in that order — a test that never failed proves nothing.
- Cover the meaningful edge and failure cases. Each guard clause is a test case; the rejection paths matter as much as the success path.

## Project-specific standards

Project rules override this baseline. Before starting, check for `{project-root}/**/project-context.md`, the repo's own contributing or convention docs, and the linter/formatter/type-checker config. Where they conflict with anything here, they win — see Rule precedence above.