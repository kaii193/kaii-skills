---
name: gf-start-coding
description: The mandatory baseline for writing, refactoring, debugging, or reviewing code in any project — naming, guard-clause control flow, error handling, security, type safety, testing, scope discipline, the absolute zero-comments/no-JSDoc/no-docstring rule, system-design principles (SOLID, testability, abstraction, avoiding global state, continuous refactoring), which actions require asking the user first, and the required worklog workflow (success criteria before implementing, verification, what counts as done). Consult this on every coding-related request, even when the task looks purely line-level and the user doesn't mention standards at all. Git branch, commit, and PR conventions live in the separate gf-start-code-commit skill.
---

# Start Coding

The shared quality floor and design discipline for every implementation and review. Language-agnostic — framework and pattern specifics belong in separate skills layered on top.

**When implementing**, treat each rule as a guardrail while writing, and run the design principles before writing the first class/function signature — they're cheaper to apply at design time than to retrofit. **When reviewing**, treat each as a checklist item.

## Read next

This file covers how code should be written. The surrounding workflow lives elsewhere:

| Read | When |
|---|---|
| `references/worklog.md` | Starting any task that changes code. Covers success criteria, the worklog file, verification, and what counts as done. Read it **before writing code** — the criteria come first. |
| **`gf-start-code-commit` skill** | Naming a branch, writing a commit message, or opening and reviewing a pull request. Separate skill, not part of this one. |

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

## No comments, no JSDoc, no docstrings — absolute rule

Write no comments. None. Not a single line comment, block comment, JSDoc/TSDoc annotation, or docstring, in any language, in any file, ever — including on exported/public functions, classes, and types. There is no exception for "why," constraints, trade-offs, or workarounds. This rule holds even when nothing else here gets loaded.

The name is the documentation. A comment restating the code is a maintenance liability — it goes stale, and it signals that the code failed to communicate. **Fix the name, don't add the comment.**

When tempted to write an explanatory comment, do this instead:

1. **Rename** the variable or function so the intent is obvious.
2. **Extract** the confusing block into a well-named function — the name replaces the comment.
3. If a genuine **why** needs recording — a workaround, a rejected alternative, a non-obvious trade-off — put it in the **commit message** or the **PR/worklog description**, never in the source file.

Two patterns this specifically replaces:

- A comment describing **what a condition means** — extract the condition into a named boolean instead. The name travels with the value.
- A comment used as a **section header inside a long function** — each header is a function waiting to be extracted. The body then reads as a list of named steps.

**Never**: commented-out code, changelog or authorship comments (dates, names, ticket history — that is the commit log's job), a comment narrating the next line, a doc comment on a public/exported API, or a language docstring (Python `"""..."""`, Go doc comments, etc.) — the signature and types carry the contract. Applies to every language and every project, not just the one you're currently in.

If an exported function's contract isn't obvious from its name and types, that's a naming or type-design problem — fix the signature, don't paper over it with a doc comment.

A project's own linter rule requiring doc comments on public APIs (rare, but possible for a published library) wins over this rule per precedence above — flag the conflict to the user rather than silently picking one.

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
- Where in-place mutation is a deliberate performance decision, keep it local to the function and record why in the commit message or PR/worklog description — not a comment in the file.

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
- Decide the trust boundary and the authorization model as part of the design, not as a check bolted on after the feature works — retrofitting authorization or input validation after the shape is set is where vulnerabilities get missed.

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

## Design principles

Principles for structuring code, not just writing individual lines — use these when the question is "how should this be shaped" rather than "is this line clean." (Documentation/comments is covered above under "No comments, no JSDoc, no docstrings" rather than repeated here.) Run through them before writing the first class/function signature; they're cheaper to apply at design time than to retrofit. When reviewing, use them to judge shape and structure, separately from the line-level checklist above.

### 1. Follow code specifications

Conform to the established style guide for the language/ecosystem in use (PEP 8, Google Java Style Guide, Airbnb JS, etc.) rather than a personal preference.

- Check for a linter/formatter config in the repo first — it is the enforceable version of the style guide and wins over any general guidance here.
- If no config exists and the task adds meaningful new code, say so rather than silently picking a style — this is a project-config gap, not something to solve unilaterally.
- Consistency beats personal taste: match what the surrounding file already does even if a different valid style exists.

### 2. Robustness

Design for the inputs and failure modes the code will actually encounter — not an infinite hypothetical space (see YAGNI above), but every boundary where untrusted or unpredictable data enters: user input, network responses, file reads, third-party APIs.

- Identify the trust boundaries in the design up front; decide there what gets validated and what gets trusted inward.
- Prefer failing loud and specific (a typed error, a clear rejection) over failing silent or producing a plausible-looking wrong answer.

### 3. Follow SOLID

- **Single Responsibility** — a class/module has one reason to change. If describing its job needs "and," it's two responsibilities; split them.
- **Open/Closed** — extend behavior by adding new code (new subclass, new strategy, new handler), not by editing a working function's internals for every new case. A growing `if/else` or `switch` on a type code is the usual tell.
- **Liskov Substitution** — a subtype must be usable anywhere its base type is expected, without the caller needing to know which one it got. If a subclass throws on a method the base class documents as safe, or narrows what it accepts, it violates this.
- **Interface Segregation** — many small, specific interfaces beat one large one that forces implementers to stub out methods they don't need.
- **Dependency Inversion** — depend on an abstraction (interface, protocol, injected collaborator), not a concrete implementation, for anything that varies or needs to be tested in isolation. High-level policy code shouldn't import low-level detail directly.

Apply SOLID where the code actually has more than one reason to vary or needs to be tested in isolation. Do not apply it reflexively to a script that will run once — see principle 7 below.

### 4. Make testing easy

Design for testability rather than bolting tests on after.

- Keep components small enough that a unit test can set up their inputs without a large fixture.
- Push side effects (I/O, time, randomness, network) to the edges; keep core logic as pure functions of their inputs so tests don't need mocks to exercise the interesting branches.
- If a function is hard to test, that's usually a design signal (too many responsibilities, a hidden dependency) — fix the shape rather than reaching for heavier test tooling.

### 5. Abstraction

Extract the stable, conceptual core and hide the volatile implementation detail behind it — but only to the depth the current requirement justifies.

- Under-abstraction: the same decision or logic is duplicated in multiple places, so a change has to happen N times and will eventually be missed in one.
- Over-abstraction: an interface, factory, or config layer built for a second implementation or a future case that doesn't exist yet. This is YAGNI territory.
- A good abstraction hides *how*, not *that* — the caller should be able to state what the abstraction does without knowing its internals, but shouldn't have to guess what it does at all.

### 6. Use design patterns — don't over-design

A pattern is justified by a problem already present, not by a wish to look designed.

- Name the specific problem the pattern solves *here* (e.g., "callers need to swap the payment strategy at runtime") before reaching for it. If you can't name it, the pattern is decoration.
- Prefer the plainest structure that solves today's problem. A pattern adds indirection; that indirection has to earn its cost in actual flexibility used, not flexibility imagined.
- Watch for pattern-shaped code applied where a plain function or a data structure would do — a `Factory` for one concrete type, a `Strategy` with only one strategy, a `Singleton` used to smuggle in global state (see principle 7 below).

### 7. Reduce global dependencies

Prefer localized state and explicit parameter passing over globals, module-level mutable singletons, or ambient context.

- A function's dependencies should be visible in its signature. A function that silently reaches into a global, a shared singleton, or ambient config is harder to test, harder to reason about, and unsafe to call concurrently.
- Prefer pure functions (output determined by input, no side effects) for logic; isolate the side-effecting shell around them (see principle 4 above).
- Where shared state is genuinely required (a cache, a connection pool), inject it explicitly rather than importing a global instance, so callers and tests can control what it points to.

### 8. Continuous refactoring

Treat structural cleanup as part of delivering the change, not a separate task to schedule later — but only within what the current task touched (see Scope discipline above; this is not license to refactor unrelated code without asking).

- When a change reveals that a design decision (principle 3, 5, or 6 above) no longer fits, note it — fix it now if it's inside the diff's scope, otherwise record it as a follow-up rather than letting it silently rot.
- Refactoring changes structure without changing behavior. If a "refactor" changes what the code does, it's not a refactor — split it into its own reviewable step.
- Small, frequent structural corrections are cheaper than a big rewrite later; that's the argument for doing this continuously rather than deferring it.

### 9. Security by design

Full mechanics are in the Security section above. At the design level, the addition is: **decide the trust boundary and the authorization model as part of the design**, not as a check bolted on after the feature works.

## Project-specific standards

Project rules override this baseline. Before starting, check for `{project-root}/**/project-context.md`, the repo's own contributing or convention docs, and the linter/formatter/type-checker config. Where they conflict with anything here, they win — see Rule precedence above.
