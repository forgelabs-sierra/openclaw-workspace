# Zotero Web API v3 — Endpoint Reference

Complete reference for the Zotero API as available from inside the OpenClaw container.
Source: https://www.zotero.org/support/dev/web_api/v3/basics

## Connection — Two APIs

### Local API (Reads — fast, unlimited)

```
Base URL: http://host.containers.internal:23119/api/users/0/
Required header: -H 'Host: localhost:23119'
Auth: None
Format: Always append ?format=json
Rate limits: None
Requires: Zotero desktop running on host Mac
```

### Remote API (Writes — rate limited)

```
Base URL: https://api.zotero.org/users/$ZOTERO_USER_ID/
Auth header: -H "Zotero-API-Key: $ZOTERO_API_KEY"
Format: ?format=json
Rate limits: HTTP 429 with Retry-After header
```

---

## Read Endpoints

All paths below are relative to the base URL.

### Items

| Endpoint | Description |
| --- | --- |
| `items` | All items, excluding trashed |
| `items/top` | Top-level items only (no attachments/notes) |
| `items/trash` | Items in the trash |
| `items/<itemKey>` | A specific item |
| `items/<itemKey>/children` | Child items (attachments, notes) |
| `items/<itemKey>/tags` | Tags on a specific item |
| `collections/<collectionKey>/items` | Items in a collection |
| `collections/<collectionKey>/items/top` | Top-level items in a collection |

### Collections

| Endpoint | Description |
| --- | --- |
| `collections` | All collections |
| `collections/top` | Top-level collections only |
| `collections/<collectionKey>` | A specific collection |
| `collections/<collectionKey>/collections` | Subcollections |

### Tags

| Endpoint | Description |
| --- | --- |
| `tags` | All tags in the library |
| `items/tags` | All tags with item-based filtering |
| `items/top/tags` | Tags on top-level items |
| `collections/<collectionKey>/tags` | Tags in a collection |
| `collections/<collectionKey>/items/tags` | Tags on items in a collection |

### Searches

| Endpoint | Description |
| --- | --- |
| `searches` | All saved searches (metadata only) |

---

## Search Parameters

### General (all item endpoints)

| Parameter | Values | Default | Description |
| --- | --- | --- | --- |
| `q` | string | — | Quick search (titles + creator fields by default) |
| `qmode` | `titleCreatorYear`, `everything` | `titleCreatorYear` | Search mode. `everything` includes full-text content |
| `itemType` | search syntax | — | Filter by item type |
| `tag` | search syntax | — | Filter by tag |
| `itemKey` | comma-separated keys | — | Fetch specific items (up to 50) |
| `since` | integer | 0 | Only objects modified after this version |
| `includeTrashed` | `0`, `1` | `0` | Include trashed items |

### Tag endpoints

| Parameter | Values | Default | Description |
| --- | --- | --- | --- |
| `qmode` | `contains`, `startsWith` | `contains` | Tag search mode |

### Search Syntax (Boolean)

For `itemType` and `tag` parameters:

```
Single:     itemType=book
OR:         itemType=book || journalArticle
NOT:        itemType=-attachment
Tag AND:    tag=foo&tag=bar        (multiple params)
Tag OR:     tag=foo || bar
Tag NOT:    tag=-foo
Literal -:  tag=\-foo
```

---

## Sorting & Pagination

| Parameter | Values | Default |
| --- | --- | --- |
| `sort` | `dateAdded`, `dateModified`, `title`, `creator`, `itemType`, `date`, `publisher`, `publicationTitle`, `journalAbbreviation`, `language`, `accessDate`, `libraryCatalog`, `callNumber`, `rights`, `addedBy`, `numItems` (tags only) | `dateModified` |
| `direction` | `asc`, `desc` | varies by sort field |
| `limit` | 1–100 | 25 |
| `start` | integer | 0 |

### Pagination pattern

```bash
# Page 1
curl ... '...?limit=25&start=0&format=json'
# Page 2
curl ... '...?limit=25&start=25&format=json'
# Page 3
curl ... '...?limit=25&start=50&format=json'
```

---

## Response Format

### format=json (default, recommended)

Returns JSON array for multi-object requests, single JSON object for single-object requests.

#### Item structure

```json
{
  "key": "ABCD1234",
  "version": 1234,
  "library": {
    "type": "user",
    "id": 12230544,
    "name": "My Library"
  },
  "meta": {
    "numChildren": 2,
    "creatorSummary": "Smith",
    "parsedDate": "2024-01-15"
  },
  "data": {
    "key": "ABCD1234",
    "version": 1234,
    "itemType": "journalArticle",
    "title": "Article Title",
    "abstractNote": "Abstract text...",
    "url": "https://...",
    "date": "2024-01-15",
    "language": "en",
    "DOI": "10.1234/example",
    "ISSN": "1234-5678",
    "publicationTitle": "Journal Name",
    "volume": "42",
    "issue": "3",
    "pages": "100-115",
    "creators": [
      { "creatorType": "author", "firstName": "John", "lastName": "Smith" },
      { "creatorType": "author", "name": "Research Lab" }
    ],
    "tags": [
      { "tag": "machine-learning" },
      { "tag": "neural-networks", "type": 1 }
    ],
    "collections": ["COLL1234"],
    "relations": {},
    "dateAdded": "2024-01-20T10:30:00Z",
    "dateModified": "2024-01-20T10:30:00Z"
  }
}
```

#### Collection structure

```json
{
  "key": "COLL1234",
  "version": 100,
  "data": {
    "key": "COLL1234",
    "name": "My Collection",
    "parentCollection": false
  },
  "meta": {
    "numCollections": 2,
    "numItems": 150
  }
}
```

### format=keys

Returns newline-separated list of object keys. No limit cap.

### format=versions

Returns JSON object mapping keys to version numbers:

```json
{
  "ABCD1234": 1234,
  "EFGH5678": 5678
}
```

---

## Item Types

### Common types

| Type | Description |
| --- | --- |
| `book` | Book |
| `bookSection` | Book Section / Chapter |
| `journalArticle` | Journal Article |
| `conferencePaper` | Conference Paper |
| `webpage` | Web Page |
| `blogPost` | Blog Post |
| `report` | Report |
| `thesis` | Thesis |
| `document` | Document |
| `computerProgram` | Software |
| `videoRecording` | Video Recording |
| `podcast` | Podcast |
| `presentation` | Presentation |
| `patent` | Patent |
| `preprint` | Preprint |
| `letter` | Letter |
| `manuscript` | Manuscript |

### Special types

| Type | Description |
| --- | --- |
| `note` | Standalone note |
| `attachment` | File attachment (PDF, snapshot, etc.) |

### Getting all valid types

```bash
curl -s 'https://api.zotero.org/itemTypes' | jq '.[].itemType'
```

### Getting valid fields for a type

```bash
curl -s 'https://api.zotero.org/itemTypeFields?itemType=journalArticle' | jq '.[].field'
```

### Getting a blank template

```bash
curl -s 'https://api.zotero.org/items/new?itemType=book'
```

---

## Creator Types

Vary by item type. Common ones:

| Creator Type | Description |
| --- | --- |
| `author` | Author |
| `editor` | Editor |
| `contributor` | Contributor |
| `translator` | Translator |
| `reviewedAuthor` | Reviewed Author |
| `seriesEditor` | Series Editor |

Creator format (two forms):

```json
{ "creatorType": "author", "firstName": "John", "lastName": "Smith" }
{ "creatorType": "author", "name": "Research Lab Inc." }
```

---

## Item Fields by Type

### journalArticle

`title`, `abstractNote`, `publicationTitle`, `volume`, `issue`, `pages`, `date`, `series`, `seriesTitle`, `seriesText`, `journalAbbreviation`, `language`, `DOI`, `ISSN`, `shortTitle`, `url`, `accessDate`, `archive`, `archiveLocation`, `libraryCatalog`, `callNumber`, `rights`, `extra`, `tags`, `collections`, `relations`

### book

`title`, `abstractNote`, `series`, `seriesNumber`, `volume`, `numberOfVolumes`, `edition`, `place`, `publisher`, `date`, `numPages`, `language`, `ISBN`, `shortTitle`, `url`, `accessDate`, `archive`, `archiveLocation`, `libraryCatalog`, `callNumber`, `rights`, `extra`, `tags`, `collections`, `relations`

### webpage

`title`, `abstractNote`, `websiteTitle`, `websiteType`, `date`, `shortTitle`, `url`, `accessDate`, `language`, `rights`, `extra`, `tags`, `collections`, `relations`

### blogPost

`title`, `abstractNote`, `blogTitle`, `websiteType`, `date`, `url`, `accessDate`, `language`, `shortTitle`, `rights`, `extra`, `tags`, `collections`, `relations`

---

## Export Formats (Web API only)

These work with the web API (`api.zotero.org`) but NOT with the local API:

| Format | Description |
| --- | --- |
| `bibtex` | BibTeX |
| `biblatex` | BibLaTeX |
| `csljson` | CSL-JSON |
| `csv` | CSV |
| `ris` | RIS |
| `mods` | MODS |
| `tei` | TEI |
| `wikipedia` | Wikipedia Citation Templates |
| `refer` | Refer/BibIX |
| `rdf_zotero` | Zotero RDF |
| `rdf_dc` | Dublin Core RDF |

Usage: `?format=bibtex` or `?include=data,bibtex` (with format=json)

### Bibliography formatting (Web API only)

| Parameter | Values | Default |
| --- | --- | --- |
| `style` | CSL style name (e.g., `apa`, `chicago-note-bibliography`) | `chicago-note-bibliography` |
| `linkwrap` | `0`, `1` | `0` |
| `locale` | e.g., `en-US`, `fr-FR` | `en-US` |

---

## Write Endpoints (Remote API Only)

All write operations use the remote API at `https://api.zotero.org/users/$ZOTERO_USER_ID/`.
Every request requires `-H "Zotero-API-Key: $ZOTERO_API_KEY"`.

### Create Items (POST)

```
POST /users/$ZOTERO_USER_ID/items
Content-Type: application/json
```

Body: JSON array of item objects (max 50 per request).

```json
[{
  "itemType": "webpage",
  "title": "Article Title",
  "url": "https://example.com",
  "creators": [{"creatorType": "author", "firstName": "Jane", "lastName": "Doe"}],
  "abstractNote": "Summary text.",
  "websiteTitle": "Site Name",
  "tags": [{"tag": "topic"}],
  "accessDate": "2026-03-08T10:00:00Z"
}]
```

Response (HTTP 200):

```json
{
  "successful": { "0": { "key": "NEWKEY", "version": 1234, "data": {...} } },
  "success": { "0": "NEWKEY" },
  "unchanged": {},
  "failed": {}
}
```

### Update an Item (PATCH)

Requires `If-Unmodified-Since-Version` header with the item's current version number.

```
PATCH /users/$ZOTERO_USER_ID/items/<itemKey>
Content-Type: application/json
If-Unmodified-Since-Version: <version>
```

Body: JSON object with only the fields to change.

```json
{"title": "Updated Title", "tags": [{"tag": "new-tag"}]}
```

Returns HTTP 204 on success. Returns HTTP 412 if the version has changed (conflict).

### Delete an Item (DELETE)

```
DELETE /users/$ZOTERO_USER_ID/items/<itemKey>
If-Unmodified-Since-Version: <version>
```

Returns HTTP 204 on success.

### Delete Multiple Items

```
DELETE /users/$ZOTERO_USER_ID/items?itemKey=KEY1,KEY2,KEY3
If-Unmodified-Since-Version: <libraryVersion>
```

Max 50 keys per request.

### Create a Collection (POST)

```
POST /users/$ZOTERO_USER_ID/collections
Content-Type: application/json
```

```json
[{"name": "New Collection", "parentCollection": false}]
```

### Add Item to Collection

Update the item's `collections` array to include the collection key:

```
PATCH /users/$ZOTERO_USER_ID/items/<itemKey>
If-Unmodified-Since-Version: <version>
```

```json
{"collections": ["EXISTINGCOLL1", "NEWCOLLKEY"]}
```

### Get an Item Template

Get a blank template for any item type (useful before creating items):

```bash
curl -s "https://api.zotero.org/items/new?itemType=webpage"
curl -s "https://api.zotero.org/items/new?itemType=journalArticle"
curl -s "https://api.zotero.org/items/new?itemType=book"
```

No auth required — this is a public endpoint.

### Version Handling

Every item has a `version` number. To update or delete, you must provide the current version:

1. GET the item → read `version` from the response
2. Include `If-Unmodified-Since-Version: <version>` in the PATCH/DELETE request
3. If the item was modified between GET and PATCH, the server returns HTTP 412 (Precondition Failed)
4. On 412: re-GET the item, check for conflicts, retry with new version

### Rate Limiting

The remote API enforces rate limits:

- HTTP 429 response when limit exceeded
- `Retry-After` header indicates seconds to wait
- `Backoff` header may also be present
- For batch operations, space requests 1-2 seconds apart
- Use local API for all reads to avoid rate limit consumption

---

## Local API Limitations

The local Zotero API (port 23119) has these limitations compared to the web API:

| Feature | Local API | Web API |
| --- | --- | --- |
| Authentication | None needed | API key required |
| Read items | Yes | Yes |
| Write/create items | **No** | Yes |
| Delete items | **No** | Yes |
| Export formats (bibtex, ris, etc.) | **No** | Yes |
| Atom format | **No** | Yes |
| Bibliography formatting | **No** | Yes |
| Rate limiting | None | Yes (backoff headers) |
| Full-text content endpoint | **No** | Yes |
| Requires Zotero running | **Yes** | No |

---

## Practical Examples

### Find recent AI papers

```bash
curl -s -H 'Host: localhost:23119' \
  'http://host.containers.internal:23119/api/users/0/items?q=artificial+intelligence&itemType=journalArticle||conferencePaper||preprint&sort=dateAdded&direction=desc&limit=10&format=json' \
  | jq '.[].data | {title, date, url, creators: [.creators[].lastName // .creators[].name]}'
```

### Get all items in Astronomy collection

First get collection keys:

```bash
curl -s -H 'Host: localhost:23119' \
  'http://host.containers.internal:23119/api/users/0/collections?format=json' \
  | jq '.[] | {key: .key, name: .data.name, items: .meta.numItems}'
```

Then fetch items using the collection key:

```bash
curl -s -H 'Host: localhost:23119' \
  'http://host.containers.internal:23119/api/users/0/collections/COLLKEY/items/top?format=json' \
  | jq '.[].data | {title, itemType, date, url}'
```

### Search across everything (including PDF full-text)

```bash
curl -s -H 'Host: localhost:23119' \
  'http://host.containers.internal:23119/api/users/0/items?q=cosmological+redshift&qmode=everything&limit=20&format=json' \
  | jq '.[].data | {title, itemType, abstractNote}'
```

### Get items added in the last 7 days

```bash
curl -s -H 'Host: localhost:23119' \
  'http://host.containers.internal:23119/api/users/0/items/top?sort=dateAdded&direction=desc&limit=50&format=json' \
  | jq '[.[] | select(.data.dateAdded > "2026-03-01")] | .[].data | {title, dateAdded, itemType}'
```

### Browse by tag

```bash
# List all tags
curl -s -H 'Host: localhost:23119' \
  'http://host.containers.internal:23119/api/users/0/tags' \
  | jq '.[].tag'

# Get items with a specific tag
curl -s -H 'Host: localhost:23119' \
  'http://host.containers.internal:23119/api/users/0/items?tag=python&format=json' \
  | jq '.[].data | {title, url}'
```

### Paginate through all items

```bash
# Get first 100
curl -s -H 'Host: localhost:23119' \
  'http://host.containers.internal:23119/api/users/0/items/top?limit=100&start=0&format=json' | jq length
# Get next 100
curl -s -H 'Host: localhost:23119' \
  'http://host.containers.internal:23119/api/users/0/items/top?limit=100&start=100&format=json' | jq length
```
