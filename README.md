# crawler — Mozhi Web Crawler & Asset Extractor

A full-spectrum web crawler and structured asset extractor written **100% in pure Mozhi**. Extracts categorized web data into `txt`, `img`, `video`, `files`, `links`, and `more data` with **zero database dependencies**.

## Features

* **Zero external dependencies**: Built purely on native Mozhi standard modules (`http`, `json`).
* **No Database**: Pure in-memory streaming and live structured JSON output.
* **Categorized Extractions**:
  * `txt`: Titles, meta descriptions, authors, OpenGraph tags, headings (`<h1>`-`<h3>`), and text paragraphs.
  * `img`: Images with absolute URLs, alt text, formats (PNG, JPG, SVG, WEBP), and dimensions.
  * `video`: `<video>` tags, `<source>` streams, embedded players (YouTube, Vimeo), and direct media links (`.mp4`, `.webm`, `.mov`, `.mp3`).
  * `files`: Downloadable files and documents (`.pdf`, `.zip`, `.csv`, `.docx`, `.tar.gz`, `.txt`, `.xlsx`).
  * `links`: Internal and external hyperlinks with anchor text.
  * `more data`: CSS stylesheets, JavaScript files, favicons, and canonical tags.
* **URL Resolution**: Full relative-to-absolute path resolution and fragment stripping.

## Install

```bash
mz install crawler
# or directly from this repository:
mz install @crossberry-in/mozhi-crawler
```

## Usage

```mozhi
import crawler

# 1. Scrape a single web page:
let page = crawler.scrape("http://127.0.0.1:8089/media-showcase")
echo("Title: " + page.txt.title)
echo("Images: " + string(page.img.len()))
echo("Files: " + string(page.files.len()))

# 2. Multi-page recursive crawl:
let crawl_results = crawler.crawl({
    url: "http://127.0.0.1:8089/",
    max_pages: 5,
    max_depth: 2,
    same_domain: true
})
echo("Crawled pages: " + string(crawl_results.total_pages))
```

## Develop

```bash
mozhi test                 # run tests/
mozhi run examples/demo.mz # run the demo
```

## Release

Releases are automated: push a `v*` tag and the GitHub Actions workflow attaches `crawler.mz.pkg.zip` to the release.

```bash
git tag v1.0.0 && git push origin v1.0.0
```
