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
| **Version** | `1.0.0` |

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
| `q` | string | Last name, or first and last |
| `specialty` | string | Case-insensitive specialty or alias, such as `cardiology` or `GI` |
| `state` | string | Two-letter state abbreviation |
| `city` | string | City name. Not a device location |
| `zip` | string | ZIP code. Not a device location |
| `type` | string | `provider` (default) or `provider-group` |
| `filedate` | string | `YYYY-MM-DD` snapshot. Defaults to the latest loaded file |
| `limit` | integer | Max results; default 25, max 50 |
| `offset` | integer | Skip this many unique NPIs or groups |

When `q` is a single word it is treated as a last name; multi-word values are split into first and last name for doctor search.

### `get_provider_details`

Get detailed information for a specific provider or provider group.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | string | yes | NPI |
| `filedate` | string | no | `YYYY-MM-DD` snapshot |

An unpublished MIPS score is null.

### `compare_providers`

Compare up to 10 clinicians by NPI on one snapshot. `ids` is required (an array or a comma-separated string). `filedate` is optional.

### `list_file_dates`

Clinician snapshot dates already loaded, including latest and prior. No parameters.

### `list_specialties`

Medicare specialty catalog with `dataKey` and slug. Search accepts aliases such as cardiology, heart, GI, eye, skin, kidney, lung, and bones. No parameters.

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

## Google Connected Apps

There is no OAuth. `initialize` returns corpus instructions and a favicon. Every tool includes hint annotations. `GET /mcp` may return 405; use `POST`. Domain verification is `/.well-known/google-connected-apps`. Privacy: [findseniormed.com/privacy](https://www.findseniormed.com/privacy). Terms: [findseniormed.com/terms](https://www.findseniormed.com/terms).

## License and data

Data is derived from public CMS and related sources (NPI, Medicare provider data, etc.). Use is subject to the terms published on [findseniormed.com](https://www.findseniormed.com).