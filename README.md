# MCP Server Documentation

Public documentation for the [Model Context Protocol](https://modelcontextprotocol.io/) servers implemented in the Laravel13 application. Each subdirectory documents one production MCP server: endpoint, authentication, and available tools.

**Published site:** [https://jmtroller.github.io/mcp-documentation/](https://jmtroller.github.io/mcp-documentation/) (after enabling GitHub Pages on this repo)

All servers use **Streamable HTTP** (JSON-RPC 2.0 over HTTPS). A typical session is:

`initialize` → `notifications/initialized` (optional) → `tools/list` → `tools/call`

## Google Connected Apps

Each public host answers `GET` and `HEAD` `/.well-known/google-connected-apps` with `{"verification_tokens":["…"]}` after that host's token is placed in server storage. Until then the route returns `verification_not_configured`.

`initialize` negotiates `2025-11-25`, `2025-06-18`, or `2025-03-26`, and otherwise answers `2025-03-26`. The result includes corpus `instructions` and does not name tools. Every tool in `tools/list` carries `readOnlyHint`, `destructiveHint`, `idempotentHint`, `openWorldHint`, and a `title`.

Nursing Home Database, FindSeniorMed, and SeniorHealthDatabase stay public: no OAuth, and their server cards do not advertise an authorization server. A `GET` of `/mcp` on those hosts can return 405; clients should `POST`. Bankruptcy Observer uses OAuth. An invalid `Authorization: Bearer` token is HTTP 401 with `WWW-Authenticate`. A request with no bearer token can still use the free lookups.

## Servers

| Server | Directory | Endpoint | Auth |
|--------|-----------|----------|------|
| Nursing Home Database | [nursing-home-database](./nursing-home-database/) | `https://mcp.nursinghomedatabase.com/mcp` | Public |
| Bankruptcy Observer | [bankruptcy-observer](./bankruptcy-observer/) | `https://mcp.bankruptcyobserver.com/mcp` | Free tier + subscription |
| TrollerBk | [trollerbk](./trollerbk/) | `https://mcp.trollerbk.com/mcp` | Required |
| FindSeniorMed / FindMed | [findseniormed](./findseniormed/) | `https://mcp.findseniormed.com/mcp` | Public |
| SeniorHealthDatabase | [seniorhealthdatabase](./seniorhealthdatabase/) | `https://mcp.seniorhealthdatabase.com/mcp` | Public |

## Implementation

Routes are defined in `Laravel13/routes/mcp.php`. Each server has a dedicated controller under `Laravel13/app/Http/Controllers/`. OAuth discovery and token endpoints are shared across authenticated servers via `Laravel13/app/Http/Controllers/McpAuth/OAuthController.php`.

## MCP Registry manifests

Each server folder includes a [`server.json`](https://modelcontextprotocol.io/docs/registry) for the official MCP Registry.

For monorepos, the MCP Registry requires:

- `repository.url` — the **git-cloneable repo root** (same for every server in this repo)
- `repository.subfolder` — the **per-server directory** (e.g. `bankruptcy-observer`)
- `websiteUrl` — the **published documentation page** for that server on GitHub Pages

Example:

```json
"websiteUrl": "https://jmtroller.github.io/mcp-documentation/bankruptcy-observer/",
"repository": {
  "url": "https://github.com/jmtroller/mcp-documentation",
  "source": "github",
  "subfolder": "bankruptcy-observer"
}
```

| Server | Registry name | Manifest |
|--------|-----------------|----------|
| Nursing Home Database | `com.nursinghomedatabase/mcp` | [server.json](./nursing-home-database/server.json) |
| Bankruptcy Observer | `com.bankruptcyobserver/mcp` | [server.json](./bankruptcy-observer/server.json) |
| TrollerBk | `com.trollerbk/mcp` | [server.json](./trollerbk/server.json) |
| FindSeniorMed | `com.findseniormed/mcp` | [server.json](./findseniormed/server.json) |
| SeniorHealthDatabase | `com.seniorhealthdatabase/mcp` | [server.json](./seniorhealthdatabase/server.json) |

Older per-server documentation repos (for example `nhd-mcp-public-documentation`) may still exist; this repository is the consolidated source of truth derived from Laravel13.

## GitHub Pages

This repo is configured for GitHub Pages via Jekyll (see `_config.yml` and `.github/workflows/pages.yml`).

1. Create a GitHub repo named `mcp-documentation` under the `jmtroller` account (or update `_config.yml` `url` / `baseurl` for your org).
2. Push this directory to the `main` branch.
3. In repo **Settings → Pages**, set **Source** to **GitHub Actions**.
4. The workflow publishes to `https://<user>.github.io/mcp-documentation/`.

Local preview:

```bash
cd mcp-documentation
bundle install
bundle exec jekyll serve
```