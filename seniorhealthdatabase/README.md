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

Domain aliases also work:

- `https://seniorhealthdatabase.com/mcp`
- `https://www.seniorhealthdatabase.com/mcp`

## Tools

### `search_facilities`

Search healthcare facilities. Filter by type, location, ratings.

| Parameter | Type | Description |
|-----------|------|-------------|
| `q` | string | Search term (name, city, etc.) |
| `type` | string | `dialysis` (default), `home_health`, `hospice`, or `hospital` |
| `state` | string | Two-letter state abbreviation |
| `city` | string | City name |
| `min_rating` | number | Minimum overall/star rating filter |
| `limit` | integer | Max results; default 25, max 50 |

### `get_facility_details`

Get detailed ratings and information for a specific facility.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | string | yes | Facility identifier |
| `type` | string | no | Optional type hint: `dialysis`, `home_health`, `hospice`, or `hospital` |

When `type` is omitted, the server tries dialysis, home health, hospice, and hospital lookups in sequence.

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

## License and data

Data is derived from public CMS and related sources. Use is subject to the terms published on [seniorhealthdatabase.com](https://www.seniorhealthdatabase.com).