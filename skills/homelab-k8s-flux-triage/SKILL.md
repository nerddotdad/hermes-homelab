---
name: homelab-k8s-flux-triage
description: Read-only Kubernetes/Flux triage via the Hearth sandbox MCP tools.
---

# Homelab Kubernetes / Flux triage

You are investigating a homelab incident. Prefer **Hearth sandbox MCP tools** over the local terminal for cluster inspection.

## Tools (preferred)

Use MCP tools from server `hearth` (names may be prefixed `mcp_hearth_`):

| Tool | Use |
|------|-----|
| `sandbox_ensure` | Ensure the incident sandbox pod is running (`incident_id` required) |
| `sandbox_status` | Check sandbox readiness / TTL |
| `sandbox_exec` | Run shell in the sandbox (`kubectl`, `flux`, `jq`, `curl`, …) |

Always pass the **incident id** from the investigate prompt.

Example:

```text
sandbox_exec incident_id=<id> command="kubectl get pods -A | head -50"
sandbox_exec incident_id=<id> command="flux get helmreleases -A"
sandbox_exec incident_id=<id> command="kubectl -n flux-system get kustomizations"
```

## Rules

1. **Read-only** — get/list/logs/describe only. Do not patch, delete, scale, or apply live.
2. Prefer **GitOps** remediation suggestions (edit truecharts manifests) over live cluster mutation.
3. Use `HOMELAB_DOCS_BASE_URL` / runbook links from the alert when present.
4. Local Hermes `terminal` is OK for non-cluster tasks; for kubectl/flux use **sandbox_exec**.

## Typical flow

1. `sandbox_ensure` for the incident
2. Inspect failing workload / HelmRelease / events with `sandbox_exec`
3. Summarize root cause and proposed GitOps fix in the chat
