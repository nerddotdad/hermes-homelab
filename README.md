# hermes-homelab

> **Source of truth** for the `ghcr.io/nerddotdad/hermes-homelab` image. The [truecharts](https://github.com/nerddotdad/truecharts) repo only pins the GHCR tag in HelmRelease manifests; edit Docker sources here.


Extends [nesquena/hermes-webui](https://github.com/nesquena/hermes-webui) for homelab on-call:

- **[Hermes Agent](https://github.com/NousResearch/hermes-agent)** cloned to `/opt/hermes` at image build (`run_agent` import)
- **Pre-baked `/apptoo/venv`** — WebUI + `hermes-agent[all]` + plugins deps installed at build time (rsync’d to `/app` on start; skips runtime `uv pip install`)
- **Dashboard SPA built at image build** — `web/` → `hermes_cli/web_dist` (required for `hermes dashboard` on `:9119`)
- `HERMES_BUNDLED_SKILLS=/opt/hermes/skills` — upstream `skills_sync` excludes paths containing `venv`; bundled skills sync from the agent checkout instead
- `kubectl` + `flux` + `jq` CLI (legacy in-pod terminal; **prefer Hearth sandbox MCP** for cluster triage)
- Homelab skill `homelab-k8s-flux-triage` (seeded from image `skills/`) instructs agents to use Hearth `sandbox_exec`
- MCP client → `http://hearth.observability.svc.cluster.local:8000/mcp` (patched by `ensure-homelab-config.py`)
- Incident investigations are started by **Hearth** (`ghcr.io/nerddotdad/hearth`) via the WebUI API; open chats with `?session_id=<id>`
- Common shell utilities for the agent terminal: `tail`, `head`, `less`, `grep`, `awk`, `sed`, `find`, `ps`, `top`, `ip`, `ping`, `dig`, `nc`, `vim.tiny`, `tar`, `gzip`
- **Hermes Agent dashboard** on `:9119` (`hermes dashboard --host 0.0.0.0 --insecure --tui`) — exposed via separate ingress `hermes-dash.${DOMAIN_0}`
- Ollama provider seed (`qwen3.5:9b` → in-cluster Ollama), SearXNG web search, terminal `env_passthrough` for homelab secrets

Built by **Build Image** (`.github/workflows/build-image.yml`) on push to `main` or manual **workflow_dispatch**.

- **`VERSION`** + CI publish `x.y.z` to GHCR (auto patch bump when you did not edit `VERSION` in the commit)
- **Renovate** updates `hermes-oncall/app/helm-release.yaml` (`tag: x.y.z@sha256:…`) when a newer tag is on GHCR

Deploy: `clusters/main/kubernetes/my-apps/ai/hermes-oncall/`
