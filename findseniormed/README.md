# FindSeniorMed / FindMed — MCP

Public documentation for the **FindMed** (FindSeniorMed) [Model Context Protocol](https://modelcontextprotocol.io/) server. It exposes search and details for doctors, specialists, clinicians, and provider groups focused on senior care / Medicare providers.

**Website:** [findseniormed.com](https://www.findseniormed.com)  
**Contact:** [Contact page](https://www.findseniormed.com/contact)  
**Laravel controller:** `Laravel13/app/Http/Controllers/FindMed/FindMedMcpController.php`

## MCP endpoint

| | |
|---|---|
| **Transport** | Streamable HTTP (JSON-RPC 2.0 over HTTPS) |
| **URL** | `https://mcp.findseniormed.com/mcp` (recommended) |
| **Auth** | None required (public access) |
| **Server name** | `findmed-mcp` |

Domain aliases also work:

- `https://findseniormed.com/mcp`
- `https://www.findseniormed.com/mcp`
- `https://mcp.findmed.com/mcp`
- `https://findmed.com/mcp` / `https://www.findmed.com/mcp`

## Tools

### `search_providers`

Search for doctors, specialists, or provider groups for seniors. Supports specialty, location, and name filters.

| Parameter | Type | Description |
|-----------|------|-------------|
| `q` | string | Search term (name, specialty, etc.) |
| `specialty` | string | Specialty name filter |
| `state` | string | Two-letter state abbreviation |
| `city` | string | City name |
| `type` | string | `provider` (default, individual clinicians) or `provider-group` |
| `limit` | integer | Max results; default 25, max 50 |

When `q` is a single word it is treated as a last name; multi-word values are split into first and last name for doctor search.

### `get_provider_details`

Get detailed information for a specific provider or provider group.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | string | yes | Provider or group identifier (NPI, org ID, etc.) |

### `get_data_freshness`

Return CMS source dataset release dates and row counts for doctors, clinicians, and affiliations. No parameters.

## MCP Registry

Listed (or published) in the official MCP Registry as **`com.findseniormed/mcp`**.

- Registry manifest: [`server.json`](./server.json)
- Search: `https://registry.modelcontextprotocol.io/v0.1/servers?search=com.findseniormed`

## Example: Cursor / IDE config

```json
{
  "mcpServers": {
    "findseniormed": {
      "type": "streamableHttp",
      "url": "https://mcp.findseniormed.com/mcp"
    }
  }
}
```

## License and data

Data is derived from public CMS and related sources (NPI, Medicare provider data, etc.). Use is subject to the terms published on [findseniormed.com](https://www.findseniormed.com).