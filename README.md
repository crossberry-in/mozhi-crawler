# crawler — Production-Ready Web Crawler & Selector Engine for Mozhi

A general-purpose web crawler, pure DOM tree parser, CSS selector engine, and structured data extractor written **100% in pure Mozhi**. Built exclusively on the native Mozhi standard library (`http`, `json`, `fs`), with **zero database dependencies** and **built-in SSRF protection**.

[![Release](https://img.shields.io/github/v/release/crossberry-in/mozhi-crawler?color=blue)](https://github.com/crossberry-in/mozhi-crawler/releases)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

---

## Highlights

* **100% Pure Mozhi**: Zero external dependencies, bridges, or child processes.
* **Full CSS Selector Engine**:
  * Tag selectors (`div`, `p`, universal `*`)
  * Class selectors (`.card`, `.post.featured`)
  * ID selectors (`#app`, `div#main`)
  * Combinators (direct child `>`, descendant ` `)
  * Selector groups (comma `,`)
  * Attribute existence and operators (`[attr]`, `[attr="val"]`, `[attr^="prefix"]`, `[attr$="suffix"]`, `[attr*="contains"]`, `[attr~="word"]`)
  * Structural pseudo-classes (`:first-child`, `:last-child`, `:nth-child()`, `:not()`)
* **Declarative Schema Extraction (`crawler.extract`)**:
  * Extract structured maps and lists of records with custom selectors, attributes, and nested fields.
* **Metadata & Structured Data**:
  * Open Graph (`og:*`), Twitter Cards (`twitter:*`), canonical links, languages, titles, descriptions.
  * JSON-LD structured data extraction from `<script type="application/ld+json">`.
  * HTML `<table>` extraction into structured headers and rows.
* **Reliable Crawling**:
  * BFS queue with visited-URL deduplication, depth limits, and max page limits.
  * Domain filtering and URL pattern matching (include/exclude rules).
  * HTTP redirect following with loop detection.
  * Robots.txt parsing & path authorization (`parse_robots_txt`, `is_robots_allowed`).
  * XML Sitemap parsing (`parse_sitemap`).
* **Security & SSRF Protection**:
  * Rejection of loopback addresses (`127.0.0.0/8`, `::1`, `localhost`).
  * Rejection of RFC 1918 private IP subnets (`10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`).
  * Rejection of cloud metadata services (`169.254.169.254`) and link-local ranges.
  * Protocol scheme verification.
* **Export Formats & Visualization**:
  * Formatted ASCII categorized terminal tables for immediate inspection.
  * Live JSON output stream and file export (`export_json`).
  * JSON Lines (`export_jsonl`) and CSV (`export_csv`).

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

## Quickstart

### 1. CSS Selector Queries & DOM Traversal

```mozhi
import crawler

let html = "<div id='catalog'><article class='item featured' data-id='101'><h2>Mozhi Book</h2><span class='price'>$29</span></article></div>"
let doc = crawler.parse_html(html)

# Query elements
let card = crawler.query_selector(doc, "article.item.featured", null)
let title_nid = crawler.query_selector(doc, "h2", card)

echo("Title: " + crawler.node_text(doc, title_nid))
echo("Item ID: " + crawler.node_attr(doc, card, "data-id"))
```

### 2. Declarative Schema Extraction

```mozhi
import crawler

let schema = {
    "site_title": "header h1",
    "products": {
        "selector": "article.product-item",
        "multiple": true,
        "fields": {
            "id": { "attribute": "data-id" },
            "name": "h2.prod-title",
            "price": "span.prod-price",
            "link": { "selector": "a.buy", "attribute": "href" }
        }
    }
}

let page = crawler.scrape("http://127.0.0.1:8089/products")
let data = crawler.extract(page.doc, schema)

for prod in data.products {
    echo(prod.name + " (" + prod.price + ") -> " + prod.link)
}
```

### 3. Multi-Page Web Crawling with Live JSON Result

```mozhi
import crawler

let results = crawler.crawl({
    url: "http://127.0.0.1:8089/",
    max_pages: 10,
    max_depth: 2,
    same_domain: true,
    robots_txt: true,
    export_json_path: "live_result.json"
})

echo("Total crawled pages: " + string(results.total_pages))
```

### 4. Categorized Terminal Tables

```mozhi
import crawler

let page = crawler.scrape("http://127.0.0.1:8089/media-showcase")
crawler.print_tables(page)
```

---

## API Reference

### DOM Tree & Selectors
* `crawler.parse_html(html)`: Parses raw HTML into a node-keyed DOM tree.
* `crawler.query_selector(doc, selector, context_nid)`: Returns the first matching node ID or `null`.
* `crawler.query_selector_all(doc, selector, context_nid)`: Returns array of matching node IDs.
* `crawler.matches_selector(doc, nid, selector)`: Checks whether a specific node matches a selector.
* `crawler.node_text(doc, nid)`: Recursively extracts plain text content.
* `crawler.node_attr(doc, nid, attr_name)`: Retrieves an attribute value.
* `crawler.node_html(doc, nid)`: Serializes node outer HTML.
* `crawler.node_inner_html(doc, nid)`: Serializes node inner HTML.

### Structured Extraction
* `crawler.extract(doc_or_html, schema)`: Extracts data records based on a declarative schema map.
* `crawler.extract_metadata(doc_or_html)`: Extracts titles, descriptions, Open Graph, and Twitter metadata.
* `crawler.extract_json_ld(doc_or_html)`: Extracts and parses JSON-LD structured data scripts.
* `crawler.extract_tables(doc_or_html)`: Extracts HTML `<table>` elements into structured arrays.
* `crawler.extract_page_data(base_url, html)`: Full backward-compatible asset extraction (`txt`, `img`, `video`, `files`, `links`, `more_data`).

### Crawling & Scraping
* `crawler.scrape(url_or_options)`: Fetches, resolves redirects, validates SSRF security, and parses a single web page.
* `crawler.crawl(options)`: BFS recursive crawler with deduplication, depth limits, robots.txt, and live JSON streaming.
* `crawler.parse_robots_txt(content, target_ua)`: Parses robots.txt rules and directives.
* `crawler.is_robots_allowed(robots, path, target_ua)`: Evaluates path permissions against robots rules.
* `crawler.parse_sitemap(xml_content)`: Parses `<loc>` entries from XML sitemaps.

### URL & Security
* `crawler.parse_url(url)`: Parses URL into protocol, hostname, port, path, query, and fragment.
* `crawler.normalize_path(path)`: Resolves dot segments (`.` and `..`).
* `crawler.normalize_url(url)`: Canonicalizes URLs, removes default ports (80/443) and fragments.
* `crawler.resolve_url(base, relative)`: Resolves relative paths to absolute URLs with dot-segment normalization.
* `crawler.is_private_ip(host)`: Checks for loopback, RFC 1918 private subnets, and cloud metadata IPs.
* `crawler.validate_target_url(url, allow_private_ips)`: Validates protocol and SSRF constraints.

### Export Utilities
* `crawler.print_tables(page_data)`: Pretty-prints categorized ASCII tables to the console.
* `crawler.export_json(data, file_path)`: Writes formatted JSON to disk.
* `crawler.export_jsonl(records, file_path)`: Writes records to a JSON Lines (`.jsonl`) file.
* `crawler.export_csv(headers, rows, file_path)`: Writes records to a CSV file with quote escaping.

---

## Testing & Demos

Run the full test suite:

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
