# Evidence guide: where evidence lives in a pull request package

Two places to look, depending on the mode. In eval mode, the bundle is the whole world: every fact comes from a section of the bundle file. In live mode, the same facts come from the student's working copy and the repository. Read each family in the order the procedure gives.

## Plan fidelity (harness category: silent-drift)

**Where it lives.**
- Practice package: the "Plan context" section holds the accepted plan: the files it names, its in-scope and not-in-scope lines, the deliverables it promises (code, tests, docs, configuration), and anything it defers. The candidate pull request's "Diff" section shows what actually changed (read the `---` and `+++` file headers, then every hunk). The "Description" and "Commits" sections hold the claims to check against the diff. A deviation note, if any, sits in the plan context or is stated in the description.
- Live: `plan.md` in the working directory, including its `## Deviations` section, is the plan. The diff comes from running `git diff main...HEAD` (three dots) in the working copy, which lists every committed change the branch makes compared with main. The description is `pr_draft.md` (first line is the title). Run `git log main..HEAD --oneline` for the commit list.

**What good looks like.** Every changed file and hunk falls inside the plan's stated boundary or a deviation note, and every deliverable the plan promises is in the diff. Silent drift has two directions: more than the plan (a new option or flag, a renaming pass, reworded unrelated text, a rewrite of a file the plan never touches) or less than the plan (a promised docs update or test that is not in the diff). Either direction is fine only when the plan's deviation notes or the description plainly say what was added, left out, or deferred. A description that claims "exactly as planned", "no changes outside X", or "the docs now document Y" while the diff contradicts it is drift, however polished the description reads.

## Test evidence (harness category: not-tested)

**Where it lives.**
- Practice package: the "Test evidence" section of the candidate pull request, read next to the plan context's test plan and repro evidence, and the repo-facts block's list of the repository's own check commands.
- Live: `test_evidence.md` next to `plan.md`. It holds the before-and-after output of the unit 2 reproduction steps (before on main, after on the branch) and the output of the four checks the Path Review pull request template lists: `make test-unit`, `make test-integration`, `make lint`, and `make typecheck`. Also read the Testing section of `pr_draft.md` for the boxes ticked. Path Review has no integration tests yet, so `make test-integration` ending with "no tests ran" is expected and is honest evidence.

**What good looks like.** The plan's own reproduction steps appear with real output on the code before the change and on the code after it, the two outputs differ the way the plan said they should, and the run exercises the path the diff changes (the same input shape, the same arguments, not a control case that never broke). The repository's own check commands appear by name with their results. A failing check reported honestly with its reason counts as evidence. "Tested locally", "works on my machine", or "tests pass" with no command or output is not evidence. A test added by the diff should check the fixed behavior itself.

## Diff quality (harness category: unreviewable)

**Where it lives.**
- Practice package: the "Diff" section and the "Commits" list of the candidate pull request.
- Live: the output of `git diff main...HEAD` (and `git diff main...HEAD --stat` for the file list), plus `git log main..HEAD --oneline` for commit titles.

**What good looks like.** The fix is visible and nothing unrelated rides along. Every added or removed line serves the planned change. The tells of debris: debug prints or log lines, commented-out code, dead or unused functions or imports, TODO or work-in-progress markers, whitespace-only or formatting-only hunks, reordered imports, lines re-printed unchanged, and commit titles like "wip", "fmt", or "misc cleanups". Any of these buries the change, even beside a correct fix. In live mode, the stat list should show only the files the fix changes: the planning files (`plan.md`, `comment.md`, `pr_draft.md`, `test_evidence.md`) must never appear in the diff.

## Standards and comms (harness category: standards-wall)

**Where it lives.**
- Practice package: the "Repo facts" section states the repository's pull request template asks (sections, how to reference the issue, artifacts required inside the change such as a changelog or release-notes entry) and its contribution policy, including any artificial intelligence disclosure rule. The "Description" section is read against those asks, and the diff's file headers show whether a required artifact is present. Thread highlights show any direction a maintainer gave.
- Live: the repository's `.github/PULL_REQUEST_TEMPLATE.md` and `docs/CONTRIBUTING.md` hold the stated asks. The filled-in `pr_draft.md` is read against them. The issue thread shows maintainer direction.

**What good looks like.** Every template section has real content (or an explained "N/A" with the reason), the issue is referenced the way the template asks (for Path Review, `Closes #` followed by the issue number), the Testing boxes ticked are true (the continuous integration box stays unticked, since it only runs after the pull request opens), and every artifact the repository requires inside the change is in the diff. Where the contribution policy requires disclosure of artificial intelligence use, the description says in the author's own words how it was used (for Path Review, this goes under Notes for Reviewers). Boilerplate, blank sections, and placeholder text left in place are a visibly ignored ask. Whether the description's claims match the diff is plan fidelity, above, not this family.
