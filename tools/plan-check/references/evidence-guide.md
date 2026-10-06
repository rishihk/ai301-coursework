# Evidence guide: where evidence lives in a plan package

## Diagnosis and grounding

**Where it lives.** In a practice package: the plan's "cause" or "diagnosis" statement, and the repro-evidence block (commands plus pasted output). Live: the draft plan.md diagnosis section, and the repro comment you posted on the issue thread.

**What good looks like.** The cause names something the repro output shows (an error line, a diff, a missing file, a "no match"), and the planned fix removes that cause rather than hiding the symptom. A diagnosis is bad if the repro output never mentions the thing it blames, or if it blames a different thing than the output points at.

## Scope

**Where it lives.** In a package: the plan's "in scope" and "not in scope" lines and its list of files. The issue context holds what was asked. Live: plan.md scope section and the issue body.

**What good looks like.** One change a reviewer could read in one pull request, matching what the issue asks. Extra things the author noticed are named as separate work, not folded in. A drive-by rewrite adds files or features the issue never mentioned.

## Executability

**Where it lives.** In a package: the plan's files, approach, and order-of-work lines, plus the repo-facts block (which tools and files exist). Live: plan.md approach and files sections, and the repo checked out in the fork.

**What good looks like.** Every file or area is named, and each says what will be done there, so a stranger could start step one without asking. Nothing depends on a file, tool, secret, or access that the evidence says does not exist.

## Test plan

**Where it lives.** In a package: the plan's test-plan lines, read next to the repro-evidence block's steps. Live: plan.md test plan, next to the unit 2 reproduction comment.

**What good looks like.** The test plan re-runs the repro's own steps (or a check on the same behavior) and names the output that proves the fix, for example "alembic check exits 0" or "the CI job fails on the broken migration." It would fail before the change and pass after. "Run the tests" with no named result is not decisive.

## Honesty

**Where it lives.** In a package: the plan's risks and unknowns lines, plus any claims in the diagnosis and approach. Live: plan.md risks and unknowns, and the Deviations heading at the end of plan.md.

**What good looks like.** Open questions are written as open ("I have not checked whether..."). Facts are quoted from repro output or code. A guess written as a fact, or "no risks", is false confidence. If the build changes the plan, the change is recorded under Deviations.

## Comms

**Where it lives.** In a package: the plan comment, read next to the thread highlights (what maintainers said) and the repo-facts block (branch and commit rules, comment rules, AI-use policy). Live: comment.md, the issue thread, and docs/CONTRIBUTING.md in the repo.

**What good looks like.** The comment answers what maintainers actually said in this thread and follows the repo's stated rules. If the repo requires AI-use disclosure for comments, the comment discloses it. If the repo states no rule for comments, nothing is required. Boilerplate that could be pasted on any issue is not thread-aware.
