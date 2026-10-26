# crawler — Production-Ready Web Crawler & Spider Engine for Mozhi

A general-purpose web crawler, spider, pure DOM tree parser, CSS selector engine, persistent cache, and structured data extractor written **100% in pure Mozhi**. Built exclusively on native Mozhi standard modules (`http`, `json`, `fs`, `db`), with **zero external runtime dependencies** and **built-in SSRF protection**.

[![Release](https://img.shields.io/github/v/release/crossberry-in/mozhi-crawler?color=blue)](https://github.com/crossberry-in/mozhi-crawler/releases)
[![Docs](https://img.shields.io/badge/docs-GitHub%20Pages-1976d2)](https://crossberry-in.github.io/mozhi-crawler/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Pure Mozhi](https://img.shields.io/badge/Language-Pure%20Mozhi%203.2-brightgreen)](https://github.com/crossberry-in/mozhi-doc)

🌐 **Interactive Documentation Site**: [https://crossberry-in.github.io/mozhi-crawler/](https://crossberry-in.github.io/mozhi-crawler/)

---

## Architecture & Visual Diagrams

### 1. End-to-End Spider Data Pipeline
```
 +---------------------------------------------------------------------------------------+
 |                                  URL Frontier (BFS Queue)                             |
 +---------------------------------------------------------------------------------------+
                                            │
                                            ▼
 +---------------------------------------------------------------------------------------+
 |                     Security & SSRF Validation (RFC 1918 / Loopback)                  |
 +---------------------------------------------------------------------------------------+
                     │                                              │
          [Destination Allowed]                             [Blocked / Loopback]
                     ▼                                              ▼
 +---------------------------------------+                  +----------------------------+
 |      Storage & Cache Decision         |                  |  Return ERR_SSRF_BLOCKED   |
 |  (Live Fetch vs Offline Storage Read) |                  +----------------------------+
 +---------------------------------------+
                     │
      ┌──────────────┴──────────────┐
  [Live Fetch]                 [Offline Cache Read]
      ▼                             ▼
 [HTTP Request / Redirects]    [Read Stored Record]
      │                             │
      └──────────────┬──────────────┘
                     ▼
 +---------------------------------------------------------------------------------------+
 |                  Pure-Mozhi DOM Tree Parser & CSS Selector Engine                     |
 +---------------------------------------------------------------------------------------+
                                            │
                                            ▼
 +---------------------------------------------------------------------------------------+
 |           Structured Extraction (Schema, OpenGraph, JSON-LD, Media & Tables)          |
 +---------------------------------------------------------------------------------------+
                                            │
                                            ▼
 +---------------------------------------------------------------------------------------+
 |         Link Graph Engine (Incoming/Outgoing Topology, Cycle & Duplicate Detection)   |
 +---------------------------------------------------------------------------------------+
                                            │
                                            ▼
 +---------------------------------------------------------------------------------------+
 |            Multi-Format Exporters (JSON, JSONL, CSV, Link Graph JSON/CSV)             |
 +---------------------------------------------------------------------------------------+
```

### 2. Live Fetch vs Storage Decision Matrix

| Option Combination | Mode Description | Network Requests? | Persists to Disk? | Behavior on Cache Miss |
|---|---|:---:|:---:|---|
| `live_fetch: true, save_db: false` | **Live Ephemeral** | **YES** | **NO** | Fetches live data; in-memory only |
| `live_fetch: true, save_db: true` | **Live & Persist** | **YES** | **YES** | Fetches live, computes fingerprint, writes cache |
| `live_fetch: false, save_db: false` | **Offline Read** | **NO** | **NO** | Reads previously stored cache record |
| `live_fetch: false, save_db: true` | **Offline Read** | **NO** | **NO** | Reads stored cache record; never fetches |
| `offline: true` | **Offline Alias** | **NO** | Configurable | Convenience alias for `live_fetch: false` |

```
                       [ Incoming Spider Request ]
                                    │
                                    ▼
                     Is offline=true or live_fetch=false?
                               /         \
                            [YES]        [NO] (Live Fetch Enabled)
                             /             \
             Lookup record in cache         Issue HTTP Request (With SSRF checks)
                 /             \                       │
             [Found]        [Not Found]         Compute content fingerprint
               /                 \                     │
     Return cached page   ERR_OFFLINE_CACHE_MISS       Is save_db=true?
     (Zero network calls) (Never calls network)      /         \
                                                  [YES]        [NO]
                                                   /             \
                                        Persist to Cache      Return in-memory
```

---

## Features

* **100% Native Pure Mozhi**: Zero external runtimes, Python, Node.js, PHP, or Go bridges. Built strictly on native standard modules (`http`, `json`, `fs`, `db`).
* **Advanced Web Spider (`crawler.spider`)**:
  * Breadth-first crawl queue with URL normalization and deduplication.
  * Per-domain request budgets and page limits (`max_pages_per_domain`, `max_external_domains`).
  * Hard limits on total HTTP calls (`max_total_requests`), depth (`max_depth`), and pages (`max_pages`).
  * External link controls: Distinguishes link discovery from link fetching (`same_domain`, `same_origin`).
  * Incremental change detection via deterministic content fingerprinting (`new`, `unchanged`, `changed`).
* **Dual Persistent Storage Backends**:
  * Native filesystem JSON cache (`.crawler_cache/`).
  * Native SQLite database backend (`crawler.db`).
  * TTL expiration, corrupt cache recovery, and cache invalidation.
* **Full CSS Selector Engine**:
  * Tags, universal `*`, IDs (`#id`), classes (`.class`, `.card.featured`).
  * Child combinator (`>`) and descendant combinator (space ` `).
  * Selector groups (comma `,`).
  * Attribute existence and operators (`[attr]`, `[attr="val"]`, `^=`, `$=`, `*=`, `~=`).
  * Structural pseudo-classes (`:first-child`, `:last-child`, `:nth-child()`, `:not()`).
* **Declarative Schema Extraction (`crawler.extract`)**:
  * Map HTML elements directly into structured objects, lists, and custom attributes.
* **Metadata & Structured Data**:
  * Open Graph (`og:*`), Twitter Cards (`twitter:*`), canonical links, titles, descriptions.
  * JSON-LD extraction and parsing from `<script type="application/ld+json">`.
  * HTML `<table>` parsing into structured headers and rows.
* **Link Graph & Website Analysis**:
  * Incoming and outgoing link graphs (`get_incoming_links`, `get_outgoing_links`).
  * Confirmed broken link discovery (`find_broken_links`).
  * Duplicate content detection (`find_duplicate_content`).
  * Link graph exports (`export_graph_json`, `export_graph_csv`).
* **SSRF Security**:
  * Blocks loopback (`127.0.0.0/8`, `::1`), private RFC 1918 subnets (`10/8`, `172.16/12`, `192.168/16`), and cloud metadata (`169.254.169.254`).
  * Revalidates every redirect hop against security constraints.
* **Multi-Format Exports**:
  * JSON, JSON Lines (`.jsonl`), CSV, Graph JSON, Graph CSV, and categorized terminal ASCII tables.

---

## Installation

```bash
mz install @crossberry-in/mozhi-crawler
```

Or add to your `package.toml`:

```toml
[dependencies]
crawler = "@crossberry-in/mozhi-crawler"
```

---

## Quickstart Examples

### 1. Running the Advanced Web Spider
```mozhi
import crawler

let result = crawler.spider({
    url: "https://example.com",
    live_fetch: true,
    save_db: true,
    cache_dir: ".crawler_cache",
    max_pages: 50,
    max_depth: 3,
    same_domain: true,
    max_total_requests: 100,
    respect_robots: true
})

echo("Spider Success: " + string(result.ok))
echo("Pages Crawled:  " + string(result.pages.len()))
echo("Total Requests: " + string(result.stats.total_requests))
echo("Stopped Reason: " + result.stopped_reason)
```

### 2. Reading from Cache in Offline Mode
```mozhi
import crawler

# Zero network calls; strictly reads from stored cache:
let offline = crawler.spider({
    url: "https://example.com",
    offline: true,
    cache_dir: ".crawler_cache",
    max_pages: 10
})

if offline.ok {
    echo("Read " + string(offline.pages.len()) + " pages from cache.")
    echo("Network calls made: " + string(offline.stats.total_requests)) # 0
} else {
    echo("Cache miss: " + offline.errors[0].message)
}
```

### 3. CSS Selector Queries
```mozhi
import crawler

let html = "<div id='catalog'><article class='card featured' data-id='101'><h2>Mozhi Guide</h2><span class='price'>$25</span></article></div>"
let doc = crawler.parse_html(html)

let card_nid = crawler.query_selector(doc, "article.card.featured", null)
let title_nid = crawler.query_selector(doc, "h2", card_nid)

echo("Title: " + crawler.node_text(doc, title_nid))
echo("Item ID: " + crawler.node_attr(doc, card_nid, "data-id"))
```

### 4. Declarative Schema Extraction
```mozhi
import crawler

let schema = {
    "site_title": "header h1",
    "articles": {
        "selector": "article.post",
        "multiple": true,
        "fields": {
            "id": { "attribute": "data-id" },
            "heading": "h2",
            "summary": "p.snippet",
            "read_more": { "selector": "a.more", "attribute": "href" }
        }
    }
}

let page = crawler.scrape("https://example.com/blog")
let data = crawler.extract(page.doc, schema)

for a in data.articles {
    echo(a.heading + " -> " + a.read_more)
}
```

### 5. Link Graph Analysis & Graph Exports
```mozhi
import crawler

let res = crawler.spider({ url: "https://example.com", max_pages: 20 })

# Analyze link topology:
let outgoing = crawler.get_outgoing_links(res, "https://example.com/")
let incoming = crawler.get_incoming_links(res, "https://example.com/docs")
let duplicates = crawler.find_duplicate_content(res)
let broken = crawler.find_broken_links(res)

# Export graph for visualization (e.g. Gephi, D3.js, Cytoscape):
crawler.export_graph_json(res, "graph.json")
crawler.export_graph_csv(res, "graph.csv")
```

---

## API Reference

| Category | Function | Signature | Description |
|---|---|---|---|
| **Spider** | `crawler.spider` | `(config: map) -> map` | Configurable BFS web spider with limits and storage. |
| **Spider** | `crawler.spider_stats` | `(result: map) -> map` | Returns crawl metrics summary. |
| **Scraper** | `crawler.scrape` | `(url_or_opts) -> map` | Fetches and parses a single web page. |
| **Scraper** | `crawler.crawl` | `(url, max_p, max_d) -> map` | High-level crawler backward compatibility. |
| **DOM** | `crawler.parse_html` | `(html: str) -> map` | Parses HTML into node-keyed DOM tree. |
| **DOM** | `crawler.query_selector` | `(doc, sel, ctx) -> int` | First matching node ID or `null`. |
| **DOM** | `crawler.query_selector_all` | `(doc, sel, ctx) -> array` | Array of matching node IDs. |
| **DOM** | `crawler.matches_selector` | `(doc, nid, sel) -> bool` | Checks if element satisfies selector. |
| **DOM** | `crawler.node_text` | `(doc, nid) -> str` | Extracts recursive plain text. |
| **DOM** | `crawler.node_attr` | `(doc, nid, attr) -> str` | Gets element attribute value. |
| **DOM** | `crawler.node_html` | `(doc, nid) -> str` | Serializes outer HTML. |
| **Extraction** | `crawler.extract` | `(doc, schema) -> map` | Extracts structured records via schema. |
| **Extraction** | `crawler.extract_metadata` | `(doc) -> map` | Extracts SEO, Open Graph & Twitter tags. |
| **Extraction** | `crawler.extract_json_ld` | `(doc) -> array` | Extracts parsed JSON-LD objects. |
| **Extraction** | `crawler.extract_tables` | `(doc) -> array` | Parses HTML tables into arrays of rows. |
| **Link Graph**| `crawler.get_outgoing_links` | `(res, url) -> array` | Outgoing URLs originating from page. |
| **Link Graph**| `crawler.get_incoming_links` | `(res, url) -> array` | Incoming URLs targeting page. |
| **Link Graph**| `crawler.find_broken_links` | `(res) -> array` | Confirmed HTTP 4xx/5xx failures. |
| **Link Graph**| `crawler.find_duplicate_content` | `(res) -> array` | Groups URLs with identical content fingerprints. |
| **Storage** | `crawler.init_storage` | `(opts: map) -> map` | Initializes filesystem or SQLite cache. |
| **Storage** | `crawler.storage_get` | `(store, url, now, refresh)` | Retrieves cached record. |
| **Storage** | `crawler.storage_put` | `(store, url, data, body)` | Persists record if `save_db: true`. |
| **Storage** | `crawler.storage_invalidate` | `(store, url) -> bool` | Evicts cached record. |
| **Security** | `crawler.is_private_ip` | `(host: str) -> bool` | Loopback, RFC 1918, or metadata IP check. |
| **Security** | `crawler.validate_target_url` | `(url, allow_priv) -> map` | Validates scheme and destination safety. |
| **Exports** | `crawler.export_json` | `(data, path) -> bool` | Exports data as formatted JSON. |
| **Exports** | `crawler.export_jsonl` | `(records, path) -> bool`| Exports array as JSON Lines. |
| **Exports** | `crawler.export_csv` | `(headers, rows, path) -> bool` | Exports tabular data to CSV. |
| **Exports** | `crawler.export_graph_json` | `(res, path) -> bool` | Exports link graph topology as JSON. |
| **Exports** | `crawler.export_graph_csv` | `(res, path) -> bool` | Exports link graph edges as CSV. |
| **Visuals** | `crawler.print_tables` | `(data: map)` | Renders categorized ASCII terminal tables. |

---

## Error Handling & Error Codes

Every operation returns explicit, structured diagnostics:

```json
{
  "code": "ERR_OFFLINE_CACHE_MISS",
  "message": "Page not in offline cache: http://example.com/item",
  "url": "http://example.com/item",
  "operation": "fetch",
  "retryable": false,
  "suggestion": "Run spider with live_fetch=true and save_db=true first to populate offline cache."
}
```

* `ERR_CONFIG_CONFLICT`: Conflicting options (e.g. `offline=true` and `live_fetch=true`).
* `ERR_OFFLINE_CACHE_MISS`: Requested URL not found in cache while offline.
* `ERR_SSRF_BLOCKED`: Destination resolved to private, loopback, or metadata subnet.
* `ERR_RESPONSE_TOO_LARGE`: Response body exceeded `max_response_bytes`.
* `ERR_HTTP_STATUS`: Server responded with HTTP 4xx or 5xx code.
* `ERR_REDIRECT_LOOP`: Redirect cycle detected.

---

## Testing

Run the full regression test suite:

```bash
mozhi test tests/
```

Run the showcase demo:

```bash
mozhi run examples/demo.mz
```

---

## License

MIT © [Crossberry](https://github.com/crossberry-in)
