# Unit 4: pull request and pr-precheck

## Pull request

https://github.com/codepath/pathreview-ai301-fa26-s3/pull/284

## Branch

`fix/24-role-switch-whitespace` (type `fix`, issue `24`, short description `role-switch-whitespace`)

## Run history

Ordered agreement scores, oldest first. Only runs 4 and 5 graded all 20 packages in the same run.

1. Full run, stopped before finishing: 9 of 20 packages graded, all 9 as reject, including gold-accept pkg-02 and pkg-05. No score; `eval-run.txt` not written.
2. Partial run (pkg-02, pkg-05, pkg-08, pkg-11): 1/4. The three clear accepts failed on ai-disclosure, repo-checks-run, repro-before-after, template-sections, and testing-boxes-match-evidence.
3. Partial run after rewriting those five checks (pkg-01, 02, 03, 04, 05, 08, 12, 20): 8/8.
4. Full run: 9 packages graded and agreed (9/9); the other 11 returned `claude exited 1` with no verdict, so the run has no valid score. Not saved.
5. Full run with `--workers 3`: **20/20** (bar 18/20: PASS). Categories: clear-accept 7/7, not-tested 4/4, silent-drift 4/4, standards-wall 2/2, unreviewable 3/3. This is the run in `eval-run.txt`.

## Package analysis

**pkg-03** (starship/starship#7648). Gold: reject (silent-drift). My tool: reject, agreement yes.

The plan scopes the change to `src/modules/directory.rs` plus one module test and rules out any config change. The diff also rewrites `truncate()` and adds a `resolve_symlinks` option in `src/configs/directory.rs`, while the description says it "implements the posted plan exactly" and makes "No other behavior changes". The tool failed four checks on this: plan-scope-match (extra files, no deviation note), deliverables-present (the promised module test is not in the diff), description-fidelity (the "exactly" claim is contradicted by the diff), and no-debris (unrelated truncate churn plus an unused field). The repro, repo checks, and template checks passed, which is the point of the package: the evidence was good and the PR still was not ready.

## Check rationale

Check: `ai-disclosure`. Quoted from `tools/pr-precheck/rubric.md`: "Where the repo states no AI policy (eval), pass with the evidence \"no stated AI policy\"; do not invent one."

My first version failed any description without an AI statement. That rejected gold-accept pkg-02, pkg-05, and pkg-08, whose repos (Rich, nushell, lazygit) state no AI policy. The gold labels only reject for missing disclosure where the repo requires it (pkg-20, ghostty), so the check now keys on the repo's stated policy. In live mode it still requires Notes for Reviewers to say how AI was used, because the course asks for it.

## Trade-offs

The `ai-disclosure` check now gives up catching undisclosed AI use in a repo that states no policy: a PR to such a repo passes even if it says nothing about AI. I accepted that because inventing a policy the repo never stated rejected three correct PRs. A related loosening: `repo-checks-run` accepts a stated outcome such as "go test ./... passes" in eval packages without pasted output, so a fabricated claim would pass that check. The repro check, which does require observable before/after output, is the guard against that.
