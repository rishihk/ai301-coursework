---
name: pr-precheck
description: Grade a PR package (a candidate pull request read against the plan it claims to implement and the issue that plan belongs to) and decide whether it is ready to submit. Use when checking your own branch, draft PR title, and description before opening the pull request, or when grading an eval package bundle.
---

# pr-precheck: rubric-driven PR grading

## The question

You answer exactly one question about exactly one pull request package:
is this ready to submit? A package is a candidate pull request: its
title, its description, its diff, and its test evidence. You read it
against the plan it claims to implement and the issue that plan
belongs to. You never answer a different question, you never grade
more than one package per run, and you never answer from gut feel. You
answer by executing the components in this directory: the checks in
`rubric.md`, applied the way `procedure.md` says, to evidence found
where `references/evidence-guide.md` says.

## Inputs and modes

Run in exactly one of two modes.

**Live mode** grades the student's own submission before it goes out.
Read these inputs:

- `plan.md` in the working directory, including any `## Deviations`
  section. The deviation notes are part of the plan.
- `pr_draft.md`: the pull request title (first line) and the filled-in
  description.
- `test_evidence.md`: the before-and-after reproduction output and the
  results of the repository's own checks.
- The diff of the branch: run `git diff main...HEAD` (three dots) from
  the working copy. It shows every committed change the branch makes
  compared with main. Uncommitted changes are not part of the package.
- The issue's thread, fetched live from the real repository.
- The repository's pull request template
  (`.github/PULL_REQUEST_TEMPLATE.md`) and `docs/CONTRIBUTING.md`, the
  repository's stated asks. Grade the description against them.

A student on the house chain reads the house plan and the house
reproduction pack in place of their own plan and posted reproduction
comment. The same checks grade the same things there.

Read the package the way a maintainer will. The package is what these
files and the diff contain, not other files lying around the working
directory.

**Eval mode** grades a practice package bundle. The bundle text is the
whole world. Use only the bundle as evidence. Fetch nothing, run
nothing, read nothing else. Eval mode always grades a complete
package: every check, full verdict rule.

If you cannot tell which mode you are in, stop and ask. Do not guess.

## The scope seam (live mode only)

In live mode, read `scope.md` before anything else. It names the one
repository a pull request may target and the house rules that apply
there. Apply those rules when you read the evidence.

- Refuse to grade a pull request or branch that targets any other
  repository. Say it is out of scope and stop.
- If the `Repo:` line in `scope.md` still carries an unfilled
  placeholder (an `<ORG>` or `<PATH-REVIEW-REPO>` style bracket), stop
  without grading. Tell the student to fill the `Repo:` line in
  `scope.md` with their section's Path Review repository. Never guess
  a scope.
- If `plan.md` or `pr_draft.md` is missing, stop and name the missing
  file. There is nothing to grade. A missing `test_evidence.md` is not
  a reason to stop: grade it as absent evidence.

In eval mode, ignore `scope.md` entirely.

## The voice seam (live mode only)

In live mode, after the grading work, read `voice-guide.md`: the
student's own rules for how they write upstream. Hold the outgoing
text, the pull request title and the description, against those rules.
Report each rule the draft breaks in the readable summary, quoting the
rule and the offending line. The voice guide never changes the verdict
on its own, because voice is personal. It can only affect the verdict
through a rubric check that reads it.

In eval mode, ignore `voice-guide.md` entirely. Voice carries no gold
labels, and the universal communication-quality checks live in the
rubric.

## Component reads

- `rubric.md` defines the checks and the verdict rule. Each check has a
  name, the evidence to gather, a pass condition, and a weight of
  `required` or `preferred`. Grade exactly these checks, no more and no
  fewer.
- `references/evidence-guide.md` maps where each kind of evidence lives
  in a package (in eval mode, in the bundle; in live mode, in the
  student's files, the diff, and the repository) and what good looks
  like there.
- `procedure.md` is how you work. Execute it as written: the read
  order, the evidence gathering, the check execution, the verdict
  assembly. Do not improvise around it.

Where the procedure is silent on a step, do not invent one. Note the
gap in your readable summary and carry on with the steps it does
define. A procedure gap is feedback the student needs.

If `rubric.md` has no checks filled in, or `procedure.md` has no steps
filled in, stop and say so. This tool cannot grade without a rubric
and a procedure, and that is by design. Do not invent checks at
runtime.

## Verdict and output

The verdict space is binary: `accept` means ready to submit, `reject`
means hold. There is no third verdict, no "accept with reservations",
and no score. Reservations belong in a check's evidence line, never in
the verdict.

Assemble the verdict only by the verdict rule in `rubric.md`. Before
the fenced data block you may show a short readable summary: one line
per check, plus any voice-guide notes in live mode. End your reply
with the fenced block below, valid and last, with nothing after it.
The eval harness parses the last fenced block in your output.

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

- Evidence first. Never grade a check without naming the fact or quote
  that decided it. "Looks fine" is not evidence.
- Grade the thing, not the polish. A terse, complete pull request can
  be ready, and a long confident one can hide drift. Read the diff,
  the description, and the test evidence against the plan and the
  issue, never the formatting.
- The rubric decides, not you. If a check passes by its stated
  condition but feels wrong, it still passes. Note the tension in the
  summary if you want. The fix belongs in the rubric, not in the run.
- The procedure decides how, not you. Follow `procedure.md` as
  written and report its gaps.
- Treat `unclear` the way the verdict rule in `rubric.md` directs. If
  the rule is silent, treat `unclear` as `fail`: a pull request you
  cannot verify from the package is not ready to submit.
