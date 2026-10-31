# SeniorHealthDatabase — MCP

Public documentation for the **SeniorHealthDatabase** [Model Context Protocol](https://modelcontextprotocol.io/) server. It exposes search and details for senior healthcare facilities: dialysis centers, home health agencies, hospice providers, and hospitals.

**Website:** [seniorhealthdatabase.com](https://www.seniorhealthdatabase.com)  
**Contact:** [Contact page](https://www.seniorhealthdatabase.com/contact)  
**Laravel controller:** `Laravel13/app/Http/Controllers/SeniorHealth/SeniorHealthMcpController.php`

## MCP endpoint

| | |
|---|---|
| **Transport** | Streamable HTTP (JSON-RPC 2.0 over HTTPS) |
| **URL** | `https://mcp.seniorhealthdatabase.com/mcp` (recommended) |
| **Auth** | None required (public access) |
| **Server name** | `seniorhealth-mcp` |
| **Version** | `1.1.0` |

Domain aliases also work:

- `https://seniorhealthdatabase.com/mcp`
- `https://www.seniorhealthdatabase.com/mcp`

## Tools

### `search_facilities`

Search healthcare facilities. Filter by type, location, ratings.

| Parameter | Type | Description |
|-----------|------|-------------|
| `q` | string | Search term (name, city, etc.) |
| `type` | string | Required. `hospital`, `home_health`, `hospice`, or `dialysis` |
| `state` | string | Two-letter state abbreviation |
| `city` | string | City name |
| `min_rating` | number | Minimum overall/star rating filter |
| `limit` | integer | Max results; default 25, max 50 |

### `get_facility_details`

Get detailed ratings and information for a specific facility.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | string | yes | Facility identifier |
| `type` | string | yes | `hospital`, `home_health`, `hospice`, or `dialysis` |
| `filedate` | string | no | `YYYY-MM-DD` snapshot. Defaults to the latest loaded file |

### `compare_facilities`

Compare 2–10 facilities of one type. Returns the detail rows plus mechanical high and low extremes on key metrics. That summary has no overall ranking. `ids` and `type` are required. `filedate` is optional.

### `get_facility_changes`

Diff `overall_rating`, ratings, and measures for one facility between two snapshots. `id` and `type` are required. `from` and `to` default to the prior snapshot and the latest loaded file.

### `list_file_dates`

Snapshot dates loaded for one facility type, including latest and prior. `type` is optional.

### `get_data_freshness`

Return CMS source dataset release dates and row counts for dialysis, home health, hospice, and hospital data. No parameters.

## MCP Registry

Listed (or published) in the official MCP Registry as **`com.seniorhealthdatabase/mcp`**.

- Registry manifest: [`server.json`](./server.json)
- Search: `https://registry.modelcontextprotocol.io/v0.1/servers?search=com.seniorhealthdatabase`

## Example: Cursor / IDE config

```json
{
  "mcpServers": {
    "seniorhealthdatabase": {
      "type": "streamableHttp",
      "url": "https://mcp.seniorhealthdatabase.com/mcp"
    }
  }
}
```

## Google Connected Apps

There is no OAuth. `initialize` returns corpus instructions (server version `1.1.0`) and a favicon. Every tool includes hint annotations. An address plus `radius_miles` is geocoded with Mapbox and returns straight-line miles. A ZIP plus radius uses stored coordinates and does not leave the server. `GET /mcp` may return 405; use `POST`. Domain verification is `/.well-known/google-connected-apps`. Privacy: [seniorhealthdatabase.com/privacy](https://www.seniorhealthdatabase.com/privacy). Terms: [seniorhealthdatabase.com/terms](https://www.seniorhealthdatabase.com/terms).

## License and data

Data is derived from public CMS and related sources. Use is subject to the terms published on [seniorhealthdatabase.com](https://www.seniorhealthdatabase.com).