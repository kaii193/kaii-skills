---
name: kaii-design-principles
description: Ten system-design and architecture principles that sit above the line-level baseline — code style conformance, SOLID, testability, abstraction level, design-pattern restraint, avoiding global state/side effects, and continuous refactoring. MANDATORY — consult this on every coding-related request (writing, refactoring, debugging, designing, or reviewing code), together with kaii-coding-standards, even when the task looks purely line-level or the user doesn't mention design/architecture at all. Line-level rules (naming, comments, guard clauses, error handling, security, testing mechanics) live in kaii-coding-standards — this skill defers to it rather than repeating it, but both are always in scope together.
---

# Design Principles (10)

Ten principles for structuring code, not just writing individual lines. Use this when the question is "how should this be shaped" rather than "is this line clean." For line-level rules — naming, comments, guard clauses, error handling, security, test mechanics — see `kaii-coding-standards`; this skill cross-references rather than duplicates those.

**When implementing**, run through these before writing the first class/function signature — they're cheaper to apply at design time than to retrofit. **When reviewing**, use them to judge shape and structure, separately from the line-level checklist.

## 1. Follow code specifications

Conform to the established style guide for the language/ecosystem in use (PEP 8, Google Java Style Guide, Airbnb JS, etc.) rather than a personal preference.

- Check for a linter/formatter config in the repo first — it is the enforceable version of the style guide and wins over any general guidance here (see `kaii-coding-standards` → Rule precedence).
- If no config exists and the task adds meaningful new code, say so rather than silently picking a style — this is a project-config gap, not something to solve unilaterally.
- Consistency beats personal taste: match what the surrounding file already does even if a different valid style exists.

## 2. Documentation and comments

Covered in depth in `kaii-coding-standards` → "Self-documenting code over comments." The one line that matters at design time: comments and docs explain **why**, not what; a comment that restates the code is a maintenance liability, not documentation. Keep both current — a stale comment is worse than none, because it actively misleads.

## 3. Robustness

Design for the inputs and failure modes the code will actually encounter — not an infinite hypothetical space (see YAGNI in `kaii-coding-standards`), but every boundary where untrusted or unpredictable data enters: user input, network responses, file reads, third-party APIs.

- Identify the trust boundaries in the design up front; decide there what gets validated and what gets trusted inward.
- Prefer failing loud and specific (a typed error, a clear rejection) over failing silent or producing a plausible-looking wrong answer.
- Detailed exception-handling mechanics (catch specificity, message content, re-raising) are in `kaii-coding-standards` → Error handling.

## 4. Follow SOLID

- **Single Responsibility** — a class/module has one reason to change. If describing its job needs "and," it's two responsibilities; split them.
- **Open/Closed** — extend behavior by adding new code (new subclass, new strategy, new handler), not by editing a working function's internals for every new case. A growing `if/else` or `switch` on a type code is the usual tell.
- **Liskov Substitution** — a subtype must be usable anywhere its base type is expected, without the caller needing to know which one it got. If a subclass throws on a method the base class documents as safe, or narrows what it accepts, it violates this.
- **Interface Segregation** — many small, specific interfaces beat one large one that forces implementers to stub out methods they don't need.
- **Dependency Inversion** — depend on an abstraction (interface, protocol, injected collaborator), not a concrete implementation, for anything that varies or needs to be tested in isolation. High-level policy code shouldn't import low-level detail directly.

Apply SOLID where the code actually has more than one reason to vary or needs to be tested in isolation. Do not apply it reflexively to a script that will run once — see principle 7.

## 5. Make testing easy

Design for testability rather than bolting tests on after.

- Keep components small enough that a unit test can set up their inputs without a large fixture.
- Push side effects (I/O, time, randomness, network) to the edges; keep core logic as pure functions of their inputs so tests don't need mocks to exercise the interesting branches.
- If a function is hard to test, that's usually a design signal (too many responsibilities, a hidden dependency) — fix the shape rather than reaching for heavier test tooling.
- Test-writing mechanics (naming, AAA structure, coverage of edge cases) are in `kaii-coding-standards` → Testing.

## 6. Abstraction

Extract the stable, conceptual core and hide the volatile implementation detail behind it — but only to the depth the current requirement justifies.

- Under-abstraction: the same decision or logic is duplicated in multiple places, so a change has to happen N times and will eventually be missed in one.
- Over-abstraction: an interface, factory, or config layer built for a second implementation or a future case that doesn't exist yet. This is YAGNI territory — see `kaii-coding-standards` → Principles.
- A good abstraction hides *how*, not *that* — the caller should be able to state what the abstraction does without knowing its internals, but shouldn't have to guess what it does at all.

## 7. Use design patterns — don't over-design

A pattern is justified by a problem already present, not by a wish to look designed.

- Name the specific problem the pattern solves *here* (e.g., "callers need to swap the payment strategy at runtime") before reaching for it. If you can't name it, the pattern is decoration.
- Prefer the plainest structure that solves today's problem. A pattern adds indirection; that indirection has to earn its cost in actual flexibility used, not flexibility imagined.
- Watch for pattern-shaped code applied where a plain function or a data structure would do — a `Factory` for one concrete type, a `Strategy` with only one strategy, a `Singleton` used to smuggle in global state (see principle 8).

## 8. Reduce global dependencies

Prefer localized state and explicit parameter passing over globals, module-level mutable singletons, or ambient context.

- A function's dependencies should be visible in its signature. A function that silently reaches into a global, a shared singleton, or ambient config is harder to test, harder to reason about, and unsafe to call concurrently.
- Prefer pure functions (output determined by input, no side effects) for logic; isolate the side-effecting shell around them (see principle 5).
- Where shared state is genuinely required (a cache, a connection pool), inject it explicitly rather than importing a global instance, so callers and tests can control what it points to.
- This is a design-time principle, separate from immutability of individual values, which is covered in `kaii-coding-standards` → Immutability.

## 9. Continuous refactoring

Treat structural cleanup as part of delivering the change, not a separate task to schedule later — but only within what the current task touched (see `kaii-coding-standards` → Scope discipline; this is not license to refactor unrelated code without asking).

- When a change reveals that a design decision (principle 4, 6, or 7 above) no longer fits, note it — fix it now if it's inside the diff's scope, otherwise record it as a follow-up rather than letting it silently rot.
- Refactoring changes structure without changing behavior. If a "refactor" changes what the code does, it's not a refactor — split it into its own reviewable step.
- Small, frequent structural corrections are cheaper than a big rewrite later; that's the argument for doing this continuously rather than deferring it.

## 10. Security is a top priority

Full detail in `kaii-coding-standards` → Security (secrets handling, parameterized queries, output encoding, authorization placement, dependency risk). At the design level, the addition is: **decide the trust boundary and the authorization model as part of the design**, not as a check bolted on after the feature works — retrofitting authorization or input validation after the shape is set is where vulnerabilities get missed.

## Precedence

Same order as `kaii-coding-standards`: repo tool config → project documentation → this baseline → language/framework idiom. Where a design principle here would conflict with an explicit project convention (e.g., the project deliberately avoids DI containers), the project convention wins.
