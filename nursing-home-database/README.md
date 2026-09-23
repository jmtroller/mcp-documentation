# Nursing Home Database — MCP

Public documentation for the **Nursing Home Database** [Model Context Protocol](https://modelcontextprotocol.io/) server. It exposes US skilled nursing facility search, detail, ownership, and owner portfolio data — including CMS nurse hours, turnover, and inspection scores — to AI assistants and other MCP clients.

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
| **Version** | `1.4.0` |

Send a standard MCP session: `initialize` → `notifications/initialized` (optional; may receive HTTP 204) → `tools/list` → `tools/call` as needed.

## Tools

| Tool | Purpose |
|------|---------|
| `search_facilities` | Finds certified SNFs near an address (or by name/city/state/ownership type) so you can shortlist more than three homes using hours, turnover, and inspection scores |
| `search_facilities_by_ownership` | Same search, with CMS `ownership` type required (For profit / Non profit / Government, or an exact value from `list_distinct_values`) |
| `summarize_facilities` | National or grouped stats (n, mean, min, max; optional median) for nurse hours, turnover, ratings, beds, fines. Prefer this over paging `search_facilities` |
| `get_facility` | Opens one home by CMS provider number or site slug, with the same hours, turnover, inspection, beds, and penalty fields as search |
| `compare_facilities` | Same schema for up to 25 CCNs/slugs — prefer this over looping `get_facility` (MCP is 60 req/min) |
| `get_facility_changes` | Field-level diff of one home across two CMS monthly files. Defaults to the latest file versus the prior snapshot. Not owner names |
| `get_facility_ownership` | Lists who owns a home and in what role |
| `search_owners` | Finds nursing-home owners by name |
| `get_owner` | Opens an owner’s portfolio of certified SNFs |
| `list_file_dates` | Lists CMS monthly snapshot dates in the database so you can look at a prior month |
| `list_distinct_values` | Lists distinct CMS values and counts for a ProviderInfo field (`ownership`, `state`, `sffstatus`, `chainname`, …) |
| `get_data_freshness` | Reports when the latest CMS file was published and how many rows it contains |

Exact `inputSchema` objects are returned in `tools/list`.

### Historical months (`filedate`)

By default every lookup uses the **latest** CMS monthly file. To retrieve a prior month:

1. Call `list_file_dates` (returns `current`, `filedates` newest first, and `known_gaps` for unpublished months).
2. Pass one of those dates as `filedate` (`YYYY-MM-DD` or `YYYY-MM`) on `search_facilities`, `search_facilities_by_ownership`, `summarize_facilities`, `compare_facilities`, `get_facility`, `get_facility_ownership`, `search_owners`, `get_owner`, or `list_distinct_values`. `get_facility_changes` does not take `filedate`; pass `from` and `to`.

CMS snapshots are stored as the first of the month (`YYYY-MM-01`). A value like `2026-07` or `2026-07-15` is normalized to `2026-07-01`. Unknown dates return a validation error — call `list_file_dates` rather than guessing.

### `search_facilities`

Finds certified SNFs near an address so you can shortlist more than three homes using hours, turnover, and inspection scores. You can also search by name, city, state, or ZIP.

| Parameter | Type | Description |
|-----------|------|-------------|
| `q` | string | Free-text facility name |
| `address` | string | Origin address for distance search (e.g. `150 Corporate Woods Drive, Magnolia, TX 77354`) |
| `city` | string | City filter |
| `state` | string | State filter |
| `zip` | string | ZIP code filter |
| `radius_miles` | number | Straight-line radius in miles; defaults to `25` when `address` is provided |
| `min_overall_rating` | integer | Minimum CMS overall star rating (already on REST; also on this MCP tool) |
| `max_overall_rating` | integer | Maximum CMS overall star rating (already on REST; also on this MCP tool) |
| `abuse_icon` | string | Filter by CMS abuse icon (`Y` / `N`, `true` / `false`) (already on REST; also on this MCP tool) |
| `sffstatus` | string | When set, keep homes that have a CMS Special Focus Facility (or candidate) status (already on REST; also on this MCP tool) |
| `ownership` | string | CMS ownership type (`ProviderInfo.ownership`). Case-insensitive. Prefixes `For profit`, `Non profit`, and `Government` match that kind. Call `list_distinct_values` with `field=ownership` for the 13 exact values |
| `sort` | string | Sort key (see [Sort keys](#sort-keys) below) |
| `limit` | integer | Max results |
| `offset` | integer | Pagination offset |
| `filedate` | string | Optional CMS monthly snapshot from `list_file_dates` (defaults to the latest month) |

If the supplied address cannot be resolved, the API returns an address-specific validation error.

### Facility payload (search and detail)

`search_facilities` (each `items[]` row) and `get_facility` (`facility`) return the **same** richer object, so a shortlist does not require a detail call per home.

Identity, location, and CMS star ratings are included (`provnum`, `web`, `provname`, `lbn`, `address`, `city`, `state`, `zip`, `latitude`, `longitude`, `phone`, `overall_rating`, `survey_rating`, `quality_rating`, `staffing_rating`, public `urls`). Decision fields use CMS keys:

| Key | Meaning |
|-----|---------|
| `tothrd` | Total nurse hours per resident per day (HPRD) |
| `rnhrd` | RN hours per resident per day |
| `totalnumberofnursestaffhoursperresidentperdayontheweekend` | Weekend total nurse HPRD |
| `registerednursehoursperresidentperdayontheweekend` | Weekend RN HPRD |
| `totalnursingstaffturnover` | Nursing staff turnover |
| `registerednurseturnover` | RN turnover |
| `numberofadministratorswhohaveleftthenursinghome` | Administrators who have left the nursing home |
| `weighted_all_cycles_score` | Weighted health-inspection score (all cycles) |
| `bedcert` | Certified beds |
| `restot` | Residents in certified beds |
| `occupancy` | Occupancy (residents / certified beds) |
| `fine_tot` | Total fines (dollars) |
| `fine_cnt` | Number of fines |
| `abuse_icon` | CMS abuse icon |
| `sffstatus` | Special Focus Facility status |
| `chain_id` | Chain identifier |
| `chain_name` | Chain name |
| `ownership` | CMS ownership type (e.g. `For profit - Corporation`) |
| `oldsurvey` | Flag (`Y`/`N`): most recent standard health inspection is more than two years old |
| `cycle_1_survey_date` | Date of the most recent standard health inspection |

### Distance (straight-line miles, not driving minutes)

When `search_facilities` is called with `address`:

- `distance_miles` is **straight-line Haversine miles** from that origin to the facility.
- `distance` is kept as an alias of the same number.

These values are **not** driving minutes or road miles. For driving minutes, use `maps.google_dir` / `maps.apple_dir` on the row, or pass the coordinates to a mapping tool.

### Sort keys

Pass `sort` on `search_facilities` (and on `GET /api/v1/nh/facilities`). Default is `overall_rating` (stars, descending) when `sort` is omitted. Use `sort=distance` for nearest-first when searching from an address.

| `sort` value | Alias | Order |
|--------------|-------|-------|
| `overall_rating` | | Stars descending (higher better) |
| `survey_rating` | | Stars descending |
| `quality_rating` | | Stars descending |
| `staffing_rating` | | Stars descending |
| `distance` | | Miles ascending (nearest first) |
| `tothrd` | `total_nurse_hours` | Hours descending (higher better) |
| `totalnursingstaffturnover` | `nurse_turnover` | Turnover ascending (lower better) |
| `weighted_all_cycles_score` | `survey_score` | Inspection score ascending (lower better) |

### `search_facilities_by_ownership`

Same result schema as `search_facilities`, with `ownership` required. Use this when the question is “find government / nonprofit / for-profit homes in …”. Combine with `state`, `city`, `zip`, or `address`. Call `list_distinct_values` with `field=ownership` first for the exact CMS strings.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `ownership` | string | yes | CMS ownership type (prefix or exact value) |
| `state` | string | no | Two-letter state |
| `city` | string | no | City filter |
| `zip` | string | no | ZIP filter |
| `address` | string | no | Origin address for distance search |
| `filedate` | string | no | CMS monthly snapshot from `list_file_dates` |

### `summarize_facilities`

Aggregate numeric CMS ProviderInfo fields across certified SNFs. Prefer this over paging `search_facilities` for national or grouped stats (government vs private hours, state means, etc.). Means are **facility-equal** (not bed-weighted). Facilities missing a metric are excluded from that metric’s n/mean/min/max only — values are not imputed. Not causal. Not a substitute for `get_facility` on a named home.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `metrics` | array of strings | yes | 1–20 keys: `tothrd`, `rnhrd`, `weekend_tothrd`, `weekend_rnhrd`, `totalnursingstaffturnover`, `registerednurseturnover`, `weighted_all_cycles_score`, `overall_rating`, `survey_rating`, `staffing_rating`, `quality_rating`, `bedcert`, `restot`, `fine_tot`, `fine_cnt`, `occupancy`. Synonyms: `total_nurse_hours`, `nurse_turnover`, `rn_turnover`, `survey_score` |
| `group_by` | string | no | `ownership`, `ownership_kind` (Government / For profit / Non profit), `state`, `overall_rating`, `staffing_rating`, `survey_rating`, `quality_rating`, `sffstatus`, `abuse_icon`. Omit for one national (or filter-scoped) row |
| `ownership` | string | no | Same prefix/exact filter as `search_facilities` |
| `state` | string | no | Two-letter state |
| `city` | string | no | Requires `state` |
| `zip` | string | no | ZIP filter |
| `abuse_icon` | string | no | Same as `search_facilities` |
| `sffstatus` | string | no | Same as `search_facilities` |
| `min_overall_rating` | integer | no | 1–5 |
| `max_overall_rating` | integer | no | 1–5 |
| `include_median` | boolean | no | Default false. When true, each stats object includes `median` |
| `filedate` | string | no | CMS monthly snapshot from `list_file_dates` |

REST: `GET /api/v1/nh/facilities/summary?metrics=tothrd,rnhrd&group_by=ownership_kind&include_median=true`

Response `groups[]` each have `group_key`, `group_label`, `facilities_in_group`, and `stats` keyed by canonical metric (`n`, `mean`, `min`, `max`, optional `median`, `unit`). `facilities_in_scope` is the filter-matched count; per-metric `n` is the non-null count in that group. Unknown metric keys return `invalid_metric` with the allowed list.

0% / 100% turnover and near-zero HPRD can be reporting artifacts.

### `list_distinct_values`

Lists distinct CMS `ProviderInfo` values and row counts for a field in the active monthly snapshot. Omit `field` to see the allowed field catalog. `field=city` requires `state`.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `field` | string | no | `ownership`, `state`, `sffstatus`, `abuse_icon`, `certification`, `chainname`, `overall_rating`, … |
| `state` | string | no | Two-letter state (required for `city`) |
| `filedate` | string | no | CMS monthly snapshot from `list_file_dates` |

### `compare_facilities`

Same decision fields as `search_facilities`, for up to 25 CMS provider numbers or web slugs, on one snapshot. Ids past 25 are dropped. This is not a month-to-month diff; use `get_facility_changes` for that.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `ids` | array of strings | yes | CMS provider numbers or web slugs. Maximum 25 |
| `filedate` | string | no | CMS monthly snapshot from `list_file_dates` |

### `get_facility`

Opens one home by CMS provider number (`provnum`) or site slug. The `facility` object matches a `search_facilities` row (hours, turnover, inspection, beds, penalties, maps links).

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | string | yes | Provider number or web slug |
| `filedate` | string | no | CMS monthly snapshot from `list_file_dates` |

### `get_facility_changes`

Diff one home across two CMS monthly snapshots. Omit `from` and `to` to compare the latest file with the previous snapshot. `changes` is field-level: stars, nurse hours, turnover, weighted inspection score, the CMS abuse icon, Special Focus status, ownership type (for-profit / nonprofit / government, not the owner’s name), fines, beds, residents, and occupancy. A home missing from one of the two files is `found.from` or `found.to` false. That is not a closure.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | string | yes | CMS provider number or web slug |
| `from` | string | no | Earlier filedate (`YYYY-MM-DD` or `YYYY-MM`). Omit to use the snapshot before `to` |
| `to` | string | no | Later filedate. Omit to use the latest CMS file |

### `get_facility_ownership`

Lists who owns a home and in what role.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | string | yes | Provider number or web slug |
| `filedate` | string | no | CMS monthly snapshot from `list_file_dates` |

### `search_owners`

Finds nursing-home owners by name.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `q` | string | yes | Owner name search term |
| `limit` | integer | no | Max results |
| `offset` | integer | no | Pagination offset |
| `filedate` | string | no | CMS monthly snapshot from `list_file_dates` |

### `get_owner`

Opens an owner’s portfolio of certified SNFs by `web_owner` slug.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `web_owner` | string | yes | Owner slug identifier |
| `filedate` | string | no | CMS monthly snapshot from `list_file_dates` |

### `list_file_dates`

Lists CMS monthly snapshot dates available in the database. No parameters. Returns `current`, `filedates` (newest first), and `known_gaps` for unpublished months.

### `get_data_freshness`

Reports latest CMS source dataset dates and row counts. No parameters. For a prior month, call `list_file_dates` and pass `filedate` to the other tools.

## MCP Registry

Listed in the official MCP Registry as **`com.nursinghomedatabase/mcp`**.

- Registry manifest: [`server.json`](./server.json)
- Search: `https://registry.modelcontextprotocol.io/v0.1/servers?search=com.nursinghomedatabase`

## REST API (same backend)

JSON discovery APIs are also available under the main site:

- `GET https://www.nursinghomedatabase.com/api/v1/nh/capabilities`
- `GET https://www.nursinghomedatabase.com/api/v1/nh/file-dates`
- `GET https://www.nursinghomedatabase.com/api/v1/nh/facilities?address=150+Corporate+Woods+Drive,+Magnolia,+TX+77354`
- `GET https://www.nursinghomedatabase.com/api/v1/nh/facilities?address=150+Corporate+Woods+Drive,+Magnolia,+TX+77354&radius_miles=10&sort=distance`
- `GET https://www.nursinghomedatabase.com/api/v1/nh/facilities?state=TX&min_overall_rating=4&sort=tothrd`
- `GET https://www.nursinghomedatabase.com/api/v1/nh/facilities?state=TX&filedate=2026-07-01`
- `GET https://www.nursinghomedatabase.com/api/v1/nh/field-values?field=ownership`
- `GET https://www.nursinghomedatabase.com/api/v1/nh/facilities?state=TX&ownership=Non+profit`
- `GET https://www.nursinghomedatabase.com/api/v1/nh/facilities/summary?metrics=tothrd,rnhrd,totalnursingstaffturnover&group_by=ownership_kind`

REST callers can pass `filedate=YYYY-MM-DD` (or `YYYY-MM`) on facilities and owners endpoints the same way MCP tools do. `min_overall_rating`, `max_overall_rating`, `abuse_icon`, `sffstatus`, and `ownership` are query parameters on `GET /api/v1/nh/facilities`.

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
