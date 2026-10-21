# Unit 4 — Test and Submit

Path: `beat-1-sandbox/unit-4/pull-request.md`

Record of the pull request you opened against the Path Review repo, and of the evaluation
runs that produced `eval-run.txt`. This file is graded at the path above; a copy kept
anywhere else in the repository is not read.

Complete every labelled field below. Each is graded on its own; content placed under the wrong
label is not graded.

---

## Your pull request

**Pull request**

https://github.com/codepath/pathreview-ai301-fa26-s1/pull/234

**Branch**

feat/51-ci-migration-check

## Eval iterations

Answer all four sections. Quote source text directly; paraphrase does not satisfy these
fields.

**Run history**

1. 0 of 0 scored. All 20 packages errored with `claude exited 1` because my Claude Code login had expired. I logged back in. This was not a graded run.
2. 16/20 (first real full run). It missed four clear accepts: pkg-05, pkg-11, pkg-16, pkg-19.
3. 11/11 (partial `--only` re-grade of pkg-05, pkg-11, pkg-16, pkg-19 plus canaries pkg-01, pkg-03, pkg-04, pkg-07, pkg-10, pkg-14, pkg-17, after loosening two checks).
4. 19/20, the confirming full run saved in `eval-run.txt`: `agreement: 19/20 scored items  (bar: 18/20: PASS)`.

**Package analysis**

Package: pkg-17 (category silent-drift, source vuejs/core#14893).

Gold label: reject. The gold note reads: "the plan promises a dev warning AND a docs update; the diff contains only the warning while the description claims the docs 'now document the limitation'".

My rubric: reject, so we agree. It failed three checks, with these evidence lines from my tool's output:
- diff-matches-plan: Plan promises "(2) update the reactivity-core documentation page"; diff touches only reactive.ts and reactive.spec.ts, and nothing discloses the omission.
- description-matches-diff: Description says "Implements the plan on #14893 in full" and "the reactivity-core documentation now documents the limitation", but the diff has no documentation change.
- repro-before-after: Only after-change warning output is shown for the primary repro; no before output, and the plan's docs-render step is not shown.

Why it read it that way: the package is built so the polished description hides the gap. My diff-matches-plan check requires every deliverable the plan promises to appear in the diff, and it only forgives a missing one when the plan or the description plainly discloses it. Here nothing discloses it, and the description claims the opposite, so description-matches-diff fails too. That is the same rule that lets pkg-13 and pkg-16 pass: their deferrals are stated in the plan and restated in the description, so they count as honest shortfalls. Claiming more than the diff delivers is what fails.

**Check rationale**

The check, quoted exactly as it reads in `tools/pr-precheck/rubric.md`:

> | repro-before-after | The test evidence, read against the plan's test plan and the repro evidence in the plan context: which command ran, on which code (before and after the change), and what output it printed. | Pass if the plan's primary repro (the issue's own failing case, as the plan's test plan re-runs it) is shown with real output both before the change and after it, the two outputs differ the way the plan said they should, and the run exercises the path the diff changes. Before and after may sit in one block or be labeled in the description or the evidence, as long as both outputs are shown. Any other step of the plan's test plan (a control, a side behavior, the project's own test suite) passes if its output is shown, or if the evidence states its result and a test added by the diff or a named repository check covers it. Fail if the primary repro is only a claim (tested locally, works now, tests pass), shows only one side, runs a control or an unchanged path instead of the primary repro, or shows no observable output. Also fail if a secondary step is neither shown, nor stated, nor covered by a test or check. Parts the plan or description honestly defers need no evidence. | required |

Why it reads that way: my first version said to fail if the evidence "skips part of what the plan's test plan requires". That scored 16/20 and wrongly rejected pkg-05, pkg-11 and pkg-19, where the issue's own repro is shown before and after but a secondary step (a control, or a second case) was only stated or was covered by a test the diff adds. I split the check into the primary repro, which must show real output on both sides and exercise the path the diff changes, and secondary steps, which pass if shown, or if stated and covered by a test or a named repository check. I rejected simply deleting the "both sides" requirement, because that would let through the not-tested packages that only claim "tested locally" or run a control path (pkg-04, pkg-07, pkg-10, pkg-14), and those all still reject.

**Trade-offs**

Loosening this check gives up strictness on secondary test-plan steps: a package whose secondary step is only stated, and is covered by a test the diff adds, now passes even if that test's run output is not pasted. I accept that, because the gold labels accept those packages. I checked what the change touched with a canary re-run using `--only`: pkg-04, pkg-07, pkg-10 and pkg-14 (the not-tested rejects), pkg-03 and pkg-17 (silent-drift) and pkg-01 (standards-wall) all kept their reject verdicts, and the four clear accepts flipped to accept. One case I know it can still miss: in the final full run pkg-02 (a clear accept) was rejected on diff-clean, a check I did not change, which shows the grader can vary between runs. I did not chase it because 19/20 clears the 18/20 bar and every category matched.

---

Related paths: `eval-run.txt` in this directory; your skill's files in
`tools/pr-precheck/`.
