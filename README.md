# Self-hosted AI Package

**Self-hosted AI Package** is a docker-compose template for running a complete
local-first AI workstation. It bundles everything needed to iterate on
retrieval-augmented agents with Flowise, serve models with Ollama, experiment
with Open WebUI, collect telemetry in Langfuse, explore the open web via
SearXNG, and persist knowledge in Qdrant — all without depending on n8n or
Supabase.

The repository is tailored for Cole's "agentic" workflow demos and keeps the
stack opinionated but lightweight. Flowise ships with a ready-to-import
agentic RAG chatflow, Qdrant is pre-configured as the canonical vector store,
and helper scripts make spinning up the stack on CPU or GPU straightforward.

## What’s included

✅ [**Flowise**](https://flowiseai.com/) – Low/no-code canvas for composing
agentic pipelines. The repository includes a RAG + tooling chatflow export to
give you a production-ready starting point.

✅ [**Open WebUI**](https://openwebui.com/) – ChatGPT-style interface wired to
your local Ollama models. Perfect for ad-hoc prompts or quick evaluation
loops.

✅ [**Ollama**](https://ollama.com/) – Cross-platform LLM runtime used for both
chat completions and local embeddings.

✅ [**Qdrant**](https://qdrant.tech/) – High-performance vector database with a
simple REST API. It is the only vector store the starter kit depends on now
that Supabase has been removed.

✅ [**Langfuse**](https://langfuse.com/) – Observability layer for tracing agent
runs, prompts, and tool calls.

✅ [**SearXNG**](https://searxng.org/) – Privacy-preserving metasearch engine for
augmenting the agent with fresh web results.

✅ [**Caddy**](https://caddyserver.com/) – Optional ingress with automatic TLS
for exposing individual services.

## Important Links

- [Local AI community](https://thinktank.ottomator.ai/c/local-ai/18)
- [GitHub Kanban board](https://github.com/users/coleam00/projects/2/views/1)
- [Original Local AI Starter Kit](https://github.com/n8n-io/self-hosted-ai-starter-kit)
  that inspired this project

## Prerequisites

Before you begin, make sure you have the following installed:

- [Python](https://www.python.org/downloads/)
- [Git](https://git-scm.com/downloads) or [GitHub Desktop](https://desktop.github.com/)
- [Docker / Docker Desktop](https://www.docker.com/products/docker-desktop/)

## Installation

Clone the repository and set up your environment variables:

```bash
git clone https://github.com/coleam00/local-ai-packaged.git
cd local-ai-packaged
cp .env.example .env
```

Edit `.env` and replace the placeholder secrets. The stack now only requires
secrets for the Langfuse/Postgres/ClickHouse/MinIO cluster and optional Caddy
hostnames. Every value can be self-generated with tools like `openssl
rand -hex 32`.

## Starting the stack

All services can be launched with the helper script:

```bash
python start_services.py --profile cpu
```

Supported profiles:

- `cpu` *(default)* – run everything without GPU acceleration
- `gpu-nvidia` – expose a single NVIDIA GPU to the Ollama container
- `gpu-amd` – use the ROCm-compatible Ollama image
- `none` – skip the profile flag entirely and let Docker choose defaults (useful
  when you run Ollama on the host and only need the other services)

If you’re running Ollama directly on macOS, set `OLLAMA_HOST=host.docker.internal:11434`
in Open WebUI and Flowise after the containers start so they point at the host
runtime.

## Flowise agentic RAG chatflow

The repository ships with `flowise/Web Search + Local Agent Chatflow.json`, an
opinionated Flowise export that demonstrates:

- Local embeddings + completions through Ollama
- Hybrid retrieval with Qdrant collections and a Brave Search action
- Tooling hooks for Slack webhooks, conversation fetching, and local document
  authoring

Import the custom tools in `flowise/*.json` first, then load the chatflow JSON
into Flowise. Configure the environment variables called out in the node
descriptions (Slack webhook, Slack token, Brave Search key, Qdrant API URL/key
if you changed defaults), and you’ll have a fully agentic RAG assistant ready
to extend. A detailed walkthrough lives in
`docs/workflows/Flowise_Agentic_RAG.md`.

## Data locations

- **Flowise storage** – persisted under `~/.flowise` on the host
- **Open WebUI history** – persisted in the `open-webui` Docker volume
- **Qdrant vectors** – stored in the `qdrant_storage` volume
- **Langfuse data** – stored in the ClickHouse, Postgres, and MinIO volumes

Back up the Docker volumes or use `docker compose cp` commands to snapshot data
before destroying the stack.

## Deploying to the cloud

When provisioning a remote machine (Ubuntu recommended):

1. Harden the firewall and open the ports you plan to expose:
   ```bash
   sudo ufw enable
   sudo ufw allow 3000  # Open WebUI
   sudo ufw allow 3001  # Flowise
   sudo ufw allow 3002  # Langfuse
   sudo ufw allow 6333  # Qdrant API
   sudo ufw allow 8080  # SearXNG
   sudo ufw allow 11434 # Ollama
   sudo ufw allow 80    # HTTP (for Caddy)
   sudo ufw allow 443   # HTTPS (for Caddy)
   sudo ufw reload
   ```
2. Populate `.env` with production-ready secrets and hostnames.
3. Run `python start_services.py --profile <cpu|gpu-nvidia|gpu-amd>` to launch
   the stack.

## Troubleshooting

- **SearXNG first run** – the helper script temporarily comments out
  `cap_drop: - ALL` so the container can generate `uwsgi.ini`. It re-enables the
  setting on subsequent runs.
- **Langfuse bootstrapping** – supply `LANGFUSE_INIT_*` values in `.env` if you
  want the web UI to auto-create an org and project.
- **Qdrant credentials** – by default the container exposes an unauthenticated
  API on `http://localhost:6333`. Update the `QDRANT_API_KEY` environment
  variable in Flowise/Open WebUI if you enable authentication.

That’s it! With n8n and Supabase removed, the stack is leaner and easier to run
on constrained hardware while still supporting powerful agentic workflows.
