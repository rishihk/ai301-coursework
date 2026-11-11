# Repro draft: skyhook-io/radar #2493 (Pause and Resume for Deployments)

Status: draft, not posted. Run 2026-11-03 on my machine (Windows 11, local kind cluster).

## Result

| Claim in the issue | Result |
|---|---|
| No Pause or Resume action in the UI | **Confirmed.** No such button on the Deployment page, and the routes return 404. |
| `manage_workload` has no pause/resume | **Confirmed.** The tool rejects both as unknown actions. |
| Rollback is refused for a paused Deployment (#2363) | **Confirmed** on REST, MCP and the UI. The UI path needed a second Deployment, because `web` has one revision. |
| A UI user has to switch to kubectl to resume | **Partly.** There is no dedicated action, but the YAML editor resumed a paused Deployment. |

## Environment

| | |
|---|---|
| OS | Windows 11 Home, 10.0.26300 (Git Bash; no `make`) |
| Go | go1.27.3 windows/amd64 (`go.mod` says 1.26.0) |
| Node / npm | v26.3.0 / 11.16.0 (CONTRIBUTING.md says 24+) |
| kind | v0.33.0 (go1.26.7) |
| Node image | kindest/node:v1.37.0@sha256:a1ed56cfb0e7b93589bdf97c8cd566405a265939e3620fc4f5de89adff580ae5 |
| Kubernetes | server v1.37.0, kubectl v1.36.1, containerd 2.3.4 |
| Docker | server 29.8.0 |
| Radar | `2fa4ccf70d1473a1b0cecfb495f742c646b6cd74` (`v1.19.0-65-g2fa4ccf70`), equal to `upstream/main` when fetched |
| Cluster | `kind-radar-repro`, one node |
| Browser | Chromium-based browser |

Fixture before the run: `default/web`, `paused=true`, `nginx:1.27`, 1/1 ready, one revision, `generation=2`, `resourceVersion=569`.

## Steps

### 1. Cluster

Created before the run with:

```bash
kind create cluster --name radar-repro
kubectl create deployment web --image=nginx:1.27
kubectl rollout pause deployment/web
```

### 2. Build and run from main

```bash
npm ci
(cd web && npm run build)
rm -rf internal/static/dist; mkdir -p internal/static/dist; cp -r web/dist/* internal/static/dist/
go build -ldflags "-X main.version=$(git describe --tags --match 'v*' --always --dirty)" -o radar.exe ./cmd/explorer
kubectl config use-context kind-radar-repro
./radar.exe --no-browser --port 9280
```

The rest of the steps run in a second terminal.

### 3. UI: open the Deployment

Open `http://localhost:9280/workload/deployments/default/web`.

Action bar: **Set image, Restart, Rollback** (disabled), plus Compare and Delete icons. A DOM search for buttons or menu items matching `pause` or `resume` found none. Searching `web/src`, `packages/k8s-ui/src`, `pkg`, `internal` and `docs` for `usePauseDeployment`, `useResumeDeployment`, `Pause rollout`, `Resume rollout` and `SetDeploymentPaused` also found nothing.

### 4. REST

```bash
curl -s -i -X POST http://localhost:9280/api/workloads/deployments/default/web/pause
curl -s -o /dev/null -w "%{http_code}\n" -X POST http://localhost:9280/api/workloads/deployments/default/web/resume
curl -s -i -X POST -H 'Content-Type: application/json' -d '{"revision":1}' http://localhost:9280/api/workloads/deployments/default/web/rollback
```

```text
pause:    HTTP/1.1 404 Not Found   (404 page not found)
resume:   404
rollback: HTTP/1.1 409 Conflict
{"error":"Deployment default/web: workload is paused; resume it first (kubectl rollout resume, or set spec.paused to false) and retry, because a paused Deployment holds a new template without rolling it out"}
```

### 5. MCP

Sent as JSON-RPC with curl, not through an agent client. `tools/list` first, then `tools/call`:

```bash
curl -s -X POST http://localhost:9280/mcp -H 'Content-Type: application/json' -H 'Accept: application/json, text/event-stream' -d '{"jsonrpc":"2.0","id":2,"method":"tools/list","params":{}}'
curl -s -X POST http://localhost:9280/mcp -H 'Content-Type: application/json' -H 'Accept: application/json, text/event-stream' -d '{"jsonrpc":"2.0","id":3,"method":"tools/call","params":{"name":"manage_workload","arguments":{"action":"pause","kind":"deployment","namespace":"default","name":"web"}}}'
```

Repeat the second command with `"action":"resume"`, and with `"action":"rollback"` plus `"revision":1`.

```text
action schema: "action to perform: restart, scale, or rollback"
pause:    unknown action "pause": must be restart, scale, or rollback          (isError: true)
resume:   unknown action "resume": must be restart, scale, or rollback         (isError: true)
rollback: rollback failed: Deployment default/web: workload is paused; resume it first (kubectl rollout resume, or set spec.paused to false) and retry, because a paused Deployment holds a new template without rolling it out   (isError: true)
```

After steps 3 to 5, `web` still had `paused=true`, `nginx:1.27`, `generation=2`, `resourceVersion=569`.

### 6. UI rollback on a paused Deployment with two revisions

Rollback is disabled on `web` because it has one revision, so I used a throwaway Deployment and deleted it afterwards:

```bash
kubectl create namespace radar-repro-scratch
kubectl -n radar-repro-scratch create deployment web2 --image=nginx:1.27
kubectl -n radar-repro-scratch rollout status deploy/web2 --timeout=90s
kubectl -n radar-repro-scratch set env deploy/web2 REPRO=second
kubectl -n radar-repro-scratch rollout status deploy/web2 --timeout=90s
kubectl -n radar-repro-scratch rollout pause deploy/web2
```

Open `http://localhost:9280/workload/deployments/radar-repro-scratch/web2`, click **Rollback**, then **Rollback** on row #1, then **Confirm**.

```text
Request: POST /api/workloads/deployments/radar-repro-scratch/web2/rollback -> 409 Conflict
Toast:   Failed to rollback workload
         Deployment radar-repro-scratch/web2: workload is paused; resume it first (kubectl rollout resume, or set spec.paused to false) and retry, because a paused Deployment holds a new template without rolling it out
State:   paused=true, revision=2, env REPRO=second (unchanged)
```

kubectl refuses the same Deployment:

```bash
kubectl -n radar-repro-scratch rollout undo deploy/web2 --to-revision=1
```

```text
error: you cannot rollback a paused deployment; resume it first with 'kubectl rollout resume' and try again
(exit 1)
```

### 7. Resume through the YAML editor, then the control rollback

On `web2`'s YAML tab: **Edit**, change `paused: true` to `paused: false`, **Review changes**, **Apply reviewed changes**.

```text
PUT /api/resources/deployments/radar-repro-scratch/web2?resourceVersion=2479&reviewedContext=kind-radar-repro -> 200 OK
State: spec.paused empty, generation=4
```

Same rollback, now that it's resumed:

```bash
curl -s -i -X POST -H 'Content-Type: application/json' -d '{"revision":1}' http://localhost:9280/api/workloads/deployments/radar-repro-scratch/web2/rollback
```

```text
HTTP/1.1 200 OK
{"message":"Rollback to revision 1 initiated"}
State: revision=3, env empty, rollout finished
```

So the pause is what blocks the rollback.

### 8. Cleanup

```bash
kubectl delete namespace radar-repro-scratch
```

`web` afterwards: `paused=true`, `nginx:1.27`, `generation=2`, `resourceVersion=569`, one revision.

## What didn't match the issue

1. **Resuming doesn't strictly require kubectl.** The issue says a UI user has to switch to kubectl. The YAML editor did it (step 7), so the gap is that there's no direct action and the refusal message only names kubectl or `spec.paused`.
2. **The UI already shows the paused state, in part.** The Overview's State card says "Rollout paused / No rollout is pending", the Runtime section repeats it, and the YAML tab has a banner. The page header has no badge, and the Resources list shows both paused Deployments as plain "Healthy". The issue's "Paused" badge would add to this, so the plan should say where it goes.
3. **The refusal comes late in the UI.** The revision dialog offers Rollback and Confirm on a paused Deployment with no warning. The error only appears after Confirm, and the dialog stays open on the Confirm row.
4. **The fixture can't reach the UI refusal.** With one revision, Rollback is disabled. The tooltip "Only one revision exists" comes from reading `ResourceActionsBar.tsx:483`; I didn't see it rendered. REST and MCP do refuse on `web`, because the pause check runs before any revision lookup.

Differences from reading the code only, not exercised at runtime:

5. **Restart has no permission check to copy for Deployments.** The issue says to follow Restart's checks so the button hides or disables without patch access. The button renders whenever `onRestart` is set (`ResourceActionsBar.tsx:462`), and only Rollouts withhold it (`WorkloadView.tsx:332`, `:419`). `workloadWrites` (`WorkloadView.tsx:307`) is what Set image uses.
6. **The button isn't in `WorkloadView.tsx`.** That file passes the callback; the button is in `packages/k8s-ui/src/components/shared/ResourceActionsBar.tsx`.
7. **Layers the issue doesn't list.** `internal/k8s/update.go` (the `*WithClient` wrapper), the version-skew flag (`FeatureCapabilities` plus `web/src/api/radarFeatures.ts`, required by CLAUDE.md), the MCP catalog `web/src/components/home/mcpToolCatalog.ts:339`, and `pkg/investigation/guidance.go:41`, which lists what `manage_workload` does.
8. **Signature and route.** `SetDeploymentPaused(ctx, namespace, name, paused)` takes no kind, but the Done list wants a k8score test rejecting non-Deployment kinds. The issue writes `/workloads/Deployment/...`, while sibling routes use `{kind}` and normalize it.

## Things that went differently on my side

- The first `go build` failed: `fork/exec C:\Program Files\Go\pkg\tool\windows_amd64\link.exe: An Application Control policy has blocked this file.` I changed nothing, scoped `go test` runs linked fine right after, and the same build then succeeded. This is my machine, not Radar.
- On Windows the build output is `radar.exe`, not the Makefile's `radar`, and `make` steps were run by hand.
- I read the toast text from the DOM with a mutation observer. My screenshot showed the dialog but not the toast.
- Scoped baseline tests passed on main: `TestRollbackPausedDeploymentIsRefused`, `TestRestartWorkload_RolloutUsesRestartAt`, `TestWorkloadRollbackErrorStatus`, `TestRestartTerminatingRolloutReturns409`, `TestRollbackResult`, `TestRegisteredToolAnnotations`, and `npm run tsc`. I did not run the full suite.

## Not tested

- Auth-enabled mode, a user without patch access, Radar Hub, and the shared no-auth deployment. Everything ran as local mode with my kubeconfig identity.
- Restart or Set image on a paused Deployment, and whether their copy explains that the change is held.
- Holding a template change while paused and rolling it out on resume. That's the issue's Done check for after the fix.
