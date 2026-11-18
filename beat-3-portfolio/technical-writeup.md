# Adding Pause and Resume to a Kubernetes Dashboard's AI Agent Interface

## Overview

Radar is an open-source Kubernetes dashboard with about 4,000 GitHub stars. It also has a built-in Model Context Protocol (MCP) server, which lets AI agents operate a cluster through it. Radar could restart, scale and roll back a Deployment, but it had no way to pause or resume one, and since an earlier change it refused to roll back a paused Deployment. A user who paused a rollout was stuck until they left Radar for kubectl. I found the issue ([#2493](https://github.com/skyhook-io/radar/issues/2493)) with issue-scout, a tool I built this term whose search strategy and rubric I wrote, and I added Pause and Resume across Radar's Go core, REST API, MCP server and React UI. The pull request, [#2542](https://github.com/skyhook-io/radar/pull/2542), was approved and merged by a maintainer on November 10, 2026.

## What happened

**Finding the issue.** My scout's first real scan on November 2 graded five issues and accepted none:

```text
Scan finished: 0 accepted, 5 rejected. Every one of the top five already has someone on it:
a recent claim comment, an open pull request, or both.
```

GitHub's search results only show assignees and formally linked pull requests, so my queries couldn't see claim comments. I rewrote my search strategy to exclude linked PRs with `-- "-linked:pr"`, drop stale and long threads, and widen from AI/ML repos to Kubernetes and DevOps work, which matches my day job. A wider sweep turned up Radar #2493, opened the day before. The scout graded it `"verdict": "accept"` with difficulty "moderate". I claimed it on November 3.

**Proving the gap was real.** Before writing any code, I built Radar from `main` (commit `2fa4ccf`), ran it against a local kind cluster with a paused Deployment, and hit every surface:

```text
POST .../web/pause     HTTP/1.1 404 Not Found
MCP manage_workload    unknown action "pause": must be restart, scale, or rollback
POST .../web/rollback  HTTP/1.1 409 Conflict
  "workload is paused; resume it first (kubectl rollout resume, or set spec.paused to false)..."
```

The reproduction also showed four places where the code differed from the issue. For example, the button the issue pointed to actually lives in `ResourceActionsBar.tsx`, and the Restart action it said to copy had no permission check. I posted those differences, with my plan and one design question, on the issue before writing code.

**Building it.** The core is an idempotent merge patch through Kubernetes' dynamic client. Repeating a pause changes nothing, so there's no need to read the object first:

```go
patch := fmt.Sprintf(`{"spec":{"paused":%t}}`, paused)
if _, err := m.dynClient.Resource(deploymentGVR).Namespace(namespace).Patch(
    ctx, name, types.MergePatchType, []byte(patch), metav1.PatchOptions{},
); err != nil {
    return fmt.Errorf("failed to patch deployment: %w", err)
}
```

On top of that I added REST routes that patch as the calling user through impersonation, so Kubernetes' own permissions decide who can pause. I also added `pause` and `resume` actions to the MCP server's `manage_workload` tool for AI agents, a feature flag so newer UIs don't show the button against older servers, and a permission-gated Pause/Resume button. Windows Smart App Control blocked Go's test binaries on my laptop, so I ran every Go test in a `golang:1.26` Docker container instead of turning off the security setting.

To check that the tests actually meant something, I broke the code on purpose and confirmed each break failed a test:

```text
KILLED  pause handler sends paused=false      <- TestPauseAndResumeDeployment
KILLED  uses Radar client, no impersonation   <- TestPauseDeploymentNeverFallsBackToRadarsIdentity
KILLED  rejection returns 422, not 400        <- TestPauseAndResumeRejectNonDeployments
```

All 19 backend variants were caught, plus 24 more across the MCP server and UI. The live re-run on kind passed end to end: pause, change the image with nothing rolling out, resume, then roll back.

**Changing my fix after review.** I opened the PR on November 4, and the repo's automated reviewer, Qodo, flagged that my handler reached the cluster "without calling s.requireConnected(w) first". That's a house rule I had missed, because the Restart handler I copied skips it too. I followed the handlers that do check, Rollback and revisions:

```go
// before
func (s *Server) setDeploymentPaused(w http.ResponseWriter, r *http.Request, paused bool) {
	kind := chi.URLParam(r, "kind")

// after (commit fe8ab24)
func (s *Server) setDeploymentPaused(w http.ResponseWriter, r *http.Request, paused bool) {
	if !s.requireConnected(w) {
		return
	}
	kind := chi.URLParam(r, "kind")
```

I added a test for the disconnected case, and Qodo marked the finding resolved. My own live check also forced a second version of the UI: the header showed "Paused" and "Rollout paused" side by side during a held rollout, so I changed the badge to step aside when the status chip already says it.

**The result.** Main kept moving, so I rebased twice. On November 10 maintainer hisco approved and merged the PR, added a follow-up commit that points the paused-rollback error at the new Resume button, and wrote:

> "Thanks, this is a clean, well-tested change, and it's merged. Your calls on the kind check and the badge were right, and we'll take it from here on the follow-ups."

Issue #2493 closed as completed.

## Limitations and what's next

AI agents can now pause and resume, but Radar's agent guidance text doesn't mention it. The instructions sit at a hard 3,440-character limit with only 4 characters free, so agents learn about pause only from the tool's own description. I also tested only against fake API servers for the logged-in and Radar Hub modes, not a real authenticated deployment. Two follow-ups are still open: Restart's message while a Deployment is paused, and the lack of a GitOps warning when a pause could be reverted.

My process had limits too. My scout's maintainer check relies on GitHub's role label, so it called Radar's real maintainer a non-maintainer. And I nudged after 2 days, when this repo's outside PRs median 17.6 days to a first response. Next, I'd patch the scout to count code owners and recent committers, set my nudge threshold from repo data before opening a PR, and propose a shorter guidance line to Radar's maintainers so agents learn about pause directly.
