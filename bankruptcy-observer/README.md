# Bankruptcy Observer — MCP

Public documentation for the **Bankruptcy Observer** [Model Context Protocol](https://modelcontextprotocol.io/) server. It provides agent-first access to US business bankruptcy filings, dockets, court documents, case summaries, and monitoring workflows.

**Website:** [bankruptcyobserver.com](https://www.bankruptcyobserver.com)  
**MCP docs:** [mcp.bankruptcyobserver.com/docs](https://mcp.bankruptcyobserver.com/docs)  
**Subscribe:** [mcp.bankruptcyobserver.com/subscribe](https://mcp.bankruptcyobserver.com/subscribe)  
**Laravel controller:** `Laravel13/app/Http/Controllers/BKO/BkoMcpController.php`

## MCP endpoint

| | |
|---|---|
| **Transport** | Streamable HTTP (JSON-RPC 2.0 over HTTPS) |
| **URL** | `https://mcp.bankruptcyobserver.com/mcp` |
| **Auth** | Optional — free tier works without credentials; paid tools require a subscriber MCP token |
| **Server name** | `bko-mcp` |

Send a standard MCP session: `initialize` → `notifications/initialized` (optional) → `tools/list` → `tools/call`.

## Authentication

The BKO middleware never returns HTTP 401. Instead:

- **Free tier (no auth):** Limited lookups for exact debtor name (no `*` wildcard) and 7-digit case numbers.
- **Subscriber (auth required for full data):** Pass your MCP API token via:
  - `Authorization: Bearer <api_token>`
  - `X-API-Key: <api_token>` (optionally with `X-Client-Email`)
  - OAuth Bearer token from the shared MCP OAuth flow (`/oauth/token`)

Website subscribers get MCP access included. Tokens are visible in the subscriber dashboard at `/subscriber/mcp-setup`. New users can also call `list_plans_tool` and `purchase_plan_tool` without auth to start Stripe checkout.

## Subscription tiers

| Tier | Access |
|------|--------|
| **Anonymous** | Free-shape lookups only (exact name, 7-digit case number) |
| **Single-Case Agent** | Case tools scoped to monitored cases on your BKO account |
| **Full Access Agent** | All search tools plus full case/docket/document access |

## Tools

### Free / lookup tools

#### `search_bankruptcy_cases_tool`

Search US business bankruptcy cases by debtor name.

| Parameter | Type | Description |
|-----------|------|-------------|
| `search_term` | string | Debtor name (prefix or `*contains` for paid wildcard search) |
| `limit` | integer | Max results (1–100); default 25 |
| `skip` | integer | Pagination offset |
| `court_id` | string | Optional court filter |
| `court_state` | string | Two-letter state to filter court name |

**Free:** exact/prefix match without `*`. **Paid:** `*term` contains search and full fields.

#### `get_case_by_case_number_tool`

Look up case(s) by short case number (e.g. `26-10543`).

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `short_case_number` | string | yes | Short case number |
| `court_id` | string | no | Court filter |
| `court_state` | string | no | Two-letter state |
| `live_update` | boolean | no | One-time refresh from court sources (subscribers) |

**Free:** 7-digit format returns limited fields across all courts. **Paid:** full data, `live_update`, court filters.

### Paid search tools (Full Access Agent)

#### `get_case_by_ein_tool`

Look up cases by debtor EIN.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `debtor_ein` | string | yes | Employer Identification Number |
| `include_last_filing` | boolean | no | Include `lastFilingDate` from docket |
| `live_update` | boolean | no | One-time source refresh (subscribers) |

#### `get_cases_by_industry_tool`

Cases by industry label or key (Hotels, Retail, real-estate, etc.).

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `industry` | string | yes | Industry label or key |
| `limit` | integer | no | Max results |
| `start_date` | string | no | YYYY-MM-DD |
| `end_date` | string | no | YYYY-MM-DD |

#### `get_cases_by_naics_tool`

Cases by NAICS code (2–4 digits).

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `naics` | string | yes | NAICS code |
| `limit` | integer | no | Max results |
| `start_date` | string | no | YYYY-MM-DD |
| `end_date` | string | no | YYYY-MM-DD |

#### `get_cases_by_state_tool`

Cases by debtor state.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `state` | string | yes | Two-letter state |
| `limit` | integer | no | Max results |
| `start_date` | string | no | YYYY-MM-DD |
| `end_date` | string | no | YYYY-MM-DD |

#### `get_cases_by_date_range_tool`

Cases filed in a date range on `dateFiled`. Omitting both dates defaults to yesterday.

| Parameter | Type | Description |
|-----------|------|-------------|
| `start_date` | string | YYYY-MM-DD |
| `end_date` | string | YYYY-MM-DD |
| `limit` | integer | Max results |

### Case intelligence tools (subscribers)

#### `get_docket_entries_tool`

Docket entries for a case. Each entry includes `docket_id` for document tools.

| Parameter | Type | Description |
|-----------|------|-------------|
| `simple_name` | string | Case simpleName (preferred) |
| `case_number` | string | Short or full case number |
| `short_case_number` | string | Alias for `case_number` |
| `court_id` | string | Optional court filter |
| `court_state` | string | Two-letter state to disambiguate |
| `limit` | integer | Max entries (default 25, max 50) |
| `live_update` | boolean | Refresh from court sources first |

#### `get_case_summary_tool`

Plain-English structured summary: debtor, chapter, court, key dates, asset/liability ranges, nature of business, judge, and AI-extracted profile data when available.

| Parameter | Type | Description |
|-----------|------|-------------|
| `simple_name` | string | Case simpleName (preferred) |
| `case_number` | string | 7-digit short case number |
| `court_state` | string | Two-letter state to disambiguate |

#### `list_monitored_cases_tool`

List cases on your BankruptcyObserver.com monitoring list. No parameters.

#### `add_monitored_case_tool`

Add a case to your monitoring list (hourly court checks, daily docket updates).

| Parameter | Type | Description |
|-----------|------|-------------|
| `simple_name` | string | Case simpleName |
| `case_number` | string | 7-digit short case number alternative |
| `court_state` | string | Two-letter state to disambiguate |
| `court_id` | string | Optional court filter |

#### `refresh_docket_tool`

Request an immediate docket refresh from court sources.

| Parameter | Type | Description |
|-----------|------|-------------|
| `simple_name` | string | Case simpleName |
| `case_number` | string | 7-digit short case number |
| `court_state` | string | Two-letter state |
| `court_id` | string | Optional court filter |

#### `get_recent_developments_tool`

Docket entries since a date (default: last 7 days) for one monitored case or all monitored cases.

| Parameter | Type | Description |
|-----------|------|-------------|
| `simple_name` | string | Optional; omit for all monitored cases |
| `since_date` | string | YYYY-MM-DD; default 7 days ago |
| `limit` | integer | Max entries (default 100, max 250) |

### Document tools (subscribers)

Recommended flow:

1. `get_docket_entries_tool` → obtain `docket_id`
2. `get_document_tool` with `docket_id` only → cost preview
3. User confirms → `get_document_tool` with `accept_charge: true` → signed PDF URL

#### `get_document_cost_tool`

Preview cost to download a court filing PDF. Does not pull or charge.

#### `get_document_tool`

Court filing PDF. Omit `accept_charge`/`download` for preview; set `accept_charge: true` or `download: true` after user confirms.

Shared parameters for document tools:

| Parameter | Type | Description |
|-----------|------|-------------|
| `docket_id` | string | Preferred: 40-character hex hash from docket tools |
| `simple_name` | string | Legacy: case simpleName (use with `item_number`) |
| `item_number` | integer | Legacy: docket item number |
| `accept_charge` | boolean | `true` after user approves cost |
| `download` | boolean | Legacy alias for `accept_charge: true` |

> Note: `get_document_cost` is a dispatch alias of `get_document_cost_tool` but is filtered out of `tools/list` to avoid duplicate catalog entries.

### Meta / account tools

#### `list_plans_tool`

List available BKO subscription plans with pricing and agent tiers. **No auth required.**

#### `purchase_plan_tool`

Get a Stripe checkout URL for a plan. **No auth required.**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `plan_id` | string | yes | Plan ID or Stripe price ID from `list_plans_tool` |
| `email` | string | no | Email to prefill on checkout |

#### `check_subscription_tool`

Check MCP subscription status and remaining quota.

| Parameter | Type | Description |
|-----------|------|-------------|
| `api_key` | string | Optional; uses request credentials if omitted |

## MCP Registry

Listed (or publishable) in the official MCP Registry as **`com.bankruptcyobserver/mcp`**.

- Registry manifest: [`server.json`](./server.json)
- Search: `https://registry.modelcontextprotocol.io/v0.1/servers?search=com.bankruptcyobserver`

## Example: Cursor / IDE config

Free tier (no headers):

```json
{
  "mcpServers": {
    "bankruptcy-observer": {
      "type": "streamableHttp",
      "url": "https://mcp.bankruptcyobserver.com/mcp"
    }
  }
}
```

Subscriber:

```json
{
  "mcpServers": {
    "bankruptcy-observer": {
      "type": "streamableHttp",
      "url": "https://mcp.bankruptcyobserver.com/mcp",
      "headers": {
        "Authorization": "Bearer YOUR_BKO_MCP_TOKEN"
      }
    }
  }
}
```

## License and data

Bankruptcy data is aggregated from public court records. Access and use are subject to Bankruptcy Observer terms of service and subscriber agreements.