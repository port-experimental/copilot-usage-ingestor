# Dashboard (AI consumption)

Create a **Dashboard** page in Port and add these widgets. Filter by last 30 days where applicable.

## KPI Row
- **GitHub Acceptance Rate (avg 30d)** — blueprint: `github_copilot_usage`, property: `acceptance_rate` (avg).
- **M365 License Utilization %** — blueprint: `m365_copilot_usage_summary`, property: `license_utilization_rate` (latest).
- **Seats Active (14d)** — blueprint: `github_copilot_seats`, property: `seats_active_14d` (latest).
- **Chat Adoption %** — blueprint: `github_copilot_usage`, property: `chat_adoption_rate` (avg).

## Scorecards (AP usage snapshots)
Create two scorecards per product so leaders can glance at “AP usage” (how many provisioned seats are producing value) and depth of usage.

### GitHub Copilot scorecards
- **Active Seats % (14d)**  
  - Blueprint: `github_copilot_seats`.  
  - Formula: `seats_active_14d / seats_total`.  
  - Thresholds: green ≥ 0.7, yellow 0.4–0.69, red < 0.4.
- **Suggestions Per Active Dev**  
  - Blueprint: `github_copilot_usage`.  
  - Formula: `total_suggestions / total_active_users`.  
  - Use this to highlight teams with low prompt usage even if they’re technically active.

### Microsoft 365 Copilot scorecards
- **License Utilization %**  
  - Blueprint: `m365_copilot_usage_summary`.  
  - Formula: `active_user_count / sku_total`.  
  - Thresholds: green ≥ 0.6, yellow 0.35–0.59, red < 0.35.
- **Weekly AP Depth**  
  - Blueprint: `m365_copilot_user`.  
  - Metric: average `days_since_last_activity` inverted (e.g., `1 / (days_since_last_activity + 1)`), or simpler, percentage of users with activity ≤ 7 days.  
  - Helps identify when users are enabled but rarely invoking Copilot experiences.

## Trends (Line)
- **Active Users (GitHub)** — x:`record_date`, y:`total_active_users`.
- **Chat Turns (GitHub)** — x:`record_date`, y:`total_chat_turns`.
- **Active Users (M365)** — x:`report_date`, y:`active_user_count` per period entity.

## Breakdowns
- **Top Editors (GitHub)** — pie by `editor_top`.
- **Top Languages (GitHub)** — pie by `language_top`.
- **App Mix (M365)** — create stacked bars from user detail using `*_last_activity` recency (optional).

## Tables
- **Dormant M365 Users (≥30d)** — from `m365_copilot_user`, compute `DATEDIFF(now, last_activity_date)` and sort.
- **Top Teams by Acceptance Rate** — if you ingest team-level usage, sort `acceptance_rate` with a minimum suggestions threshold.
