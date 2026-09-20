# Rubric: is this a good first issue?

All recency thresholds are measured against the bundle's `captured:` date in
eval mode, and against today in live mode. Every evidence source below is a
field a grader can point at: a line in the bundle's Repo facts block, the
issue header, the issue body, or the Comments section (its live-mode twin is
the same signal on github.com).

## Checks

| Check | Evidence | Pass condition | Weight |
|---|---|---|---|
| `maintainer-alive` | Repo facts: author names and commit titles in "last 5 default-branch commits", and the days-to-first-response figures in "maintainer first-response sample". Comments section: `author_association` on each comment | Passes on either signal: a default-branch commit within 60 days of capture that is human work, counting a bot merge commit whose title merges a human contributor's pull request ("Merge pull request #N from someuser") and not counting bot automation such as dependency bumps, generated docs, or leaderboard updates; or a maintainer first response of 60 days or less in the sample, or an OWNER, MEMBER, or COLLABORATOR comment in this issue's thread. Grade `unclear` when no source establishes recent human maintainer activity either way | required |
| `repo-in-use` | Repo facts: "archived:" on the repo line, the dates in "last 5 default-branch commits", and "last push to any branch" | Not archived, and either the newest default-branch commit or the last push to any branch is dated within 90 days of capture. Fail if archived, or if both dates are more than 90 days old | required |
| `scope-bounded` | The issue body, and the full Comments section | The issue asks for one bounded change with a single contribution outcome. A body that lists several parts, files, sub-headings, or named causes still passes as long as they all serve that one outcome and one PR could plausibly cover them (e.g. one new docs page plus updating the handful of existing pages that should point to it; one bug fixed the same way across a short enumerated list of call sites; a maintainer naming two candidate root causes for one bug). Fail only on: a tracking or umbrella body that explicitly says its sub-items are meant to become separate, independently tracked issues or PRs; a thread showing genuine unsettled disagreement, meaning two or more people (not one maintainer alone) still proposing competing designs at capture with no maintainer-stated spec or accepted approach, or a maintainer doubting whether to do the work at all; a required product, design, or input decision left explicitly open or TBD; a maintainer saying the fix reaches core internals such as the parser or the renderer; or a usage or support question rather than a change to the code. A maintainer or collaborator listing multiple candidate causes or approaches for their own bug report is scoping an investigation, not an unsettled debate, and does not fail this check by itself. A terse body, absent reproduction steps, or a bare acceptance-criteria checklist do not fail it either: grade the size of the work asked for, not the polish of the writeup | required |
| `unclaimed` | Issue header: state. Repo facts: "this issue: assignees", "linked PRs" with each state. The Comments section, for pull requests mentioned there and for claim language | Open, assignees none, no linked or thread-mentioned pull request that is open at capture, and no claim comment ("I'll take this", "can I work on this", "working on this", "/assign") dated within 60 days of capture and not withdrawn. Two things do not fail this check: a closed unmerged pull request, which is an abandoned attempt rather than a live claim, and a claim comment older than 60 days with no open pull request behind it. In live mode a house rule in scope.md may say whose claim comments count here; apply it | required |
| `attempt-history` | Repo facts: the issue's opened date and "linked PRs" with each state. Comments section: claim, abandonment, and unassignment history | Fail only when both hold at once: the issue has been open more than 365 days, and it carries 2 or more closed unmerged pull requests literally linked or mentioned against it. Nothing else fails this check. Age alone passes. A single closed unmerged PR passes, however old the issue. A stale claim comment with no PR behind it, a stale-bot label, or a maintainer inviting someone to try the issue after an earlier claim went quiet all pass: those are normal history for an old issue, not abandoned attempts, and only a literal count of 2+ dead PRs says the work is harder than it looks | required |
| `ai-policy-permits` | Repo facts: the "contribution policy" line, including any AI policy file or template it quotes | No outright ban on AI-assisted contributions. Silence passes: a policy that says nothing about AI, or no policy line at all, passes. Conditions pass: disclosure, personally understanding the change, testing it, human review of AI output, and bans on fully AI-generated work that still allow assistive use, are terms to follow rather than reasons to walk away. Fail only on a stated ban covering assisted work, e.g. "we do not accept AI-generated code" or AI-assisted pull requests will be closed | required |
| `good-first-issue-label` | Issue header: the labels line | Labels include one of: good first issue, good-first-issue, beginner friendly, help wanted, documentation | preferred |
| `fix-located` | The issue body, and any maintainer comment in the Comments section | The body or a maintainer names the file, function, or line to change, or shows the intended diff | preferred |
| `maintainer-in-thread` | Comments section: the `author_association` on each comment; issue header: the opener's association | At least one comment in this thread, or the issue itself, comes from an OWNER, MEMBER, or COLLABORATOR | preferred |

## Verdict rule

Accept only if all six `required` checks grade `pass`. Any single required
`fail` rejects the issue.

`unclear` on a required check counts as `fail`: a first issue whose evidence I
cannot verify is not one I should take. The one place absent evidence still
passes is `ai-policy-permits`, whose pass condition says so outright, because
most repos state no AI policy and silence is not a restriction.

`preferred` checks never change the verdict. They rank the issues that are
accepted: more preferred passes ranks higher, and ties break in the table
order above, `good-first-issue-label` first. On an accepted issue, report them
as the reasons to prefer it over the other accepted candidates.
