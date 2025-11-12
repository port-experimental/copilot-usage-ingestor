# AI Consumption in Port — Go Ingestion (GitHub Copilot + Microsoft 365 Copilot)

This repository bundles everything needed to ingest GitHub Copilot seats + usage and Microsoft 365 Copilot usage into Port with a Go worker and opinionated dashboards.

## Repository layout
- `docs/` — ordered runbook pages (`01-Sequence.md`, tokens, dashboard, validation, references, etc.).
- `configs/blueprints/` — Port blueprint JSON files to apply via Builder/API.
- `configs/mappings/` — mapping override + webhook payload definitions for seats and M365.
- `worker/go-ingestor/` — Go implementation (`main.go`, `go.mod`, `config.example.env`, Dockerfile, compiled helper binary).
- `deploy/` — automation examples for GitHub Actions and a Kubernetes CronJob.

## Recommended reading order
1. `docs/01-Sequence.md` — step-by-step execution plan (blueprints → mappings → secrets → deploy).
2. `docs/04-Tokens-and-Permissions.md` — scopes, roles, and token flows.
3. `worker/go-ingestor/` — configure `.env`, run locally, or build the container image.
4. `deploy/` — schedule the worker (GitHub Actions or CronJob).
5. `docs/07-Dashboard.md` — Port dashboard widgets for AI consumption insights.
6. `docs/08-Differences-vs-Port-Default.md` — how this package extends Port’s defaults.
7. `docs/09-Validation-and-Guardrails.md` — validation, privacy, ops guardrails.
8. `docs/10-References.md` — API references for GitHub, Microsoft Graph, and Port.

## Quick start
1. Apply the JSON in `configs/blueprints/`, then wire up Port data sources with the files in `configs/mappings/`.
2. Copy `worker/go-ingestor/config.example.env` to `.env`, populate secrets (GitHub PAT, Graph app credentials, Port tokens, webhook URLs), toggle `INGEST_GITHUB` / `INGEST_M365` as needed, and run `go build -o ingest ./worker/go-ingestor/...`.
3. Choose a deployment target from `deploy/` and plug in the same environment variables for scheduled runs.
4. Finish with the widgets in `docs/07-Dashboard.md` and the validation checklist in `docs/09-Validation-and-Guardrails.md`.

Everything under `docs/` can be read independently, but the sequence page keeps you in the right order.
