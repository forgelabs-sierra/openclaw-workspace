---
name: zotero
description: "Search, browse, and create entries in Cameron's Zotero research library (6,300+ items) via flzotero CLI."
homepage: https://www.zotero.org/support/dev/web_api/v3/basics
metadata: {"clawdbot":{"emoji":"📚","requires":{"bins":["flzotero"]}}}
---

# Zotero Library Access

Query and manage Cameron's Zotero research library via the `flzotero` CLI. All requests route through the host clitools API — no credentials needed in the sandbox.

- **Reads (GET)** — routed to local Zotero desktop API (fast, unlimited)
- **Writes (POST/PUT/PATCH/DELETE)** — routed to remote Zotero web API (auth injected by host, rate limited)

## References

- `references/api-reference.md` (complete Zotero Web API v3 endpoint reference — read + write endpoints, search parameters, sorting, pagination, export formats, item types, rate limiting)

## Usage

```bash
flzotero METHOD PATH [BODY] [OPTIONS]
```

- `METHOD` — GET, POST, PUT, PATCH, DELETE
- `PATH` — relative to user root (e.g. `/items`, `/collections`)
- `BODY` — JSON string for writes, or `@file.json` to read from file
- `--version N` — sets `If-Unmodified-Since-Version` header (needed for updates/deletes)
- `--header KEY:VALUE` — extra header (repeatable)

Output: JSON to stdout, metadata (version, total, rate-limit) to stderr.

## Library Overview

- **Items:** ~6,300 (webpages, blog posts, journal articles, books, software, videos)
- **Collections:** All (3550), Astronomy (58), Everything (2518), Moon-News (1), Python (1), Redshift Drift (5), Supernova-Curve Fitting (2)
- **Library ID:** 12230544
- **Zotero version:** 8.0.4

## Read Operations

### Search items by keyword

```bash
flzotero GET '/items?q=machine+learning&limit=10&format=json'
```

### Full-text search (searches PDF content too)

```bash
flzotero GET '/items?q=neural+networks&qmode=everything&limit=10&format=json'
```

### Get a specific item by key

```bash
flzotero GET '/items/ITEMKEY?format=json'
```

### Get item's children (attachments, notes)

```bash
flzotero GET '/items/ITEMKEY/children?format=json'
```

### Recent items (newest first)

```bash
flzotero GET '/items?sort=dateAdded&direction=desc&limit=10&format=json'
```

### List collections

```bash
flzotero GET '/collections?format=json'
```

### Items in a collection

```bash
flzotero GET '/collections/COLLKEY/items?format=json'
```

### Search by item type

```bash
flzotero GET '/items?itemType=journalArticle&limit=20&format=json'
```

### Search by tag

```bash
flzotero GET '/items?tag=TAGNAME&limit=20&format=json'
```

### List all tags

```bash
flzotero GET /tags
```

### Top-level items only (excludes attachments/notes)

```bash
flzotero GET '/items/top?limit=20&format=json'
```

## Write Operations

### Create an item

Items are sent as a JSON array (even for a single item). Max 50 per request.

```bash
flzotero POST /items '[{
  "itemType": "webpage",
  "title": "Article Title",
  "url": "https://example.com/article",
  "creators": [{"creatorType": "author", "firstName": "Jane", "lastName": "Doe"}],
  "abstractNote": "Brief summary of the article.",
  "websiteTitle": "Site Name",
  "tags": [{"tag": "topic1"}, {"tag": "topic2"}],
  "accessDate": "2026-03-08T10:00:00Z"
}]'
```

Response includes `successful`, `success`, `unchanged`, and `failed` objects. Check `successful.0.key` for the new item key. The `version` number is printed to stderr.

### Update an item

Requires the item's current `version` number (from a previous read or write response — printed to stderr).

```bash
# First, get the current item (version printed to stderr)
flzotero GET '/items/ITEMKEY?format=json'

# Then update with that version
flzotero PATCH /items/ITEMKEY '{"title": "Updated Title", "tags": [{"tag": "updated"}]}' --version VERSION
```

### Delete an item

Also requires the `--version` flag.

```bash
flzotero DELETE '/items?itemKey=ITEMKEY' --version VERSION
```

Returns HTTP 204 on success.

### Delete multiple items

```bash
flzotero DELETE '/items?itemKey=KEY1,KEY2,KEY3' --version LIBRARY_VERSION
```

## Rate Limit Handling

The remote API returns HTTP 429 when rate limited:
- flzotero prints `rate-limited: retry after Ns` to stderr
- Wait that many seconds, then retry
- For batch operations, space requests 1-2 seconds apart
- Reads are NOT rate limited — routed to local Zotero desktop API

## Search Syntax

### Boolean operators for itemType and tag

- `itemType=book` — single type
- `itemType=book || journalArticle` — OR
- `itemType=-attachment` — NOT (exclude type)
- `tag=foo&tag=bar` — AND (multiple tag params)
- `tag=foo || bar` — OR
- `tag=-foo` — NOT

### Quick search modes

- `qmode=titleCreatorYear` (default) — searches titles and creator names
- `qmode=everything` — includes full-text content of PDFs/documents

## Sorting and Pagination

| Parameter | Values | Default |
|-----------|--------|---------|
| `sort` | `dateAdded`, `dateModified`, `title`, `creator`, `itemType`, `date`, `publisher` | `dateModified` |
| `direction` | `asc`, `desc` | varies by sort |
| `limit` | 1-100 | 25 |
| `start` | integer | 0 |

Paginate by incrementing `start` by `limit` each request. The `total` count is printed to stderr.

## Response Structure

Each item returns JSON with this structure:

```json
{
  "key": "SC8E885Y",
  "version": 4951,
  "library": { "type": "user", "id": 12230544, "name": "My Library" },
  "meta": { "numChildren": 1 },
  "data": {
    "key": "SC8E885Y",
    "itemType": "webpage",
    "title": "Example Title",
    "abstractNote": "...",
    "url": "https://...",
    "creators": [{ "creatorType": "author", "firstName": "...", "lastName": "..." }],
    "tags": [{ "tag": "machine-learning" }],
    "collections": ["COLLKEY1"],
    "dateAdded": "2026-03-07T12:11:41Z",
    "dateModified": "2026-03-07T12:11:41Z"
  }
}
```

The `data` block contains all the item fields. Key fields: `title`, `abstractNote`, `url`, `creators`, `tags`, `itemType`, `dateAdded`.

## Common Item Types

`book`, `journalArticle`, `webpage`, `blogPost`, `conferencePaper`, `report`, `thesis`, `computerProgram`, `videoRecording`, `document`, `note`, `attachment`

## Tips

- All reads and writes go through `flzotero` — no need to manage endpoints or credentials
- URL-encode search strings: spaces as `+` or `%20`
- Use `jq` to filter JSON responses: `flzotero GET '/items?limit=5&format=json' | jq '.[].data.title'`
- The API returns max 100 items per request — paginate with `start` and `limit`
- Attachments and notes are child items — use `/items/top` to get only parent items
- Cameron's library focuses on: AI/ML, astronomy (redshift drift, supernovae), Python, software tools
- Zotero must be running on the host Mac for reads to work — if reads fail with a connection error, ask Cameron to start Zotero
- Writes always go to the remote API and work regardless of whether Zotero desktop is running
