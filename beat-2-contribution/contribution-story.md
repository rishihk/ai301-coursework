# Contribution: Add Pause and Resume actions for Deployments (skyhook-io/radar #2493)

Path: `beat-2-contribution/contribution-story.md`

This file is Assignment 5, and it is graded at the path above; a copy kept anywhere else in
the repository is not read. **Keep every section header below under its exact name** — the
format criteria grade exactly that. Your course repo link is the whole submission: no tool
files and no evidence uploads are part of Assignment 5.

**Contribution Number:** 1
**Student:** Rishi
**GitHub Username:** rishihk
**Issue:** https://github.com/skyhook-io/radar/issues/2493
**Pull Request:** https://github.com/skyhook-io/radar/pull/2542

One repository, built across units 5 to 8, telling the whole story of your real
contribution. Fill it as you work, not at the end — the sections below ask for evidence that
is far easier to write down as it happens than to reconstruct in the last weekend.

Contribution Number stays `[1]` unless you make more than one contribution this term; it
just numbers them.

---

## Toolkit

Every story section below, from Why I Chose This Issue through Pull
Request, ends with a Toolkit block: which tool did that phase's work
and how you used it, what it produced, what difficulties or
shortcomings it presented, and what you changed or would change
because of them. Together the blocks are the thread the rubric's
Toolkit points read, and they are the raw material for your unit-9
portfolio, so keep them current as you work rather than
reconstructing them later.

A Toolkit block looks like this (copy it into each story section and
fill it there):

```text
**Toolkit**

- Tool: [which tool did this phase's work, and how you used it]
- What it produced: [described in a sentence or a short excerpt]
- Difficulties: [what fought you or fell short, or "none met"]
- What changed: [the patch, revision, or workaround, what you would
  update, or "nothing"]
```

This section also keeps one short dated entry per week of tool work,
starting below: what you built or changed that week, what broke, and
what you changed because of it. The blocks tell the story where it
happened; these entries are the toolkit's own timeline.

All four entries are graded. Keep every heading below, and fill each
one in the week it belongs to.

**Unit 5: Built issue-scout, a three-stage scanner that searches GitHub for open issues, ranks them, and grades the top few against a fit profile**

I installed the course's starter scout as issue-scout, wrote my fit profile into scope.md (data engineer, Kubernetes and cloud background, wants a well-scoped bug fix I can reproduce locally), and wrote a search strategy of gh queries that each trace to a profile sentence, plus ranking rules. The first real scan graded five issues and accepted none, because every one already had a claim comment or an open pull request; the search results only show assignees and formally linked pull requests, so they cannot see claims, and many top results were enhancement or maintenance titles. I rewrote the strategy to exclude linked pull requests, drop stale and over-discussed threads, and widen from AI/ML repos to Kubernetes, DevOps and data-engineering repos, then swept the course issue site for free candidates. I also replaced the rubric's scope check with a scope-feasible check plus a difficulty note, and the regrade accepted all four direct picks while exposing a false negative in maintainer-in-thread, which I logged in scan-notes.md.

**Unit 6: Switched repro-check and plan-check to the wild scopes, graded my claim and repro with repro-check, and built a CLAUDE.local.md repo-memory file for Radar**

repro-check first refused to grade my claim on skyhook-io/radar because its scope.md still allowed only the classroom repo, so I installed the course's wild scopes for repro-check and plan-check and regraded: the claim and the repro both came back accept, and the voice-guide notes made me cut a PR timeline out of the claim. The repo-memory file (CLAUDE.local.md, kept out of git with .git/info/exclude) recorded Radar's conventions, the five layers the Restart action runs through, the build commands that worked on Windows without make, and my commit rules, and every later session started from it instead of re-reading the repo. What fought me was the machine, not the tools: Windows Smart App Control blocked Go's unsigned test binaries, so I moved the Go tests into a golang:1.26 Docker container and recorded that command in CLAUDE.local.md. pr-precheck also still has the classroom scope, so my PR review this week was a plain diff-against-description check rather than a rubric grade. The voice guard came last: I raced to claim a one-day-old issue, so my claim was checked by repro-check's voice-guide notes, and I installed the guard on 2026-11-10. I made the pr-precheck copy of my voice guide canonical (it already held the other two) and added an Upstream section with five rules (two for claims, two for questions, one for pings), each Right line quoting a comment I actually posted on #2493 or PR #2542. The wiring test stopped `gh issue comment --help` with "voice-guard: STOP. no body the guard can read", which INSTALL.md counts as the pass. Windows fought the install twice: `python3` is the Microsoft Store stub, so the hook points at my real python.exe with `-X utf8`, and `ln -s` copies instead of linking in Git Bash, so each skill uses INSTALL.md's one-line pointer file instead of a symlink.

**Unit 7: Installed the ship-gate with a Radar manifest; all four repo checks passed, and my pr-precheck rejected the branch for a missing plan.md and test_evidence.md**

I opened PR #2542 on 2026-11-04, before the unit 7 steps, so I ran the checks by hand that week and installed the ship-gate afterwards, on 2026-11-10. `install.py --repo C:\dev\radar` wrote `ship-manifest.md`, and I filled it with Radar's real pre-PR checks: Go tests and `go vet` on Linux in Docker (through a small script, because manifest rows run in cmd.exe and Smart App Control blocks Go test binaries on Windows), `npm run tsc` and the web tests. The hook test stopped `gh pr create --help` twice, once with "ship-gate: STOP. no PR text to check" and once from the voice guard. The full gate run on my branch passed all four checks (Go tests 224s, vet 31s, tsc 10s, web tests 140s), then my pr-precheck rejected it on diff-matches-plan and repro-before-after, because I never wrote `plan.md` or `test_evidence.md` at the clone root: my plan and before/after output lived in my course repo and PR description instead. That's the change I'd make next time: write those two files in the clone before opening the PR, so the gate can read them.

**Unit 8: Installed the PR monitor, wrote a playbook from Radar's real merge data, and ran it on my merged PR; it drafted a thank-you I won't post as written**

I watched PR #2542 by hand during review: Qodo's fixes, two rebases, and one nudge, until hisco approved and merged it on 2026-11-10. I installed the PR monitor the same day. `monitor.py --state` reported the PR as "MERGED", approved, with 8 checks passing and 3 skipped, and the full run wrote `run-report.md` plus a merged follow-up in `drafted-move.md`. For the playbook's staleness thresholds I timed the five most recently merged outside PRs (#1941, #2266, #2288, #2417, #2434): they waited 11 to 42 days for a maintainer's first comment, median 17.6, so I set 21 quiet days before a first nudge and 14 for a stalled review. That data showed my own nudge went out after only 2.1 days, earlier than this repo's pace supports, which I'd hold to next time. The draft thanked hisco for "going with the kind check in the handlers and the badge behavior as they were", which nobody said, so I'm not posting it as written. hisco's actual words were that my calls were right, and any thank-you will quote that instead.
---

## Why I Chose This Issue

I started out looking for an AI/ML bug fix, but my first scout scan accepted nothing: all five top results already had a claim comment or an open pull request, and checking by hand showed the popular AI/ML repos were saturated during Hacktoberfest. I widened my profile to Kubernetes, DevOps and data-engineering work, which matches my day job, and swept the course's issue site against GitHub for issues with no assignee, no linked or cross-referenced pull request and no recent claim. My first graded picks (mozilla-ai/cq #393, OpenTelemetry eBPF #1951, External Secrets #6239, Longhorn #6509) were all accepted, but each had a catch: a design to negotiate, a maintainer doubting the bug, a cloud-only reproduction, or work living in another repo. A last, wider sweep found skyhook-io/radar #2493, which the scout graded accept (all six required checks, two of three preferred). It is a Kubernetes dashboard with a built-in MCP server for AI agents, written in Go and TypeScript, so it covers both the infrastructure side and the agent side I care about. The issue was one day old, unclaimed, labeled good first issue, and laid out layer by layer against an existing pattern (Restart), and the maintainer merges outside contributions within about a day. It is a small feature rather than the bug fix my profile asked for, which I accepted for the clear scope and the fast review. I hope to learn how a real Go and React codebase threads one action through its core, API, MCP and UI layers, and how to get a first pull request reviewed by an active maintainer.

The scout held up on the facts it can see, but it has blind spots on a real wild search. Search results only show assignees and formally linked pull requests, so claims and cross-referenced pull requests are invisible until the grade step, which is why my first scan wasted all five grades. Its maintainer-in-thread check relies on GitHub's author_association, so it marked Radar's actual maintainer (nadaverell, a code owner who writes most commits) as a non-maintainer, a false negative it repeated on OpenTelemetry the same day. It also has no check for a maintainer documenting an issue as an accepted limitation, a stale bot about to close the issue, or the fix living in another repository. I logged all of these in scan-notes.md; the maintainer-in-thread check is the next thing I would patch.

Claim comment: https://github.com/skyhook-io/radar/issues/2493#issuecomment-6622019627

**Toolkit**

- Tool: issue-scout (my search strategy v2 and patched rubric). I ran one scan, then graded direct URLs from my sweep of the course issue site, ending with `claude "issue-scout: grade https://github.com/skyhook-io/radar/issues/2493"`.
- What it produced: an accept verdict on #2493, difficulty moderate. All six required checks passed; good-first-issue-label and fix-located passed; maintainer-in-thread failed.
- Difficulties: the first scan accepted nothing because claims and cross-referenced PRs are invisible to search; four direct grades in a row were accepted but each had friction the rubric doesn't catch; maintainer-in-thread gave a false negative twice.
- What changed: strategy v2 with `-linked:pr` and stricter ranking rules, a scope-feasible check plus a difficulty note in the rubric, and a site sweep that checked claims before grading. Next I would patch maintainer-in-thread to count code owners and recent committers.

---

## Understanding the Issue

**Problem Description**

A paused Kubernetes Deployment holds pod template changes without rolling them out. Radar could restart, scale and roll back a Deployment, but it had no way to pause or resume one. Since #2363, Radar also refuses to roll back a paused Deployment and tells the user to resume it first, so a dashboard user hits a dead end and has to leave Radar to resume.

**Expected Behavior**

Radar offers Pause and Resume for Deployments, equivalent to `kubectl rollout pause` and `kubectl rollout resume`, in the UI (one button that reads Pause rollout or Resume rollout from `spec.paused`, plus a Paused badge), in the REST API, and as `pause` and `resume` actions on the MCP `manage_workload` tool. Repeating a pause or resume succeeds without changing anything, and other kinds are rejected.

**Current Behavior**

On main (`2fa4ccf`), the Deployment page has no Pause or Resume action, the `/pause` and `/resume` routes return 404, and `manage_workload` rejects both as unknown actions. Rollback of a paused Deployment returns 409 with the #2363 message on REST, MCP and the UI. A user can technically resume by editing `spec.paused` in the YAML tab, so the gap is a missing direct action rather than a hard kubectl requirement.

**Affected Components**

- Core: `pkg/k8score/workload.go` (next to `RestartWorkload`) and the wrapper in `internal/k8s/update.go`
- REST: `internal/server/server.go`
- MCP: `internal/mcp/tools_workloads.go`, `internal/mcp/tools.go`, `docs/mcp.md`, `web/src/components/home/mcpToolCatalog.ts`
- UI: `web/src/api/client.ts`, `packages/k8s-ui/src/components/shared/ResourceActionsBar.tsx` (where the buttons actually live), `WorkloadView.tsx`
- Version skew: the `FeatureCapabilities` flag and `web/src/api/radarFeatures.ts`, which the repo's CLAUDE.md requires for new UI-called endpoints

**Toolkit**

- Tool: Claude Code running locally in my Radar clone, guided by CLAUDE.local.md. I had it do one scoped read of only the files the issue names and their tests.
- What it produced: a map of the Restart action through every layer, with file and line references, distilled into CLAUDE.local.md.
- Difficulties: the issue's file list was incomplete. The button lives in `ResourceActionsBar.tsx`, not `WorkloadView.tsx`; Restart has no permission check to copy; and the feature flag, MCP catalog and guidance text weren't listed.
- What changed: I raised those differences on the issue before writing code and planned around them.

---

## Reproduction Process

**Environment Setup**

Windows 11 with Git Bash (no `make`), Docker Desktop 29.8.0, kind v0.33.0 (node image kindest/node:v1.37.0), kubectl v1.36.1, Go 1.27.3, Node v26.3.0 and npm 11.16.0, Radar at `2fa4ccf` (equal to upstream main at the time). I installed kind and Go with winget, forked and cloned with `gh repo fork skyhook-io/radar --clone`, and ran the Makefile's frontend, embed and backend steps by hand: `npm ci`, `(cd web && npm run build)`, copy `web/dist` into `internal/static/dist`, then `go build -o radar.exe ./cmd/explorer`. Challenges: Docker Desktop had to be running before kind could create the cluster; Windows Application Control blocked Go's linker once (a plain retry worked); and Smart App Control blocked Go test binaries, so I ran the Go tests in a `golang:1.26` Docker container instead of changing the security setting.

**Steps to Reproduce**

1. `kind create cluster --name radar-repro`, then `kubectl create deployment web --image=nginx:1.27` and `kubectl rollout pause deployment/web`. `kubectl get deployment web -o jsonpath='{.spec.paused}'` prints `true`.
2. Build Radar from main and run `./radar.exe --no-browser --port 9280` against the cluster, then open `http://localhost:9280/workload/deployments/default/web`.
3. Observed: the action bar shows Set image, Restart and Rollback, but no Pause or Resume. `POST /api/workloads/deployments/default/web/pause` and `/resume` return 404.
4. Observed: the MCP `manage_workload` tool returns `unknown action "pause": must be restart, scale, or rollback`, and the same for resume.
5. Observed: rollback of the paused Deployment returns 409 ("workload is paused; resume it first…") on REST, MCP and the UI (on a second, two-revision Deployment), and kubectl refuses too. As a control, the same rollback returns 200 once the Deployment is resumed.

**Reproduction Evidence**

- **Commit showing reproduction:** the reproduction ran on unmodified upstream main, https://github.com/skyhook-io/radar/commit/2fa4ccf70d1473a1b0cecfb495f742c646b6cd74, since the issue is a missing feature; no repro commit was needed. Full write-up: `repro-draft.md` in this repo, graded accept by repro-check (all seven checks pass).
- **Screenshots/logs:** command output is in `repro-draft.md`; the reproduction comment I posted is https://github.com/skyhook-io/radar/issues/2493#issuecomment-6623687438
- **My findings:** all three claims in the issue reproduced. Four details didn't match: resuming is already possible by editing `spec.paused` in the YAML tab; the page already shows "Rollout paused" in places but the header and resource list don't; the one-revision fixture can't reach the UI rollback refusal; and Restart has no patch-permission check to copy. I posted these on the issue along with one question for the maintainer about where the kind check should live.

**Toolkit**

- Tool: Claude Code locally, to build and drive Radar against my kind cluster, then repro-check (`claude "repro-check: grade my repro draft in repro-draft.md for issue …"`) to grade the write-up.
- What it produced: `repro-draft.md` with versions, rerunnable commands and pasted output; repro-check graded it accept on all seven checks.
- Difficulties: repro-check first refused because of the classroom scope; the fixture had one revision, so a second Deployment was needed to reach the UI refusal; Smart App Control and the linker block on Windows.
- What changed: installed the wild scopes; added a throwaway two-revision Deployment to the steps; moved the full write-up into the course repo and posted a short version on the issue instead.

---

## Solution Approach

**Analysis**

Nothing is broken; the action simply doesn't exist at any layer. Restart is implemented as `RestartWorkload` in `pkg/k8score/workload.go`, wrapped by `RestartWorkloadWithClient` in `internal/k8s/update.go`, exposed by a `{kind}` route in `internal/server/server.go`, offered as the `restart` action of `manage_workload` in `internal/mcp/tools_workloads.go`, and called from the UI through `useRestartWorkload` in `web/src/api/client.ts` and a button in `ResourceActionsBar.tsx`. Pause and Resume need the same path. The rollback refusal from #2363 (`ErrWorkloadPaused` in `rollbackDeployment`) is correct and stays as it is.

**Proposed Solution**

Add `SetDeploymentPaused`, a merge patch of `spec.paused` that is idempotent without reading first, and thread it through the wrapper, `POST /api/workloads/{kind}/{namespace}/{name}/pause` and `/resume` (400 for anything that isn't a Deployment, same audit log and impersonation as Restart), the MCP `pause` and `resume` actions, a `deploymentPause` feature flag for version skew, two UI hooks, one Pause/Resume button gated on `workloadWrites` like Set image, and a Paused badge. Deliberately not changed: StatefulSets, DaemonSets and Argo Rollouts (they have no pause, or have their own); the rollback refusal; and `pkg/investigation/guidance.go`.

**Implementation Plan**

1. Core: `SetDeploymentPaused` plus its `*WithClient` wrapper, with unit tests for pause, resume and the no-op cases.
2. REST: pause and resume routes following the sibling `{kind}` routes and `NormalizeWorkloadKind`, with handler tests including the 400.
3. MCP: `pause` and `resume` on `manage_workload`, with the schema, tool description, `docs/mcp.md` and `mcpToolCatalog.ts` updated.
4. UI: the feature flag, `usePauseDeployment` and `useResumeDeployment`, the button in `ResourceActionsBar`, and the Paused badge.
5. Live check on kind: pause from the UI, change the image and confirm nothing rolls out, resume and confirm it does, then roll back.

Deviations as they happened:
- `guidance.go` stayed unchanged: even the shortest wording would push the exec-enabled server instructions past the 3,440-character limit `TestServerInstructionsFitClientLimits` enforces.
- Added a `fix(ui)` commit: the page fetched revisions only once, so after a resume created a new revision, Rollback stayed disabled until a reload. It now refetches when the revision annotation changes.
- The kind check lives in the REST and MCP handlers, because `SetDeploymentPaused` takes no kind as the issue specified. I asked the maintainer about this on the issue.
- Permission gating follows Set image (`workloadWrites`), because Restart has no permission check for Deployments.
- After Qodo's review: added `requireConnected` before the cluster call, matching Rollback and revisions, and reworded a test comment that recorded change history.
- The Paused badge hides while a held rollout already shows "Rollout paused", so the header doesn't say it twice. Qodo flagged this against the issue's wording, and I explained the choice on the PR.

**Toolkit**

- Tool: Claude Code locally, working from CLAUDE.local.md one layer at a time, with each layer reviewed by me before moving on.
- What it produced: the plan above, and a list of places the code disagreed with the issue before any code was written.
- Difficulties: size budgets in the MCP tests left almost no room for new text; the issue's file list missed several layers.
- What changed: chose the shortest accurate MCP wording, raised the catalog budget deliberately, left `guidance.go` alone, and recorded every deviation in the PR description.

---

## Testing Strategy

- **Reproduction re-run on the built branch (live, kind cluster):** pausing from the UI sets `spec.paused=true` and flips the button to Resume rollout; changing the image while paused moves the template to `nginx:1.29-alpine` while the same ReplicaSet keeps running `nginx:1.27` (`rollout status` reports 0 of 1 updated); rollback while paused still returns 409; Resume from the UI rolls out revision 7; the new revision appears in the Rollback dialog without a reload; rolling back to revision 6 works. Pausing a StatefulSet or resuming a Rollout returns 400, a missing Deployment 404, a cross-site request 403. Over MCP, pause and resume succeed, repeating them leaves `generation` unchanged, and the StatefulSet and missing cases return errors. Before the change, the same steps gave 404 and "unknown action".
- **Unit and handler tests added:** core (pause, resume, no-op, keeps the rest of the spec, not found), REST (every kind spelling, 400 for 16 other kinds, error statuses, never falls back to Radar's own identity, patches as the caller at a fake API server, audit log, real router wiring, requires a connection), MCP, the UI button, the badge and the revision refresh.
- **Mutation checks:** I broke the code on purpose more than 40 ways (inverted actions, removed kind check, wrong status code, Radar's own client instead of the caller's, removed routes, duplicate badge) and confirmed a test failed for each.
- **The repo's own suites:** `go test ./...` at the root and in `pkg/` passes in `golang:1.26` on Linux; `go vet` is clean on the touched packages; `npm run tsc` has 0 errors in `web` and `packages/k8s-ui`; `web` passes 3,716 tests; `packages/k8s-ui` passes 5,963 with one file (`gpu-ecosystem-registry.test.ts`) failing to load on my Windows CRLF checkout, a file this branch doesn't touch. CI on the PR is waiting for maintainer approval to run.

**Toolkit**

- Tool: Claude Code locally for the live check and test runs; Docker (`golang:1.26`) for the Go suites; later the ship-gate, whose manifest reruns the same four checks (all passed on 2026-11-10).
- What it produced: the live results table, screenshots, and full test output, all in the PR description.
- Difficulties: Smart App Control blocked Go test binaries; CRLF line endings made gofmt and one k8s-ui test misreport on Windows; one mutation script matched the wrong line and showed a false survivor.
- What changed: Go tests moved into Docker on an LF export of the branch; the mutation pattern was fixed and rerun before committing.

---

## Implementation Notes

**2026-11-03**

Chose #2493 after the scout accepted it, posted my claim, installed kind and Go, created the `radar-repro` cluster with a paused `web` Deployment, forked and cloned Radar, and wrote CLAUDE.local.md. Reproduced all three claims on main and posted a short reproduction with my plan and one question. Wrong turn: my first Claude Code session started in the cloud instead of on my machine, so I stopped it and reran locally. Committed the core layer (`SetDeploymentPaused`) at 11:59 PM.

**2026-11-04**

No reply from the maintainer by morning, so I kept building (CONTRIBUTING allows PRs without assignment). Committed the REST layer after moving Go tests into Docker, then the MCP layer, the UI layer with the feature flag, a fix for revisions not refreshing after a resume, and a docs line. Switched to conventional commit messages as CONTRIBUTING asks. Rebased onto 9 new upstream commits (one conflict in the MCP catalog budget), fixed a duplicate "Paused" indicator, ran the full Go suite, and opened PR #2542. Qodo's review flagged a missing connectivity check and a history-style comment; fixed both in two new commits and replied about the badge.

Commits: [feat(k8score)](https://github.com/rishihk/radar/commit/2c45c7fb53cdec4b77aef54aed829ee8036c141a), [feat(server)](https://github.com/rishihk/radar/commit/fcaf0064111ca4a29e594948f88240dda7357cdf), [test(server)](https://github.com/rishihk/radar/commit/2119fdf157ce47c6691938d7c78e42b5271608cd), [feat(mcp)](https://github.com/rishihk/radar/commit/4d111df020c2c718ebe2e34a34287b3f49498d6e), [feat(ui)](https://github.com/rishihk/radar/commit/ac8c18d64bf72fcaaad708ce2428daaacb6c05f9), [fix(ui)](https://github.com/rishihk/radar/commit/886b11fbc89d61319420b7e37d047effe291b2c3), [docs(server)](https://github.com/rishihk/radar/commit/301a8987c18c461ebb6266cf3839202ba22ada6f), [fix(server)](https://github.com/rishihk/radar/commit/f13ee47fcc157da2e267e33e32aa3765137359af), [test(mcp)](https://github.com/rishihk/radar/commit/1a9d0202a3dcea6a52fbb8ce21857224a7c2e80d)

**2026-11-05**

Main moved and the PR showed conflicts, so I rebased onto upstream/main, reran the affected Go packages in Docker and the frontend checks, and force-pushed with `--force-with-lease`. Still no maintainer reply, while seven other outside PRs merged that week, so I kept the branch current rather than chasing reviewers.

**2026-11-06**

Posted one short nudge on the PR: rebased, ready for review, and in need of a maintainer approving the CI run. I also started a backup second contribution (kubernetes/minikube#12632, `--ssh-user` ignored on a reused profile) and confirmed it still reproduces on v1.39.0, but did not post on it once Radar started moving.

**2026-11-10**

hisco merged main into my branch, added a follow-up commit that points the paused-rollback error at the new Resume button, approved, and merged the PR. Issue #2493 closed as completed.

**Toolkit**

- Tool: Claude Code locally, under the commit rules in CLAUDE.local.md (my git identity, no AI trailers, one layer per commit, never push without asking).
- What it produced: nine commits, each one layer or one fix, all tested before committing.
- Difficulties: auto mode blocked `gh pr create` as a public action, so I opened the PR myself; the rebase gave every commit a new committer date.
- What changed: I run anything public (comments, PRs) from my own terminal.

---

## Pull Request

**PR Link:** https://github.com/skyhook-io/radar/pull/2542

**PR Description:** Adds Pause and Resume for Deployments across the UI, REST and MCP, equivalent to `kubectl rollout pause` and `resume`. It lists the change per layer, the live test results on kind, the MCP results, the automated suites, four screenshots, the deviations from the issue (guidance.go unchanged, the added revision-refresh fix, the kind check in the handlers, permission gating that follows Set image), possible follow-ups (Restart's toast while paused, a GitOps revert warning), an AI-use note, and "Closes #2493".

**Maintainer Feedback:**

- 2026-11-04: Qodo's automated review flagged two rule violations and one requirement gap. I fixed the missing `requireConnected` check and the history-style comment in two commits (Qodo marked both resolved) and replied on the PR explaining why the Paused badge yields to the "Rollout paused" chip, offering to show both. nadaverell and hisco were requested as reviewers automatically.
- 2026-11-05 and 2026-11-09: main moved and the PR showed merge conflicts; I rebased onto upstream/main, reran the tests and pushed with `--force-with-lease`.
- 2026-11-06: no maintainer reply after two days, so I posted one nudge: the PR was rebased, ready for review, and needed a maintainer to approve the CI run as a first-time contributor.
- 2026-11-10: maintainer hisco merged main into the branch, added a commit (`fix(rollback): point a paused Deployment's rollback error at Resume`), approved, and merged the PR. His comment: "Thanks, this is a clean, well-tested change, and it's merged. Your calls on the kind check and the badge were right, and we'll take it from here on the follow-ups." Issue #2493 closed as completed.

**Status:** Merged (2026-11-10, by hisco)

**Toolkit**

- Tool: `gh pr create` and `gh pr comment` from my own terminal; Claude Code for a description-against-diff review of the PR draft, since pr-precheck still has the classroom scope.
- What it produced: PR #2542 (9 commits of mine, 25 files, +1,139/−11, about three quarters of it tests; the merged PR also includes the maintainer's merge of main and his follow-up commit) and a reply to the Qodo review.
- Difficulties: pr-precheck refused the wild repo; the draft at first overstated the gap (users could already resume through YAML) and missed some layers in the diff.
- What changed: the diff review corrected five factual mismatches in the description before I posted it.

---

## Learnings and Reflections

**Technical Skills Gained**

How Kubernetes rollout pause works (`spec.paused`, why a paused Deployment holds a new ReplicaSet and refuses rollback, and how the controller records revisions), and how one action threads through a Go backend (a dynamic-client merge patch, chi routes, impersonation and audit logging), an MCP tool schema and a React/TanStack Query UI with feature flags for version skew. I also learned to prove tests are worth something with mutation checks (break the code on purpose and confirm a test fails), to run Go tests in Docker on Linux, to rebase a long-lived PR and push with `--force-with-lease`, and to read the difference between a commit's author date and committer date.

**Challenges Overcome**

Finding an issue at all was the hardest part: my first scan accepted nothing, and the AI/ML repos I wanted were saturated, so I widened my profile and swept the course site against GitHub until a fresh, well-scoped issue turned up. On my machine, Windows Smart App Control blocked Go's test binaries and CRLF line endings broke gofmt and one test, which I solved by moving Go tests into a Linux container instead of weakening the security setting. My course tools' classroom scopes refused the real repo twice, so I installed the wild scopes for repro-check and fell back to a manual diff review for pr-precheck. And the PR sat for days with conflicts piling up; I kept it rebased, answered the automated review, and nudged once, which is what finally got it reviewed and merged.

**What I'd Do Differently Next Time**

I would set up the whole toolchain before I start: the voice guard, the ship-gate and manifest, and the PR monitor with a playbook. Racing to claim a fresh issue meant I did those checks by hand instead, and the tools exist so those decisions don't depend on my mood that day. I'd also commit my course repo as I go, not at the end, and write the story's sections the day each event happens. On the contribution itself, I'd keep the same habits that worked: reproduce before coding, raise every difference from the issue up front, and keep commits small and tested.

---

## Resources Used

- Radar's CONTRIBUTING.md (claiming, PR rules, commit style): https://github.com/skyhook-io/radar/blob/main/CONTRIBUTING.md
- Radar's DEVELOPMENT.md and Makefile, which I followed by hand to build without make: https://github.com/skyhook-io/radar/blob/main/DEVELOPMENT.md
- Radar's CLAUDE.md, the maintainers' conventions file (feature flags for version skew, the four run modes)
- The rollback refusal PR the issue builds on, linked from #2493 as #2363
- Kubernetes docs on pausing and resuming a Deployment rollout: https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#pausing-and-resuming-a-rollout-of-a-deployment
- kind quick start, for the local cluster: https://kind.sigs.k8s.io/docs/user/quick-start/
