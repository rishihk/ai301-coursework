**Title:** Add Pause and Resume actions for Deployments

## Description

Adds **Pause** and **Resume** for Deployments in the UI, REST and MCP, equivalent to `kubectl rollout pause` and `kubectl rollout resume`.

Radar refuses to roll back a paused Deployment and tells the user to resume it first, but until now Radar had no way to resume one, so a UI user had to edit `spec.paused` in the YAML tab or switch to kubectl. This closes that gap.

Scope is Deployments only. StatefulSets and DaemonSets have no pause, and Argo Rollouts keep their own pause handling.

### What changed, per layer

| Layer | Change |
|---|---|
| Core | `SetDeploymentPaused` in `pkg/k8score/workload.go`: a merge patch of `spec.paused`, plus the `SetDeploymentPausedWithClient` wrapper in `internal/k8s/update.go`. It does not read first, so it only needs `patch`; repeating a call is a no-op on the API server. |
| REST | `POST /api/workloads/{kind}/{namespace}/{name}/pause` and `/resume`, following the sibling routes and `NormalizeWorkloadKind`. Non-Deployments get a 400, a missing Deployment a 404, and an RBAC denial a 403. Same audit log and impersonation as Restart. The routes are added to the Workloads line in `CLAUDE.md`. |
| MCP | `manage_workload` actions `pause` and `resume`, Deployment only. Updated the schema, tool description, `docs/mcp.md` and `mcpToolCatalog.ts`. The MCP catalog budget goes up 89 bytes. |
| Version skew | `deploymentPause` flag in `FeatureCapabilities` (set in `handleCapabilities`, typed in k8s-ui's `types/core.ts`) and `radarFeatures.ts`, so a newer UI hides the button on an older Radar instead of showing an error. |
| UI client | `usePauseDeployment` and `useResumeDeployment` in `web/src/api/client.ts`, guarded by `useRadarFeature`. |
| UI | One button in `ResourceActionsBar` reading **Pause rollout** or **Resume rollout** from `spec.paused`, gated on `workloadWrites` like Set image (`canPauseDeployment` in `bulk-workload-actions.ts`) and hidden while the Deployment is being deleted. |
| Badge | A **Paused** `<Badge>` in the Deployment header (full view and drawer). It only appears when nothing is pending: a held rollout already reads "Rollout paused" in the status chip, so showing both would repeat it. |
| Revisions | `useRefreshRevisionsOnNewRollout`: see Deviations. |

## Type of change

- [ ] Bug fix (non-breaking change that fixes an issue)
- [x] New feature (non-breaking change that adds functionality)
- [ ] Breaking change (fix or feature that would cause existing functionality to change)
- [ ] Documentation update

## How has this been tested?

- [x] Tested locally with minikube/kind
- [ ] Tested against a remote cluster
- [x] Added/updated unit tests

### Live test results

kind cluster, Radar built from this branch, Deployment `default/web`.

| Step | Result |
|---|---|
| Pause from the UI | `spec.paused=true`; the button flips to **Resume rollout** and the badge appears |
| Change the image while paused (`kubectl set image`) | Template moved to `nginx:1.29-alpine`; the same ReplicaSet and pod kept running `nginx:1.27`; `rollout status` reported "0 out of 1 new replicas have been updated" and timed out |
| Header while the rollout is held | Shows "Rollout paused" only, with no duplicate badge |
| Roll back while paused (REST) | HTTP 409: "workload is paused; resume it first…" |
| Resume from the UI | Rollout starts and finishes: revision 7 on `nginx:1.29-alpine` |
| Revision list after resume, page not reloaded | Revision 7 appears; one extra `/revisions` request, sent when the new revision was recorded |
| Roll back after resume (UI) | Rolled back to revision 6's `nginx:1.27`; now revision 8 |
| Pause a StatefulSet, resume a Rollout (REST) | HTTP 400: "only Deployments can be paused or resumed" |
| Pause a missing Deployment (REST) | HTTP 404 |
| Cross-site browser request (REST) | HTTP 403 |

MCP, through JSON-RPC to `/mcp`:

| Call | Result | `generation` |
|---|---|---|
| `pause` | OK | 19 |
| `pause` again | OK, no change | 19 |
| `rollback` while paused | Error: "workload is paused; resume it first…" | 19 |
| `resume` | OK, with `next: wait_for` | 20 |
| `resume` again | OK, no change | 20 |
| `pause` on a StatefulSet | Error: "pause is only supported for Deployments" | 20 |
| `pause` on a missing Deployment | Error: not found | 20 |

### Automated tests

| Suite | Result |
|---|---|
| `go test ./...` at the repo root | Pass, no failures |
| `go test ./...` in `pkg/` | Pass, no failures |
| `go vet` on `internal/server`, `internal/mcp`, `internal/k8s` | Clean |
| `npm run tsc` in `web` and `packages/k8s-ui` | 0 errors |
| `web` | 3716 passed |
| `packages/k8s-ui` | 5963 passed, 1 skipped; `gpu-ecosystem-registry.test.ts` fails to load |

The Go tests ran in `golang:1.26` on Linux, on an LF export of this branch's head (`301a8987c`). The frontend tests ran on Windows, where `gpu-ecosystem-registry.test.ts` fails on a CRLF checkout: it splits on `\n` and the registry header keeps its `\r`. This branch does not touch that file.

I also checked that the tests catch regressions by breaking the code on purpose (wrong kind check, inverted action, Radar's own client instead of the caller's, removed routes, duplicate badge, and so on). Each broken version failed a test.

### Not tested

- A real auth-enabled deployment. Tests confirm the patch reaches the API server as `Impersonate-User: alice` and that Radar's own client is never used instead, but against a fake API server.
- Radar Hub. The older-Radar behavior is covered by tests only.

### The four ways Radar runs

- **Local:** tested live, as above.
- **In-cluster shared:** the patch runs as the caller, so Kubernetes RBAC decides, and a denial returns 403.
- **Radar Hub:** the `deploymentPause` flag hides the button on an older Radar. The new `pkg/` method and `ResourceActionsBar` props are additive and optional. The Paused badge reads only `spec.paused`, so it also shows against an older Radar.
- **Shared without authentication:** the routes act with Radar's own identity, like Restart, and `requireSameOrigin` refuses a request from a page on another site (tested).

## Screenshots

- `01-unpaused-pause-rollout-button.jpg`: unpaused, showing **Pause rollout**
- `02-paused-badge-resume-rollout-button.jpg`: paused with nothing pending, showing the **Paused** badge and **Resume rollout**
- `03-image-change-held-while-paused.jpg`: an image change held while paused, "Paused at 0/1 updated", old pod still running
- `04-new-revision-listed-without-reload.jpg`: the new revision listed in Rollback without a page reload

## Deviations

1. **`pkg/investigation/guidance.go` is unchanged.** Adding pause to its `manage_workload` line would push the exec-enabled server instructions past the 3,440-character limit that `TestServerInstructionsFitClientLimits` enforces so Radar Hub's preamble fits. Only 4 characters were free, and the shortest accurate wording needs 7. Agents still find pause and resume in the `manage_workload` description.
2. **Added a revision-refresh fix (`fix(ui)`), which the issue didn't ask for.** The page fetched revisions once on load, so after a resume created a new revision, Rollback stayed disabled or out of date until a reload, which blocks the "rollback works again after resume" step. It now refetches when the Deployment's `deployment.kubernetes.io/revision` annotation changes. Refetching straight after the resume request would run before the controller creates the ReplicaSet. This also helps rollouts started outside Radar.

Also worth checking against the issue:

- **The kind check is in the REST and MCP handlers, not `pkg/k8score`.** `SetDeploymentPaused(ctx, namespace, name, paused)` takes no kind, as the issue specifies, so the "non-Deployment kind is rejected" test lives at the handler level rather than in `pkg/k8score`. I asked about this on the issue and it's easy to move if you prefer it in core.
- **Permission gating follows Set image, not Restart.** The issue says to follow Restart's permission checks, but Restart has none for Deployments: its button always shows. The new button uses `workloadWrites`, as Set image does.

## Possible follow-ups

- **Restart while paused:** from reading the code, the toast still says "Workload restarting" even though nothing rolls out until resume. Not changed or tested here.
- **GitOps warning:** Pause and Resume show no GitOps revert warning, unlike Set image, so a manifest that sets `spec.paused` could undo a pause.

## Checklist

- [x] My code follows the project's coding standards
- [x] I have performed a self-review of my code
- [x] I have added comments where necessary
- [x] My changes generate no new warnings
- [x] Any dependent changes have been merged (none)

## AI use

I used AI tools like Claude to help explore the repository and code, support the implementation, and plan and review the tests. I reviewed the changes and the validation results myself.

## Related issues

Closes #2493
