# Unit 1 — Issue Selection

Path: `beat-1-sandbox/unit-1/selection.md`

Record of the issue carried into Unit 2, and of the evaluation runs that produced
`eval-run.txt`. This file is graded at the path above; a copy kept anywhere else in
the repository is not read.

Complete every labelled field below. Each is graded on its own; content placed under the
wrong label is not graded.

---

## Selected issue

**Issue link**

https://github.com/codepath/pathreview-ai301-fa26-s1/issues/51

**Verdict output**

Live-mode run: `claude "issue-select: grade these candidate first issues: https://github.com/codepath/pathreview-ai301-fa26-s1/issues/51 https://github.com/codepath/pathreview-ai301-fa26-s1/issues/68 https://github.com/codepath/pathreview-ai301-fa26-s1/issues/18"`, pasted verbatim:

## Summary

All three candidates are inside scope (codepath/pathreview-ai301-fa26-s1) and all pass every required check — accept all three. Ranked by fit profile (well-scoped bug fix, easy to reproduce/test locally, backend/DevOps leaning):

1. **#68 — Keyword search ZeroDivisionError** (accept, 3/3 preferred). Best fit: it's a genuine bug (not just an enhancement), already reproduced by another student in the thread with a full traceback — about as easy to reproduce and test locally as it gets. The claim comment from `yulijasso` doesn't block it per the Path Review house rule.
2. **#18 — Repo analyzer file list detection** (accept, 3/3 preferred). Bounded, well-located fix, but labeled `enhancement` rather than `bug` and sits more in agent/ingestion code than backend/DevOps.
3. **#51 — CI migration validation step** (accept, 2/3 preferred). Best topical fit for a DevOps/database background, but it's `tier-3` with no `good first issue` label and a larger 5–7hr estimate — more ambitious than a first issue needs.

```json
[
  {
    "item": "https://github.com/codepath/pathreview-ai301-fa26-s1/issues/68",
    "checks": [
      {"name": "maintainer-alive", "grade": "pass", "evidence": "Commit f89c06fc3 by Aburke225 (User) dated 2026-09-16, 4 days before capture (2026-09-20)"},
      {"name": "repo-in-use", "grade": "pass", "evidence": "Not archived; pushed_at 2026-09-16T21:48:27Z, within 90 days"},
      {"name": "scope-bounded", "grade": "pass", "evidence": "Single fix: guard KeywordSearcher.index() against empty corpus and remove the xfail marker on one test"},
      {"name": "unclaimed", "grade": "pass", "evidence": "No assignees, no linked PR; yulijasso's claim/repro comments are a fellow student's claim, which the Path Review house rule says does not block the issue"},
      {"name": "attempt-history", "grade": "pass", "evidence": "Opened 2026-09-10 (10 days old), zero closed PRs against it"},
      {"name": "ai-policy-permits", "grade": "pass", "evidence": "docs/CONTRIBUTING.md is silent on AI-assisted contributions"},
      {"name": "good-first-issue-label", "grade": "pass", "evidence": "Labels: bug, good first issue, rag, tier-1"},
      {"name": "fix-located", "grade": "pass", "evidence": "Body names rag/retriever/keyword_search.py, KeywordSearcher.index(), and tests/unit/test_keyword_search.py"},
      {"name": "maintainer-in-thread", "grade": "pass", "evidence": "Opener Aburke225 has author_association Collaborator"}
    ],
    "verdict": "accept"
  },
  {
    "item": "https://github.com/codepath/pathreview-ai301-fa26-s1/issues/18",
    "checks": [
      {"name": "maintainer-alive", "grade": "pass", "evidence": "Commit f89c06fc3 by Aburke225 (User) dated 2026-09-16, within 60 days"},
      {"name": "repo-in-use", "grade": "pass", "evidence": "Not archived; pushed_at 2026-09-16T21:48:27Z, within 90 days"},
      {"name": "scope-bounded", "grade": "pass", "evidence": "Single fix: populate file_structure so _detect_tests()/_detect_ci() work correctly, one outcome across two files"},
      {"name": "unclaimed", "grade": "pass", "evidence": "No assignees, no linked PR, comments array empty"},
      {"name": "attempt-history", "grade": "pass", "evidence": "Opened 2026-09-10 (10 days old), zero closed PRs against it"},
      {"name": "ai-policy-permits", "grade": "pass", "evidence": "docs/CONTRIBUTING.md is silent on AI-assisted contributions"},
      {"name": "good-first-issue-label", "grade": "pass", "evidence": "Labels: enhancement, good first issue, agent, tests, tier-1"},
      {"name": "fix-located", "grade": "pass", "evidence": "Body names agent/tools/github_tool.py, ingestion/parsers/repo_analyzer.py, and the specific methods _detect_tests()/_detect_ci()"},
      {"name": "maintainer-in-thread", "grade": "pass", "evidence": "Opener Aburke225 has author_association Collaborator"}
    ],
    "verdict": "accept"
  },
  {
    "item": "https://github.com/codepath/pathreview-ai301-fa26-s1/issues/51",
    "checks": [
      {"name": "maintainer-alive", "grade": "pass", "evidence": "Commit f89c06fc3 by Aburke225 (User) dated 2026-09-16, within 60 days"},
      {"name": "repo-in-use", "grade": "pass", "evidence": "Not archived; pushed_at 2026-09-16T21:48:27Z, within 90 days"},
      {"name": "scope-bounded", "grade": "pass", "evidence": "Single bounded ask: 'Add a step that spins up a fresh database, runs all migrations in order, and verifies the schema matches the SQLAlchemy models'"},
      {"name": "unclaimed", "grade": "pass", "evidence": "Open, no assignees, no linked PRs, comments array empty"},
      {"name": "attempt-history", "grade": "pass", "evidence": "Opened 2026-09-10 (10 days old), zero closed PRs against it"},
      {"name": "ai-policy-permits", "grade": "pass", "evidence": "docs/CONTRIBUTING.md is silent on AI-assisted contributions"},
      {"name": "good-first-issue-label", "grade": "fail", "evidence": "Labels: enhancement, devops, tests, tier-3 — no good-first-issue-style label"},
      {"name": "fix-located", "grade": "pass", "evidence": "Body names .github/workflows/ci.yml and scripts/validate_migrations.sh"},
      {"name": "maintainer-in-thread", "grade": "pass", "evidence": "Opener Aburke225 has author_association Collaborator"}
    ],
    "verdict": "accept"
  }
]
```

---

## Eval iterations

**Run history**

Run 1 (crashed, no items graded):

> `FileNotFoundError: [WinError 2] The system cannot find the file specified`

On Windows, Python's subprocess call looks for a bare `claude` command, but the installed shim is `claude.cmd`, which Windows will not resolve without a file extension. Nothing was graded.

Run 2 (crashed, no items graded), after fixing the PATH to point at `claude.exe` directly:

> `UnicodeEncodeError: 'charmap' codec can't encode character '\U0001f389' in position 18284: character maps to <undefined>`

One calibration bundle contains an emoji in a comment, and Windows' default text encoding could not send it to the `claude` process. Fixed by running with `PYTHONUTF8=1`.

Calibration check (4 unscored worksheet issues, run with `--include-calibration --only calib-01,calib-02,calib-03,calib-04`):

> `agreement: 0/0 scored items`

All four matched gold by hand-comparison of the printed table (calibration issues are never scored, so the harness prints 0/0): calib-01 accept, calib-02/03/04 reject, all correct.

First full run (20 scored issues):

> `categories: claimed 4/4  clear-accept 4/8  dead-repo 3/3  policy 1/1  scope 4/4`
> `agreement: 16/20 scored items  (bar: 18/20: below the bar)`

Every category floor was met, but `clear-accept` was badly off (4/8). Three of the four misses failed on `scope-bounded`, one failed on `attempt-history`.

Revision: `scope-bounded` was rejecting issues that listed several parts or causes even when they all served one outcome (a docs task that touches a few pages, a maintainer naming two possible causes for their own bug). I loosened it to pass a multi-part body when the parts serve one outcome, and to fail only on a genuine umbrella issue or real design disagreement between two or more people. `attempt-history` had a vague "comparable run of abandoned attempts" clause that let a single old claim comment count as a fail; I tightened it to require a literal count of 2 or more closed unmerged PRs.

Partial rerun of just the four misses, after the revision (`--only issue-01,issue-04,issue-09,issue-19`):

> `agreement: 4/4 scored items`

Re-checked calibration after the revision, to make sure the loosened check did not swallow the deliberately arguable scope trap:

> `agreement: 0/0 scored items`

All four calibration issues still matched by hand-comparison, including calib-04, which stayed reject.

Final full run (saved to `eval-run.txt`):

> `categories: claimed 4/4  clear-accept 7/8  dead-repo 3/3  policy 1/1  scope 4/4`
> `agreement: 19/20 scored items  (bar: 18/20: PASS)`

**Issue analysis**

Scored issue: `issue-19`

My rubric's decision was `reject`.

The gold label was `accept`.

The final eval output was:

> `issue-19  accept  reject   NO     failed: scope-bounded, good-first-issue-label (preferred), fix-located (preferred)`

The issue's body reads:

> `There are two potential causes which should be fixed:`
> `1. The matchers are slow for certain rewrites (quadratic instead of linear)`
> `2. UI update is waiting for the matching thread to finish`
>
> `Additional suggestions:`
> `1. We should use multi-processing to use all the cores to match rewrites in parallel`
> `2. Only match rewrites in the categories that are not collapsed...`
> `3. Applying the rewrite should also happen in a separate thread`

The issue was opened by RazinShaikh, a collaborator on the repo, describing their own bug and listing possible causes and possible fixes for it. My `scope-bounded` check is written to treat that as one maintainer scoping an investigation rather than an unsettled group debate, and in an isolated rerun of this single issue it did grade `pass` and the overall verdict came out `accept`, matching gold. In the full 20-issue run, though, the same check graded `fail` on the same wording. The check's language asks the grading model to judge whether a list of causes and suggestions is "one maintainer scoping an investigation" or "genuine unsettled disagreement," and that line is soft enough that the model did not always land on the same side of it for this particular issue.

**Check rationale**

Current check from my final `rubric.md`:

> `| `scope-bounded` | The issue body, and the full Comments section | The issue asks for one bounded change with a single contribution outcome. A body that lists several parts, files, sub-headings, or named causes still passes as long as they all serve that one outcome and one PR could plausibly cover them (e.g. one new docs page plus updating the handful of existing pages that should point to it; one bug fixed the same way across a short enumerated list of call sites; a maintainer naming two candidate root causes for one bug). Fail only on: a tracking or umbrella body that explicitly says its sub-items are meant to become separate, independently tracked issues or PRs; a thread showing genuine unsettled disagreement, meaning two or more people (not one maintainer alone) still proposing competing designs at capture with no maintainer-stated spec or accepted approach, or a maintainer doubting whether to do the work at all; a required product, design, or input decision left explicitly open or TBD; a maintainer saying the fix reaches core internals such as the parser or the renderer; or a usage or support question rather than a change to the code. A maintainer or collaborator listing multiple candidate causes or approaches for their own bug report is scoping an investigation, not an unsettled debate, and does not fail this check by itself. A terse body, absent reproduction steps, or a bare acceptance-criteria checklist do not fail it either: grade the size of the work asked for, not the polish of the writeup | required |`

I wrote it this way because my first version of this check failed whenever an issue body listed more than one part, cause, or step, and that wrongly rejected several gold-accept issues in my first full run: a docs task that touched a few related pages, a bug fix repeated across a short list of call sites, and a maintainer listing two candidate causes for their own bug. The rewrite keeps the check's real job, rejecting genuinely open-ended issues such as tracking issues, unresolved design debates, and support questions, while explicitly allowing a body with multiple parts or causes when they all point at one outcome one PR could deliver.

**Trade-offs**

The loosened `scope-bounded` check fixed three of my four missed issues in the same run (issue-01, issue-04, issue-09 all flipped from reject to accept and matched gold), which is a real gain: it stopped punishing issues for being described in multiple parts when the work itself is still one bounded outcome.

The trade-off shows up on issue-19. The check now depends on the grading model correctly telling apart "one maintainer scoping an investigation with a list of causes" from "genuine unsettled disagreement," and that line is judgment, not a number. On this issue the check landed on `pass` when graded alone and `fail` when graded as part of the full 20-issue batch, so the same rubric wording does not always produce the same grade on a borderline case like this one. A stricter, safer version of the check would go back to failing on any multi-part body, but that is the version that wrongly rejected issue-01, issue-04, and issue-09, so I judged the current wording, which risks an occasional miss on a genuinely borderline issue like issue-19, a better trade than reverting to a rule that reliably rejected bounded issues just because they listed more than one part.

---

## Selection rationale

**Selection rationale**

1. Issue #51 fits my interests because it is CI and database-related work, which lines up with my DevOps and data-engineering background. It fits the time I have less well than the other two accepted candidates: it is labeled tier-3 with no good-first-issue label, and the issue itself estimates 5-7 hours, more than double the 2-4 hour estimates on the other two accepted issues.

2. The verdict correctly identified that the repo is active, the issue is unclaimed, the fix is located to two named files (`.github/workflows/ci.yml` and `scripts/validate_migrations.sh`), and the contribution policy does not block AI-assisted work. It ranked #51 last of the three accepted issues because it only passed 2 of 3 preferred checks (it has no good-first-issue-style label) and my fit profile favors an easy-to-reproduce, easy-to-test issue over topic match. What the rubric could not weigh is that I would rather spend my first contribution on something closer to my existing DevOps and database experience than on the RAG/search bug ranked above it, even at a larger time cost.

3. I expect claiming #51 to take longer than the other two candidates, both because of its own 5-7 hour estimate and because it touches CI configuration and a shell script rather than a single Python function, so I will need to actually run the migrations locally against a fresh database to confirm the validation step behaves the way the issue describes before I can trust a fix. There is no assignee, no linked or open PR, and no comment showing anyone already working on it, so the claim itself should be uncontested.

---

Related paths: `eval-run.txt` in this directory; your skill's files in
`tools/issue-select/`.
