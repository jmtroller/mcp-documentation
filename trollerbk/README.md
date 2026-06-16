# TrollerBk — MCP

Public documentation for the **TrollerBk** [Model Context Protocol](https://modelcontextprotocol.io/) server. It exposes market-wide US business bankruptcy intelligence (filings, case screening, EIN lookups, docket access, court documents) plus personalized custom subscriber reports.

Unlike PACER (which requires knowing a specific case), TrollerBk answers cross-case and market questions.

**Website:** [trollerbk.com](https://www.trollerbk.com)  
**MCP docs:** [trollerbk.com/mcp](https://www.trollerbk.com/mcp)  
**Contact:** support@trollerbk.com  
**Laravel controller:** `Laravel13/app/Http/Controllers/TRBK/TrbkMcpController.php`

## MCP endpoint

| | |
|---|---|
| **Transport** | Streamable HTTP (JSON-RPC 2.0 over HTTPS) |
| **URL** | `https://mcp.trollerbk.com/mcp` |
| **Auth** | **Required** for all calls (including `initialize` and `tools/list`) |
| **Server name** | `trbk-mcp` |

Send a standard MCP session: `initialize` → `notifications/initialized` (optional) → `tools/list` → `tools/call`.

## Authentication

All requests require a valid TrollerBk credential:

- `Authorization: Bearer <your-api-token>`
- `X-API-Key: <your-api-token>` (optionally with `X-Client-Email`)
- OAuth Bearer token via the shared MCP OAuth flow

API tokens are obtained via subscription. Requests without valid credentials return a JSON-RPC auth error (HTTP 401).

OAuth discovery endpoints are available at the MCP host root:

- `GET https://mcp.trollerbk.com/.well-known/oauth-authorization-server`
- `GET https://mcp.trollerbk.com/.well-known/oauth-protected-resource`

## Core tools

These tools are always available to authenticated users.

### `get_new_filings`

Recent US business bankruptcy filings.

| Parameter | Type | Description |
|-----------|------|-------------|
| `days` | integer | Trailing window in days (1–90); default 7 |
| `chapter` | integer | Bankruptcy chapter (7, 11, etc.) |
| `state` | string | Two-letter US state |
| `industry` | string | Industry / nature of business keyword |
| `limit` | integer | Max results (1–100); default 25 |

### `search_cases`

Screen bankruptcy cases by multiple criteria.

| Parameter | Type | Description |
|-----------|------|-------------|
| `name` | string | Debtor name keyword |
| `chapter` | integer | Bankruptcy chapter |
| `state` | string | Two-letter US state |
| `industry` | string | Industry keyword |
| `status` | string | `open` or `closed`; default `open` |
| `date_from` | string | Filed on or after (YYYY-MM-DD) |
| `date_to` | string | Filed on or before (YYYY-MM-DD) |
| `limit` | integer | Max results (1–100); default 25 |

### `get_case_details`

Full case record for a single bankruptcy by TrollerBk `simple_name` identifier.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `simple_name` | string | yes | TrollerBk simpleName (e.g. `acme-corp-24-00123`) |

Returns `caseId` and core fields; `simpleName`, `fullCaseNumber`, and lead flag are not included in the response.

### `search_by_ein`

All bankruptcy cases for a debtor EIN.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `ein` | string | yes | Nine-digit EIN (with or without hyphen) |

### `get_case_by_number`

Resolve a court case number to basic case records.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `case_number` | string | yes | Short court case number (e.g. `24-10001`) |

### `get_docket`

Docket entries for a case. Returns `docketId`, `caseId`, `itemText`, `itemDate`, `pageCount`. Defaults to the first 10 items ordered by `itemNumber` ascending.

| Parameter | Type | Description |
|-----------|------|-------------|
| `case_id` | string | Internal TrollerBk caseId |
| `simple_name` | string | TrollerBk simpleName |
| `case_number` | string | Short court case number |
| `date_from` | string | Filter items on or after (YYYY-MM-DD) |
| `date_to` | string | Filter items on or before (YYYY-MM-DD) |
| `item_from` | integer | Filter items with itemNumber ≥ value |
| `item_to` | integer | Filter items with itemNumber ≤ value |
| `limit` | integer | Max items (default 10, max 100) |

Use returned `docketId` values with document tools below.

### `get_document_cost`

Preview cost to obtain a full court document PDF. Does not pull or charge. Call this before `request_document`.

| Parameter | Type | Description |
|-----------|------|-------------|
| `docket_id` | string | docketId or claims register document ID |
| `pacer_rss_id` | string | pacerRSSItemId or rssId from custom report rows |
| `docket_ids` | array | Multiple docket IDs |
| `pacer_rss_ids` | array | Multiple pacer RSS IDs |

### `request_document`

Request (pull if needed + charge if applicable) the full PDF document. Prefer calling `get_document_cost` first.

| Parameter | Type | Description |
|-----------|------|-------------|
| `docket_id` | string | docketId or claims register document ID |
| `pacer_rss_id` | string | pacerRSSItemId or rssId from custom report rows |
| `docket_ids` | array | Multiple docket IDs |
| `pacer_rss_ids` | array | Multiple pacer RSS IDs |
| `zip` | boolean | Deliver all documents as one zip with manifest |

## Custom report tools (`custom_*`)

When authenticated with a subscriber API token linked to `VwClientAuth`, the server dynamically registers personalized custom reports from the TrollerBk `CustomReports` register. Tool names follow the pattern `custom_<report_slug>`.

Examples that may appear depending on your subscription:

- `custom_admin_exp`
- `custom_sale_motion`
- `custom_schedules`
- `custom_adversary_proceedings`



Exact custom tools and schemas are returned live from `tools/list` for your token.

## MCP Registry

Listed in the official MCP Registry as **`com.trollerbk/mcp`**.

- Registry manifest: [`server.json`](./server.json)
- Search: `https://registry.modelcontextprotocol.io/v0.1/servers?search=com.trollerbk`

## Example: Cursor / IDE config

```json
{
  "mcpServers": {
    "trollerbk": {
      "type": "streamableHttp",
      "url": "https://mcp.trollerbk.com/mcp",
      "headers": {
        "Authorization": "Bearer YOUR_TROLLERBK_API_TOKEN"
      }
    }
  }
}
```

## License and data

Bankruptcy data is aggregated from public court records (PACER-derived). Access and use are subject to TrollerBk terms of service and subscriber agreements.
