---
name: audit
description: Careful cleanup audit of a codebase. Independent reviewers look for dead code, duplication and over-engineering, a skeptic checks every finding, then approved fixes are applied in small verified commits. Use when the user asks to audit, clean up, de-bloat, simplify or remove redundancy across the project (not just the current diff). Args - optional path(s) to limit scope, "quick" for a single-pass review, "report" to stop after the report.
---

# Audit

Goal: smaller, clearer code that behaves exactly the same. Fewer findings that are all true beat
many findings that are half true. Never trade correctness for line count.

This skill is project-agnostic. Everything project-specific comes from the project's own files.

## 1. Learn the project (you, not a subagent)

- Read `CLAUDE.md`, `README.md` and any docs they point to. Note:
  - **Protected content**: anything the project says not to change without explicit permission
    (content, data, copy, public APIs, config). Reviewers may flag it but never change it.
  - **Check commands**: how to run, build, lint and test. If none exist, work out the cheapest
    honest check (syntax check, starting the app, loading the page) and say what you chose.
- List source files with line counts. Skip generated, vendored and gitignored files.
- Run the checks once now and record the result as the baseline. If the baseline already fails,
  tell the user and stop unless they say to continue.

Scope: the paths given as args, otherwise the whole repo.

## 2. Independent review

If the scope is under ~1,500 lines or the user said "quick", do one careful pass yourself
covering all three lenses below, then go to step 3.

Otherwise launch three reviewer subagents **in parallel, in one message**. Each gets the scope,
the protected-content list and its own lens only. They work independently and do not see each
other's output (this avoids groupthink). They read and search only; they never edit.

- **Dead weight**: unused functions, variables, CSS rules, files, imports, params and branches
  that can't run; commented-out code; stale TODOs; leftovers from removed features. Must prove
  "unused" by searching every place it could be referenced (including HTML, strings, dynamic
  lookups, build scripts).
- **Duplication and over-engineering**: copy-pasted logic, near-identical functions, wrappers
  that add nothing, abstractions with one caller, config for things that never vary, defensive
  checks for impossible cases, verbose code with a simpler standard equivalent.
- **Clarity and consistency**: confusing names, functions doing too much, inconsistent patterns
  for the same job, comments that restate the code or are now wrong, magic values repeated in
  several places.

Each finding must include: `file:line`, what it is, evidence (the search done, the callers
found), the proposed change, lines saved (estimate), and risk (low / medium / high).
Ask for at most ~15 findings per reviewer, best first. Taste-only nits are not findings.

## 3. Skeptic pass

Merge and de-duplicate the findings. Then have one fresh subagent (or yourself in quick mode)
play the skeptic: for every finding, try to prove it wrong.

- Is it really unused? Search again, including dynamic access and other entry points.
- Would the change alter behavior, output, appearance or edge cases in any way?
- Does it touch protected content?
- Is the "simpler" version actually simpler, or just shorter / more clever?
- Is this duplication deliberate (e.g. kept separate because the cases will diverge)?

Verdict per finding: **keep**, **needs user decision** (with the question), or **reject**
(with the reason). Only kept findings move on.

## 4. Report and wait

Show the user a short report:

- Baseline result and scope.
- **Safe fixes** (kept): one line each with `file:line`, change, lines saved.
- **Needs your call**: the question for each.
- **Rejected**: a count, plus any notable one and why (so the user sees what was considered).
- Total estimated lines removed.

If the user said "report", stop here. Otherwise ask which fixes to apply (default suggestion:
all safe fixes). Do not edit before the user answers.

## 5. Apply, one theme at a time

- Group approved fixes into small themed batches (e.g. "remove unused CSS", "merge duplicate
  render helpers"). One commit per batch with a plain-language message.
- After each batch run the checks. If anything fails or behaves differently, revert that batch,
  note it, and move on. Never weaken or delete a test or check to make it pass.
- Make only the approved change. No drive-by rewrites, reformatting or new features.
- If the project keeps a decision log or changelog, add one entry summarizing the audit.

## 6. Final summary

Lines before and after per file, batches applied, batches reverted (and why), open questions,
and anything noticed but out of scope. Keep it short.
