# Evidence guide: where evidence lives in a PR package

## Plan fidelity (harness category: silent-drift)

- Where it lives:
  - Eval: the plan-context block's scope line ("Files:", "Not in scope:") and its deliverables; the candidate PR's diff; the candidate description's scope and fidelity claims.
  - Live: plan.md (its Files and scope lines, and its `## Deviations` section); the branch's diff from `git diff main...HEAD`; pr_draft.md's description.
- What good looks like: every changed file and hunk falls inside the plan's stated Files line, or is named in a deviation note with a reason. Every promised deliverable is in the diff or named as deferred. Every fidelity claim in the description is matched by a hunk or plan line.
- Silent drift in the more-than-plan direction: a hunk in a file the plan does not name, a new setting or flag the plan scopes out, a rewrite of a function the plan only touches to add a check.
- Silent drift in the less-than-plan direction: a promised warning, doc, or test missing from the diff with no deferral note, or a description saying "docs now document X" when the diff has no docs hunk.
- An honest deviation re-ties the mismatch: the deviation note in plan.md names the extra or missing item and its reason, and the description repeats the deferral.

## Test evidence (harness category: not-tested)

- Where it lives:
  - Eval: the candidate PR's test-evidence section; the plan's test plan and repro steps in the plan-context block.
  - Live: test_evidence.md (the captured output of your repro before and after, and of make test-unit, make test-integration, make lint, make typecheck); the plan's repro and test plan in plan.md.
- What good looks like: the plan's repro is run before the change and after it, on the same path the change touches, and the after-output shows the expected result stated in the plan. Each repo check the template or plan names is reported with its command and outcome: pasted output in live mode, and in an eval package a stated outcome such as "go test ./... passes" is enough. A failing check is shown with its reason.
- Not-tested shapes that fail:
  - A before/after on an unchanged path (for example, a GET when the fix is a POST).
  - "Tests pass" or "verified locally" with no output pasted.
  - The plan's two-case repro where only the easy case is re-run.
  - A repo check the template names that never appears in the evidence.
- "make test-integration" printing "no tests ran" with its real output counts as run.

## Diff quality (harness category: unreviewable)

- Where it lives:
  - Eval: the candidate PR's diff and commit list.
  - Live: `git diff main...HEAD` and `git log main..HEAD` from the working copy.
- What good looks like: every added line serves the planned change. The fix is visible as its own hunks. No unrelated hunks ride along.
- Debris signals to check line by line:
  - Debug output: print, console.log, eprintln, dbg!, breakpoint().
  - Commented-out code blocks, or an earlier attempt left as comments.
  - Dead functions or unused variables, including ones kept alive with an allow(dead_code) attribute.
  - TODO or FIXME added in the change.
  - Formatting churn: re-indentation or reflowed lines outside the change, and import reorders with no purpose.
  - Duplicate re-printed lines that do nothing.
- Commit messages such as "wip", "fix", or "misc cleanups while debugging" are a signal to read the diff harder. They are not a failure on their own.

## Standards and comms (harness category: standards-wall)

- Where it lives:
  - Eval: the repo-facts block's PR-template asks and stated contribution policy (including any AI-use policy); the candidate description's sections and Testing boxes.
  - Live: `.github/PULL_REQUEST_TEMPLATE.md` and `docs/CONTRIBUTING.md` in the scoped repo; the issue's thread (its comments); pr_draft.md's sections; test_evidence.md for the Testing boxes; for the seeded-bug marker, `git grep -n "issue #<n>:" main -- tests pyproject.toml` on the base against `grep -rn "issue #<n>:" tests pyproject.toml` on the branch.
- What good looks like:
  - Every ask the repo states is answered with substance (prose answering a checklist item counts): live, Summary, Issue, Changes, Testing, and Notes for Reviewers have real content and the Issue line reads "Closes #<n>"; in eval, the repo-facts asks (checklist, closes line, changelog or whatsnew entry) are met.
  - Ticked or claimed Testing items are not contradicted by the evidence.
  - Where the repo states an AI-use policy, the description discloses AI use in the author's own words. Live, Notes for Reviewers says how AI was used and notes any deviation from the plan. Where an eval package's repo states no AI policy, none is required.
  - Any xfail marker or suppression naming the issue is removed by the diff.
  - Maintainer or reviewer direction in the thread is followed or answered.
- Standards-wall shapes that fail: a template section left as placeholder text, a missing or empty Closes line, a disclosure section that is absent or says nothing about AI use, a Testing box ticked with no evidence.
- Whether the description's claims match the diff is plan fidelity (above), not a standards issue.
