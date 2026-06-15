# Nursing Home Database — MCP

Public documentation for the **Nursing Home Database** [Model Context Protocol](https://modelcontextprotocol.io/) server. It exposes US nursing facility search, detail, ownership, and owner portfolio data to AI assistants and other MCP clients.

**Website:** [nursinghomedatabase.com](https://www.nursinghomedatabase.com)  
**Contact:** [Contact page](https://www.nursinghomedatabase.com/contact)  
**Laravel controller:** `Laravel13/app/Http/Controllers/NursingHome/NhdApiController.php`

## MCP endpoint

| | |
|---|---|
| **Transport** | Streamable HTTP (JSON-RPC 2.0 over HTTPS) |
| **URL** | `https://mcp.nursinghomedatabase.com/mcp` |
| **Auth** | None required (public access) |
| **Server name** | `nhd-mcp` |

Send a standard MCP session: `initialize` → `notifications/initialized` (optional; may receive HTTP 204) → `tools/list` → `tools/call` as needed.

## Tools

### `search_facilities`

Search nursing facilities by name, geography, or distance from a free-form address.

| Parameter | Type | Description |
|-----------|------|-------------|
| `q` | string | Free-text search term |
| `address` | string | Origin address for distance search (e.g. `150 Corporate Woods Drive, Magnolia, TX 77354`) |
| `city` | string | City filter |
| `state` | string | State filter |
| `zip` | string | ZIP code filter |
| `radius_miles` | number | Distance radius; defaults to `25` when `address` is provided |
| `sort` | string | Use `distance` to sort nearest-first when using address-based search |
| `limit` | integer | Max results |
| `offset` | integer | Pagination offset |

If the supplied address cannot be resolved, the API returns an address-specific validation error.

### `get_facility`

Get one facility by CMS provider number (`provnum`) or site slug.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | string | yes | Provider number or web slug |

### `get_facility_ownership`

Get ownership records for one facility.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | string | yes | Provider number or web slug |

### `search_owners`

Search owners by name.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `q` | string | yes | Owner name search term |
| `limit` | integer | no | Max results |
| `offset` | integer | no | Pagination offset |

### `get_owner`

Get an owner portfolio by `web_owner` slug.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `web_owner` | string | yes | Owner slug identifier |

### `get_data_freshness`

Return latest CMS source dataset dates and row counts. No parameters.

## MCP Registry

Listed in the official MCP Registry as **`com.nursinghomedatabase/mcp`**.

- Registry manifest: [`server.json`](./server.json)
- Search: `https://registry.modelcontextprotocol.io/v0.1/servers?search=com.nursinghomedatabase`

## REST API (same backend)

JSON discovery APIs are also available under the main site:

- `GET https://www.nursinghomedatabase.com/api/v1/nh/capabilities`
- `GET https://www.nursinghomedatabase.com/api/v1/nh/facilities?address=...&radius_miles=10&sort=distance`

MCP and REST share the same underlying data layer.

## Example: Cursor / IDE config

```json
{
  "mcpServers": {
    "nursing-home-database": {
      "type": "streamableHttp",
      "url": "https://mcp.nursinghomedatabase.com/mcp"
    }
  }
}
```

## License and data

Data is derived from public CMS and related sources. Use is subject to the terms published on [nursinghomedatabase.com](https://www.nursinghomedatabase.com).