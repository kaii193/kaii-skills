# Worklog and definition of done

Applies to every task that changes code. The goal is that a second person — or you next week — can reconstruct what happened and re-run the verification without asking a question. "It works, I checked" is not a result.

The worklog is not a diff summary. Git already records what changed, more accurately and for free. The worklog records what git cannot: what the change was *supposed* to achieve, what was decided and rejected along the way, and the evidence that it actually works.

## 1. Write the success criteria before implementing

Criteria written afterwards only describe whatever the code happens to do. Write them first, from the requirement, and treat them as fixed once written. Each criterion must be:

- **Objective** — passes or fails on evidence, no judgement call. "Login rejects an expired token with 401" is a criterion; "auth feels solid" is not.
- **Observable** — someone else can check it by running a command or following listed steps.
- **Attributable** — tied to this change, not to the application being healthy in general.

If the requirement is too vague to produce criteria, stop and ask before writing code. That ambiguity does not get cheaper later.

## 2. Log the work to a file

One file per task: `docs/worklogs/YYYY-MM-DD-<short-slug>.md`.

````markdown
# <Task title>

Date: <YYYY-MM-DD> · Branch: <branch> · Status: DONE | PARTIAL | BLOCKED

## Goal
<One sentence: what this change is supposed to achieve.>

## Success criteria
- [ ] C1: <objective, checkable statement>
- [ ] C2: …

## Decisions and trade-offs
- <Chose X over Y because …>
- <Known limitation, accepted deliberately: …>

## How to verify
```bash
<exact commands, copy-pasteable>
```
Manual steps (only for what cannot be automated):
1. <step> → expected: <observable result>

## Result
<Raw output of the commands above — pasted, not summarised.>

- C1 — PASS: <which line of that output demonstrates it>
- C2 — FAIL: <what happened instead, and why>

## Not done / follow-ups
- <deferred item + reason>
````

Keep it short. This is a record, not a report — bullets over prose. Do not list changed files; that is what the diff is for.

## 3. How to verify

Pick the strongest option available. Dropping a level is allowed, but say why in the log.

1. **Automated test** — an assertion that fails before the change and passes after. The default. Cover each guard clause's rejection path, not just the happy path.
2. **Reproducible command** — a script, `curl`, or CLI invocation with the expected output written down. For behaviour that is not unit-testable: migrations, build output, CLI ergonomics.
3. **Manual steps** — numbered, each with its expected observable result. Last resort, only for what a machine genuinely cannot check, such as visual layout or a third-party sandbox flow.

Run the project's gates alongside the task-specific check — typically test, lint, and type-check. A change is not verified by its own test alone if it broke someone else's.

## 4. What counts as success

Status is mechanical, not a judgement:

- **DONE** — every criterion PASS with evidence in the log, and the project gates green. Nothing else qualifies.
- **PARTIAL** — some criteria pass. Name the failures and why, in Result. The log still gets written.
- **BLOCKED** — verification is impossible right now. State exactly what is needed to unblock: a credential, an environment, an upstream fix, a decision.

### Evidence, not assertion

The Result section holds **pasted command output**. `PASS: 12 tests passed` is a claim; a terminal block showing the test names and the exit status is evidence. Where output is long, paste the summary lines and the full output of anything that failed — never a paraphrase of a passing run you did not watch.

If the commands were not executed, the status is **BLOCKED**, not DONE. An unrun test is not a passing test.

### Do not edit criteria to fit the outcome

If a criterion turns out to be wrong, unachievable, or based on a misreading of the requirement, say so explicitly in the log and get agreement before changing it. Silently rewriting it into something the code already satisfies converts the whole exercise into theatre.

## 5. Close out

Report the status and the worklog path back to the user at the end of the task. If the status is PARTIAL or BLOCKED, lead with that rather than with what did work.