# Procedure: how this tool grades a PR package

## Read order

1. Live mode only: read scope.md first. If its Repo line still contains `<ORG>` or `<PATH-REVIEW-REPO>`, stop and output only the message: "Fill the Repo: line in scope.md with your Path Review repo, then re-run." Emit no JSON block. If the Repo line is filled, note the repo.
2. Confirm the rubric and procedure are filled. If rubric.md's checks table or verdict rule is missing, or procedure.md has any section still holding only template comments, refuse to grade and say which file is empty. Emit no JSON block.
3. Read the plan. Live: plan.md, including its `## Deviations` section if present. Eval: the plan-context block. Note the issue number, the Files line, the "not in scope" line, the repro steps, the test plan, and every deliverable the plan lists. Record each deviation note.
4. Read the issue. Live: fetch the issue from the scoped repo (title, body, comments). Eval: the issue block and thread highlights. Note any maintainer or reviewer direction.
5. Read the standards. Live: `.github/PULL_REQUEST_TEMPLATE.md` and `docs/CONTRIBUTING.md` from the scoped repo. Eval: the repo-facts block. Note each required section, each required check, and any disclosure ask.
6. Read the diff. Live: run `git diff main...HEAD` from the working copy. Eval: the candidate PR's diff. Read every changed file and hunk. Do not read the description yet.
7. Read the candidate PR text. Live: pr_draft.md (title on line one, then the description). Eval: the candidate title and description.
8. Read the test evidence. Live: test_evidence.md. Eval: the candidate PR's test-evidence section.
9. Live mode only, last: read voice-guide.md and hold the title and description against it. Record any broken rule for the summary. Do not use it for any grade.

## Evidence gathering

Gather these pairings. Write the fact and its location for each one before grading.

- Plan against diff (plan-scope-match, deliverables-present): list every changed file and hunk from the diff. For each, record whether the plan's Files line, scope line, or a deviation note covers it. Separately, list every deliverable the plan names and find it in the diff or in a deviation note.
- Description against diff and plan (description-fidelity, deferrals-disclosed): list every sentence in the title and description that claims scope, fidelity, or behavior (for example "exactly as planned", "no changes beyond", "now renders"). For each, find the supporting hunk or plan line, or record that the diff contradicts it.
- Repro against test evidence (repro-before-after): take the repro steps from the plan's repro and test plan. Find the before and after output for those exact steps in test_evidence.md. Confirm the after-run is on the changed path.
- Repo checks against test evidence (repo-checks-run): list the checks the PR template names: make test-unit, make test-integration, make lint, make typecheck. Find each one's captured output in test_evidence.md. Note any that is missing or failed.
- Diff against debris signals (no-debris): scan every added line for print, console.log, eprintln, dbg!, lines beginning with comment markers that contain code, TODO or FIXME, functions with no callers, and whitespace-only or import reorder hunks. Record each line number and what it is.
- Template against description (template-sections): list each template heading. For each, confirm it has real text in the description. Confirm the Issue line reads "Closes #<n>" with the plan's issue number.
- Testing boxes against evidence (testing-boxes-match-evidence): list each ticked box in the description. For each, find its matching output in test_evidence.md.
- Disclosure (ai-disclosure): find the Notes for Reviewers section. Record whether it states how AI was used.
- Seeded markers (seeded-marker-removed): live: run `grep -rn "issue #<n>:" tests` and search pyproject.toml for the issue number. Record each match. The diff should remove each one. Eval: check the diff and plan for any xfail marker the plan names.
- Thread (thread-direction-engaged): list each maintainer or reviewer direction in the thread. For each, find where the diff or description addresses it.

## Check execution

- Run the checks in rubric order. Each check gets one grade (pass, fail, or unclear) and one evidence line naming the fact or quote that decided it.
- If the evidence for a check is absent from the package, grade it unclear and say what was missing in the evidence line. Do not infer the evidence from a neighbouring check.
- A check that depends on another check's finding may cite that finding, but must still name its own evidence.
- Do not re-read the whole package for each check. Use the evidence gathered in the previous section.
- Grade the artifact, not its polish. Length, formatting, and commit count are not evidence for any check.
- Do not add checks, drop checks, or change weights at runtime. If a check needs a fact the procedure does not gather, record the gap in the summary and grade that check unclear.

## Verdict assembly

- Apply the rubric's verdict rule to the grades. Accept if every required check passes. Reject if any required check fails or is unclear. Preferred checks never change the verdict.
- Pick the deciding check. On reject, it is the first required check in rubric order whose grade is fail or unclear. On accept, it is the first required check in rubric order (plan-scope-match).
- Name the deciding check in the summary text above the JSON block, with its evidence line quoted. The JSON schema has no field for it, so its evidence stays in the checks array unchanged.
- Output the per-check summary in rubric order, then the fenced JSON block as the last thing in the reply. The item is the PR URL (live) or the bundle id (eval). Nothing follows the block.
- Live mode, after the JSON is decided: list any voice-guide rule the title or description breaks, as text above the block. Do not change the verdict for it.
- The same grades always produce the same verdict. Do not weigh the checks differently on a second pass.
