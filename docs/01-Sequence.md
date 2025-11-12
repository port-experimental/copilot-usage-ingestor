# 01 — Execution Sequence (Do It In This Order)

> Goal: ingest **GitHub Copilot** (usage + seats) and **Microsoft 365 Copilot** usage into **Port** with a Go worker; ship a dashboard.

## 1) Create Blueprints in Port
Apply the JSON files in `configs/blueprints/` via Port UI (Builder → Edit JSON) or API.

Order:
1. `github_copilot_seats.json`
2. `github_copilot_usage.json` (extends Port defaults)
3. `m365_copilot_usage_summary.json`
4. `m365_copilot_user.json`

## 2) Configure Port Data Sources & Mappings
A. **GitHub Copilot metrics** — keep Port’s **existing GitHub Copilot integration**.  
   - Add the mapping override from `configs/mappings/github_copilot_mapping_override.yaml` to compute `editor_top` and `language_top` and to set chat fields.

B. **Webhook (custom) for GitHub seats snapshot** — create a Webhook data source in Port; paste `configs/mappings/webhook_github_seats.json`.  
   - Copy the ingestion URL and the secret you configure; store them as env vars.

C. **Webhook (custom) for M365 Copilot** — create two Webhook data sources; paste:  
   - `configs/mappings/webhook_m365_summary.json`  
   - `configs/mappings/webhook_m365_users.json`  
   - Copy ingestion URLs and secret; store as env vars.

> Prefer Webhooks for simplicity. You can also upsert directly via Port’s Entities API if you don’t want Webhooks.

## 3) Obtain Tokens & Secrets
Follow `04-Tokens-and-Permissions.md` carefully to generate:  
- **Port** access token (or client credentials).  
- **GitHub** PAT (classic) with `manage_billing:copilot`, `read:org` or `read:enterprise` as needed.  
- **Microsoft Graph** app credentials and admin consent for `Reports.Read.All` (+ license read).

## 4) Configure the Worker
Make a copy of `worker/go-ingestor/config.example.env` → `.env` and fill all values.  
Toggle `INGEST_GITHUB` / `INGEST_M365` to control which sources run (defaults: both true).  
If you use Webhooks, set `USE_PORT_WEBHOOK=true` and supply the URLs + secret for the enabled sources only.

## 5) Run Locally (first backfill)
Build and run:
```bash
cd worker/go-ingestor
go build -o ingest ./...
./ingest
```
Expect one seats snapshot entity, one M365 summary entity per run, and many M365 user entities.

## 6) Deploy
Pick one:
- **Kubernetes CronJob** → `deploy/k8s-cronjob.yaml` (02:00 UTC daily).  
- **GitHub Actions** → `deploy/github-actions.yaml` (03:30 UTC daily).

## 7) Build the Dashboard
Use `docs/07-Dashboard.md` to add KPI, line, pie, and table widgets. Attach scorecards if desired.

## 8) Validate and Lock Down
Follow `docs/09-Validation-and-Guardrails.md` to check data parity, rate limits, privacy, and rotate credentials.
