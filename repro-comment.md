Reproduced on main (`2fa4ccf`) against a local kind cluster (Kubernetes v1.37.0) with a Deployment paused via `kubectl rollout pause`:

- **UI:** no Pause or Resume action on the Deployment page. `POST /api/workloads/deployments/default/web/pause` and `/resume` return 404.
- **MCP:** `manage_workload` rejects both: `unknown action "pause": must be restart, scale, or rollback`.
- **Rollback:** refused on REST (409), MCP and the UI with the #2363 message. As a control, the same rollback returns 200 once the Deployment is resumed.

A few things I noticed that differ slightly from the issue, which I'd like to account for in the plan:

1. A UI user can already resume by editing `spec.paused` in the YAML tab, so the gap is a direct action rather than a hard kubectl requirement.
2. The Overview already shows "Rollout paused", but the header and the Resources list show plain "Healthy". I'd put the "Paused" badge in the header next to the existing status.
3. Restart doesn't gate the button on patch permission for Deployments today; Set image uses `workloadWrites` from `useNamespacedCapabilities`. I'd use that for Pause/Resume.
4. The button itself lives in `ResourceActionsBar.tsx`, and per CLAUDE.md the new endpoints need a `FeatureCapabilities` flag plus a `radarFeatures.ts` entry. I'd also update `mcpToolCatalog.ts` and the action list in `pkg/investigation/guidance.go`.
5. For the routes, I'd follow the siblings: `/workloads/{kind}/{namespace}/{name}/pause` and `/resume` with `NormalizeWorkloadKind`, returning 400 for anything that isn't a Deployment. One question: the Done list wants a `pkg/k8score` test that rejects non-Deployment kinds, but `SetDeploymentPaused(ctx, namespace, name, paused)` takes no kind. Should it take a kind and reject the others there, or is the handler check enough?

Does that match what you had in mind? If so I'll start on it.
