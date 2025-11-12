# Runbook — ship the ingestion stack

Goal: ingest **GitHub Copilot** (usage + seats) and **Microsoft 365 Copilot** usage into **Port** with a Go worker; ship a dashboard and guardrails.

## Apply the Port blueprints
Upload the JSON files in `configs/blueprints/` via Port Builder (Edit JSON) or the API:
- `github_copilot_seats.json`
- `github_copilot_usage.json` (extends Port defaults)
- `m365_copilot_usage_summary.json`
- `m365_copilot_user.json`

## Wire up data sources & mappings
- **GitHub Copilot metrics** — keep Port’s built-in integration and add `configs/mappings/github_copilot_mapping_override.yaml` to enrich editor/language/chat fields.
- **GitHub seats snapshot** — create a Webhook data source, paste `configs/mappings/webhook_github_seats.json`, then save the ingestion URL + secret as env vars.
- **M365 Copilot** — create two Webhook data sources using `configs/mappings/webhook_m365_summary.json` and `configs/mappings/webhook_m365_users.json` and store their URLs + secret.

### Create the webhook data sources
**Builder UI (no code):**
1. Sign in to Port Builder and open your workspace.
2. Go to **Data sources** → **New data source** → select **Webhook**.
3. In the *General* tab:
   - Set **Identifier** to match the JSON (e.g., `gh-copilot-seats-webhook`, `m365-copilot-summary-webhook`, `m365-copilot-users-webhook`).
   - Give it a descriptive **Title** (the defaults from the JSON are fine).
4. Switch to the **Mappings** tab → click **Edit JSON** → paste the contents of the corresponding file from `configs/mappings/`. Save the JSON.
5. In the **Security** section, replace the placeholder `security.secret` value with a randomly generated string (record it—you’ll need it in `.env`). Keep the default header (`X-Signature`) and algorithm (`sha256`).
6. Click **Create data source**. Port will display the webhook ingestion URL in the right-hand panel; copy it along with the secret you set.
7. Repeat for the remaining two mapping files. When finished you should have three webhook URLs + secrets ready for `PORT_WEBHOOK_SEATS_URL`, `PORT_WEBHOOK_M365_SUMMARY_URL`, and `PORT_WEBHOOK_M365_USERS_URL` (and a shared `PORT_WEBHOOK_SECRET` if you used the same value).

**Port API (documented at `docs.port.io/api/#tag/Data-Sources/operation/createDataSource`):**
```bash
PORT_API_BASE=https://api.getport.io        # or https://api.us.getport.io
PORT_TOKEN=...                              # access token or client-credentials token

curl -X POST "$PORT_API_BASE/v1/data-sources" \
  -H "Authorization: Bearer $PORT_TOKEN" \
  -H "Content-Type: application/json" \
  --data-binary @configs/mappings/webhook_github_seats.json
```
Each JSON file already matches the webhook data source schema exposed by the API; edit the `security.secret` value (and any titles) before posting. If you prefer to inline the JSON manually, mirror the structure shown in the Swagger reference.
Repeat for the M365 summary and user files. The response payload (and `GET /v1/data-sources/{identifier}`) contains the webhook URL and security metadata—store them in your secret manager and reference them in `PORT_WEBHOOK_*` variables.

> Prefer Webhooks for simplicity. You can also upsert directly through Port’s Entities API if needed.

## Gather tokens and secrets
Follow `docs/tokens-and-permissions.md` to mint:
- **Port** access token (or client credentials).
- **GitHub** PAT (classic) with `manage_billing:copilot` plus `read:org` / `read:enterprise`.
- **Microsoft Graph** app credentials with `Reports.Read.All` and license-read permissions, plus admin consent.

## Configure the worker
- Copy `worker/go-ingestor/config.example.env` to `.env` and fill every variable.
- Toggle `INGEST_GITHUB` / `INGEST_M365` if you want to run one source at a time.
- When using webhooks, set `USE_PORT_WEBHOOK=true` and provide the seats + M365 ingestion URLs with a shared secret.

## Run locally
```bash
cd worker/go-ingestor
go build -o ingest ./...
./ingest
```
Expected outcome: a GitHub seats snapshot, one M365 summary entity, and many M365 user entities per execution.

## Deploy on a schedule
Pick whichever scheduler fits best:
- Kubernetes CronJob → `deploy/k8s-cronjob.yaml` (02:00 UTC daily).
- GitHub Actions → `deploy/github-actions.yaml` (03:30 UTC daily).

## Build the dashboard
`docs/dashboard.md` walks through KPI, trend, breakdown, and table widgets so Port surfaces the new data immediately.

## Validate and lock down
`docs/validation-and-guardrails.md` covers parity checks, rate limits, privacy, and secret rotation. Run through it after deployment and whenever the data model changes.
