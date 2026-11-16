---
name: pr-precheck
description: Grade a PR package (a candidate pull request read against the plan it claims to implement and the issue that plan belongs to) and decide whether it is ready to submit. Use when checking your own branch, draft PR title, and description before opening the pull request, or when grading an eval package bundle.
---

# pr-precheck: rubric-driven PR grading

<!--
THIS IS THE PART YOU WRITE, and it is the last one: the frame itself.
Weeks 1 through 3 handed you a working SKILL.md and you filled the
files behind it; this week the frame ships as headings, and you write
what it says. The frontmatter above and the section headings below are
fixed (CONTRACT.md's layout rule); the instructions under each heading
are yours. Write instructions to the tool, in the imperative, the way
weeks 1-3's frames spoke to you: what to read, in what order, what to
refuse, what to emit. Your executor in the rotation is the test: a
frame gap they hit (cannot tell what the tool reads, or how a verdict
gets assembled) is a missing sentence here.

One section is not yours: the JSON schema in "Verdict and output" is
reproduced from CONTRACT.md verbatim and may not be altered. Your
words decide everything around it.
-->

## The question

This tool answers: **Is this PR ready to submit?**

A PR package is a candidate pull request (its title, description, commits, and diff) read against the plan it claims to implement, the issue that plan belongs to, and the repository's stated standards and policies. The package contains four artifacts:
1. The plan: the accepted plan from week 3, with its scope, boundary, test plan, and any deviation notes
2. The diff: every committed change the branch makes compared with the repository's default branch
3. The PR text: the candidate title and description
4. The evidence: test results showing before/after behavior and the repository's own checks

The tool grades this single package against the rubric to decide whether all checks pass or any fail, producing a binary verdict: accept (ready to submit) or reject (hold). Grade exactly one package per run, and answer only this question: never review code quality in general, suggest a different design, or grade a different pull request.

## Inputs and modes

**Live mode** (grading your own submission before opening the PR):
- Read your `plan.md` (posted in week 3) with any deviation notes added under `## Deviations`
- Read the diff on your branch: run `git diff main...HEAD` (three dots) from your working copy to show every committed change relative to the repository's default branch
- Read your draft PR title and description from the `pr_draft.md` file you created
- Read your test evidence from `test_evidence.md` (the output of your repository's checks: make test-unit, make test-integration, make lint, make typecheck, and your repro before/after)
- Read the issue from the scoped repo: its title, body, and comments (the thread). Read the PR template at `.github/PULL_REQUEST_TEMPLATE.md` and the contribution policy at `docs/CONTRIBUTING.md` in the same repo.

For a house-chain student (assigned a house issue and branch by the instructor), read the house plan and house branch diff instead of your own posted plan and branch.

**Eval mode** (grading a frozen practice package):
- The eval package is the complete world: every fact comes from the bundle text
- No issue thread is fetched, no repository files are consulted, no GitHub is touched
- Every check runs with the full verdict rule, reading only the plan-context block, the candidate PR, and the repo-facts block in the package
- Eval packages contain repo-facts blocks that describe the repository's PR template and contribution policy as captured on the package's snapshot date

Both modes produce the same set of checks and verdicts via the same rubric and procedure.

## The scope seam (live mode only)

Before starting any check in live mode, read `~/.claude/skills/pr-precheck/scope.md` (the skill's own copy, not a file in the repo) to find the repository scope.

The `Repo:` line in `scope.md` names the pull request's target repository. If the `Repo:` line is still a placeholder (contains `<ORG>/<PATH-REVIEW-REPO>` or similar), stop immediately without grading and tell the student to fill in their section's Path Review repository URL on that line (see `scope.md` for instructions).

If the `Repo:` line is filled with a real repository path, verify that the PR you are grading targets that repository and belongs to the issue named in the PR's description or commits. Refuse to grade any PR that targets a different repository. Never guess a scope: if `scope.md` is unreadable or its `Repo:` line is missing, stop and say so.

In eval mode, `scope.md` is ignored entirely: the bundle is the complete world and needs no scope.

## The voice seam (live mode only)

In live mode, read `~/.claude/skills/pr-precheck/voice-guide.md` last, after the plan, diff, PR text, and test evidence have been read (the procedure's read order puts it at the end). This file contains the student's own rules for how they write upstream, carried forward from week 2.

Hold the draft PR title and description against these rules. Report each rule the draft breaks in the summary text above the JSON block, naming the rule and quoting the offending words. The JSON block records only rubric checks.

Voice rules never change the verdict on their own: they are feedback on communication quality, not gating criteria. A PR can be ready even if the description is terse, or ready while breaking a voice rule. The verdict comes from the rubric alone.

In eval mode, `voice-guide.md` is ignored entirely: voice is personal and carries no gold labels; universal checks live in the rubric.

## Component reads

This tool reads and executes three components. In live mode, all of them, plus `scope.md` and `voice-guide.md`, live in the skill directory `~/.claude/skills/pr-precheck/`, not in the working copy being graded: read them from there by absolute path. In eval mode the harness has already inlined these components and the package bundle into the prompt: read no files and run no commands.

1. **`rubric.md`** defines the checks and the verdict rule. Each row in the checks table names a check, describes the evidence to examine, states the pass condition, and sets a weight (required or preferred). The verdict rule below the table says how the per-check grades combine into accept or reject, and how `unclear` grades are treated.

2. **`references/evidence-guide.md`** maps where each kind of evidence lives in a PR package. For every check named in the rubric, this guide says where in the plan, diff, description, test evidence, or repo facts to find the supporting fact.

3. **`procedure.md`** describes the concrete steps for grading: read order (what to examine before opening checks), evidence gathering (how to pull each fact), check execution (when and how to run each check), and verdict assembly (how grades combine into the final verdict).

When `procedure.md` or `rubric.md` has no content beyond the template comments, the tool refuses to grade and says so, by design: an empty tool that invents checks at runtime is worse than no tool, because its verdicts look like judgment and are noise. A refusal emits no JSON block and invents no checks.

When the procedure is silent on a step (e.g., does not say when to gather a fact or how to handle missing evidence), the tool reports the gap instead of improvising around it. Silence is a sign that a piece is missing.

## Verdict and output

The verdict space is binary: `accept` (ready to submit) or `reject` (hold). There is no third verdict, no score, no "accept with reservations".

End your reply with a fenced JSON block, valid and last, with nothing after it. The schema is fixed and may not be altered:

```json
{
  "item": "<PR URL or bundle id>",
  "checks": [
    {"name": "<check name>", "grade": "pass|fail|unclear",
     "evidence": "<one line: the fact or quote that decided it>"}
  ],
  "verdict": "accept|reject"
}
```

## Grading discipline

Grade under these standing rules:

- **Evidence first.** No check is graded without naming the fact or quote that decided it. "Looks fine" is not evidence.
- **Grade the thing, not the polish.** A terse, complete PR can be ready and a beautiful confident one can hide drift. Every check reads the artifact itself against the plan, the issue, and the stated standards, never the formatting or length of the description.
- **The rubric decides, not the run.** If a check passes by its stated pass condition but feels wrong, it still passes. The fix belongs in the rubric for next time, not in this run.
- **The procedure decides how, not the run.** Follow `procedure.md` as written and report its gaps instead of silently inventing steps.
- **Unclear defaults to fail.** Treat `unclear` as the rubric's verdict rule directs; where the rule is silent, an unverifiable claim is a failing one. A PR you cannot verify from the package is a PR that is not ready to submit.
