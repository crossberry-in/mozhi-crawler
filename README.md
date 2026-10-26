# crawler — Complete Multi-Agent LLM Research Engine & Web Spider for Mozhi

A complete web research, multi-agent synthesis, spider, DOM tree parser, CSS selector engine, and persistent storage subsystem written **100% in pure Mozhi**. Built exclusively on native Mozhi standard modules (`http`, `json`, `fs`, `db`), with **zero external runtime dependencies** (no Python, Node.js, Go, or PHP required) and **built-in SSRF protection**.

[![Release](https://img.shields.io/github/v/release/crossberry-in/mozhi-crawler?color=blue)](https://github.com/crossberry-in/mozhi-crawler/releases)
[![Docs](https://img.shields.io/badge/docs-GitHub%20Pages-1976d2)](https://crossberry-in.github.io/mozhi-crawler/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Pure Mozhi](https://img.shields.io/badge/Language-Pure%20Mozhi%203.2-brightgreen)](https://github.com/crossberry-in/mozhi-doc)

🌐 **Interactive Documentation Site**: [https://crossberry-in.github.io/mozhi-crawler/](https://crossberry-in.github.io/mozhi-crawler/)

---

## Architecture & Visual Diagrams

### 1. Multi-Agent Research Workflow
```
 +---------------------------------------------------------------------------------------------------+
 |                                   User Research Query                                             |
 +---------------------------------------------------------------------------------------------------+
                                                   │
                                                   ▼
 +---------------------------------------------------------------------------------------------------+
 |                              Agent 1 — Query Planner                                              |
 |         Identifies key facts to establish, generates search queries, and defines criteria         |
 +---------------------------------------------------------------------------------------------------+
                                                   │
                                                   ▼
 +---------------------------------------------------------------------------------------------------+
 |                             Agent 2 — Link Discovery Agent                                        |
 |    Discovers candidate URLs from seeds, sitemaps, and robots.txt; normalizes and ranks links      |
 +---------------------------------------------------------------------------------------------------+
                                                   │
                                                   ▼
 +---------------------------------------------------------------------------------------------------+
 |                             Agent 3 — Web Collection Agent                                        |
 |     Fetches candidate pages via existing crawler with SSRF checks, parses DOM, extracts tables   |
 +---------------------------------------------------------------------------------------------------+
                                                   │
                                                   ▼
 +---------------------------------------------------------------------------------------------------+
 |                            Agent 4 — Evidence Extraction Agent                                    |
 |       Extracts atomic factual claims from text, tables, and JSON-LD; preserves provenance URLs    |
 +---------------------------------------------------------------------------------------------------+
                                                   │
                                                   ▼
 +---------------------------------------------------------------------------------------------------+
 |                            Agent 5 — Research Verification Agent                                  |
 |           Corroborates claims across sources, evaluates confidence, and detects conflicts         |
 +---------------------------------------------------------------------------------------------------+
                                                   │
                                                   ▼
 +---------------------------------------------------------------------------------------------------+
 |                                Agent 6 — Final Synthesis Agent                                    |
 |      Combines verified evidence, generates summary answer, and structures validated JSON output   |
 +---------------------------------------------------------------------------------------------------+
                                                   │
                                                   ▼
 +---------------------------------------------------------------------------------------------------+
 |                        Validated Structured JSON & Terminal Tables                                |
 +---------------------------------------------------------------------------------------------------+
```

### 2. LLM Provider Management & Strict Free-Model Routing
```
                 [ Research Task: e.g. "reasoning" or "vision" ]
                                        │
                                        ▼
                         Is free_only: true active?
                                   /         \
                                [YES]        [NO]
                                 /             \
                  Filter models where         Allow all enabled
              billing_mode == "free"          models in catalog
              or pricing == 0                 (paid tier visible in stats)
                                 \             /
                                  ▼           ▼
                      [ Match Required Task Capabilities ]
                      (chat, reasoning, coding, vision)
                                        │
                         Candidates available?
                               /         \
                            [YES]        [NO]
                             /             \
                   Select best model     Return ERR_NO_FREE_MODELS_AVAILABLE
                   based on score        (Never silently charges paid tier)
                             │
                             ▼
                   [ Execute Completion with Fallback ]
                   (Transient 429/5xx -> Backoff & Retry or Fallback Provider;
                    Permanent 401/400 -> Fail immediately or switch provider)
```

---


### Free Kilo Code Models Support

Mozhi Crawler automatically identifies and routes Kilo Code free models under `free_only: true`:

| Model Identifier | Display Name | Tasks | Context Window | Price |
|---|---|---|---|---|
| `kilo-auto/free` | Kilo Auto Free Router | coding, reasoning, chat | 256,000 | **$0.00** |
| `poolside/laguna-s-2.1:free` | Poolside Laguna S 2.1 | coding, debugging | 262,144 | **$0.00** |
| `poolside/laguna-xs-2.1:free` | Poolside Laguna XS 2.1 | fast code generation | 262,144 | **$0.00** |
| `cohere/north-mini-code:free` | Cohere North Mini Code | code analysis, tests | 256,000 | **$0.00** |
| `recursive/flint-v1.0:free` | Recursive Flint | autonomous coding agent | 262,144 | **$0.00** |
| `reflection/beam-free` | Reflection Beam 1.0 | 1M reasoning & code | 1,048,576 | **$0.00** |
| `nvidia/nemotron-3.5-super-vl:free` | Nemotron 3.5 Super VL | multimodal vision & code | 1,000,000 | **$0.00** |

## Features

* **100% Native Pure Mozhi**: Zero external runtimes, Python, Node.js, PHP, or Go bridges. Built strictly on native standard modules (`http`, `json`, `fs`, `db`).
* **Multi-Agent Research Engine (`crawler.research`)**:
  * Six specialized agents: Query Planner, Link Discovery, Web Collector, Evidence Extractor, Verifier, and Final Synthesizer.
  * Preserves source provenance (`url`, `title`, `http_status`, `retrieved_at`) for every factual finding.
  * Shared resource budgets (`max_agent_calls`, `max_pages`, `max_total_requests`).
* **Custom LLM Provider Management (`crawler.init_provider_manager`)**:
  * Configurable OpenAI-compatible endpoints without source code modifications.
  * Initial support for BlockRun (`https://blockrun.ai/api/v1`) plus custom private/local endpoints (e.g. vLLM, Ollama, LocalAI).
  * Dynamic model discovery (`GET /models`).
  * Automatic credential redaction (`[REDACTED]`) across errors, diagnostics, and logs.
* **Strict Free-Tier Policy (`free_only: true`)**:
  * Enforces verified free metadata (`billing_mode: "free"` or `$0.00` pricing).
  * Excludes paid models and halts with `ERR_NO_FREE_MODELS_AVAILABLE` instead of silently charging paid models.
* **Automatic Fallback & Error Recovery**:
  * Classifies transient failures (HTTP 429 rate limit, 5xx server error) with exponential backoff and jitter.
  * Detects permanent errors (HTTP 401 unauthorized, 400 bad request) and fails safely or switches providers.
* **Vision & Image Understanding**:
  * Identifies vision-capable models (e.g. `nvidia/llama-3.2-11b-vision`, `nvidia/nemotron-3-nano-omni-30b-a3b-reasoning`).
  * Submits image URLs via OpenAI message format.
* **Advanced Web Spider (`crawler.spider`)**:
  * Breadth-first crawl queue with URL normalization, domain budgets, and external link discovery.
  * Dual storage backends: filesystem JSON cache (`.crawler_cache/`) and native SQLite database (`crawler.db`).
  * Offline mode: Zero network requests with explicit `ERR_OFFLINE_CACHE_MISS` error reporting.
* **SSRF Security**:
  * Validates destination IPs and schemes, blocking loopback (`127.0.0.0/8`, `::1`), private RFC 1918 subnets (`10/8`, `172.16/12`, `192.168/16`), and cloud metadata (`169.254.169.254`).

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


---

## Documentation AI Helper Chat & cURL Guide

The interactive documentation site at [https://crossberry-in.github.io/mozhi-crawler/](https://crossberry-in.github.io/mozhi-crawler/) includes a responsive **AI Docs Helper** that reads the complete package documentation to answer questions and replay past responses.

Developers can also query the documentation assistant or BlockRun models directly using standard `curl`:

```bash
curl -sS https://blockrun.ai/api/v1/chat/completions \
  -H 'Content-Type: application/json' \
  -H 'Authorization: Bearer YOUR_BLOCKRUN_API_KEY' \
  -d '{
    "model": "nvidia/gpt-oss-20b",
    "messages": [
      {
        "role": "system",
        "content": "You are the Mozhi Crawler Docs Assistant. Use the official documentation to answer questions about web crawling, spidering, and multi-agent research."
      },
      {
        "role": "user",
        "content": "How do I configure BlockRun provider in Mozhi crawler with free_only policy?"
      }
    ]
  }'
```

In pure Mozhi, call it natively with:

```mozhi
import crawler

let mgr = crawler.init_provider_manager({
    default_provider: "blockrun",
    free_only: true
})

let res = crawler.call_chat_completion(mgr, "blockrun", {
    model: "nvidia/gpt-oss-20b",
    messages: [
        { "role": "system", "content": "You are the Mozhi Crawler Docs Assistant." },
        { "role": "user", "content": "Explain how multi-agent research operates." }
    ]
})

echo(res.content)
```

## Quickstart Examples

### 1. Multi-Agent Research with BlockRun Provider
```mozhi
import crawler

let result = crawler.research({
    query: "What developer products and hardware does Mozhi offer, and what are their prices?",
    url: "https://example.com/products",
    provider_config: {
        default_provider: "blockrun",
        free_only: true
    },
    max_pages: 5,
    save_cache: true,
    cache_dir: ".crawler_cache"
})

echo("Research Completed: " + string(result.ok))
echo("Answer:\n" + result.answer)
echo("Sources Accessed:  " + string(result.sources.len()))

# Display formatted ASCII tables
crawler.print_research_tables(result)
```

### 2. Custom LLM Provider Configuration
```mozhi
import crawler

let mgr = crawler.init_provider_manager({
    default_provider: "blockrun",
    free_only: true,
    providers: [
        {
            id: "blockrun",
            name: "BlockRun",
            base_url: "https://blockrun.ai/api/v1",
            api_format: "openai",
            api_key_env: "BLOCKRUN_API_KEY",
            model_discovery: true,
            enabled: true
        },
        {
            id: "local-vllm",
            name: "Local vLLM Endpoint",
            base_url: "http://127.0.0.1:8000/v1",
            api_format: "openai",
            model_discovery: true,
            enabled: true
        }
    ]
})

# Test connection & discover models
let health = crawler.test_provider_connection(mgr, "blockrun")
echo("Connection status: " + string(health.status))
```

### 3. Offline Research Mode (Zero Network Calls)
```mozhi
import crawler

# Read previously cached research; issues 0 network calls:
let offline_res = crawler.research({
    query: "What developer products and hardware does Mozhi offer, and what are their prices?",
    offline: true,
    cache_dir: ".crawler_cache"
})

if offline_res.ok {
    echo("Cached answer:\n" + offline_res.answer)
} else {
    echo("Cache miss: " + offline_res.errors[0].message)
}
```

---

## JSON Output Schema Contract

Every research call returns a structured result adhering strictly to this verified schema:

```json
{
  "ok": true,
  "query": "What products does Mozhi store offer and what are their prices?",
  "answer": "Based on verified catalog documentation, Mozhi Compiler Pro costs $99.00, Mozhi Mechanical Keyboard is available for $149.00, and the Cotton Hoodie is priced at $45.00 [Source: http://127.0.0.1:8089/products].",
  "agents": [
    { "name": "query_planner", "status": "completed" },
    { "name": "link_discovery_agent", "status": "completed" },
    { "name": "web_collector_agent", "status": "completed" },
    { "name": "evidence_extraction_agent", "status": "completed" },
    { "name": "verification_agent", "status": "completed" },
    { "name": "synthesis_agent", "status": "completed" }
  ],
  "sources": [
    {
      "url": "http://127.0.0.1:8089/products",
      "title": "Product Catalog - Mozhi Store",
      "http_status": 200,
      "retrieved_at": 1729900000
    }
  ],
  "findings": [
    {
      "claim": "Mozhi Compiler Pro costs $99.00 and is an ahead-of-time compilation suite.",
      "source_urls": ["http://127.0.0.1:8089/products"],
      "confidence": "high",
      "evidence_type": "direct"
    }
  ],
  "conflicts": [],
  "errors": [],
  "stats": {
    "links_discovered": 4,
    "pages_fetched": 3,
    "pages_parsed": 3,
    "llm_calls": 2,
    "cache_hits": 0,
    "fallbacks": 0
  },
  "stopped_reason": "completed"
}
```

---

## API Reference

| Category | Function | Signature | Description |
|---|---|---|---|
| **Research** | `crawler.research` | `(config: map) -> map` | Orchestrates end-to-end multi-agent research workflow. |
| **Research** | `crawler.validate_research_result` | `(res: map) -> map` | Validates result against documented JSON contract schema. |
| **Research** | `crawler.print_research_tables` | `(res: map)` | Renders categorized terminal ASCII tables for research results. |
| **Provider** | `crawler.init_provider_manager` | `(cfg_or_path) -> map` | Initializes provider manager with default or custom endpoints. |
| **Provider** | `crawler.add_provider` | `(mgr, prov: map) -> map` | Adds or updates custom LLM provider endpoint. |
| **Provider** | `crawler.update_provider` | `(mgr, id, updates) -> map` | Updates fields for existing provider. |
| **Provider** | `crawler.remove_provider` | `(mgr, id: str) -> map` | Removes provider by ID. |
| **Provider** | `crawler.list_providers` | `(mgr) -> array` | Lists providers with credentials automatically redacted. |
| **Provider** | `crawler.discover_models` | `(mgr, id, store) -> array` | Fetches model catalog from provider `GET /models`. |
| **Routing** | `crawler.is_model_free` | `(model: map) -> bool` | Checks if model has verified free tier pricing. |
| **Routing** | `crawler.filter_free_models` | `(models: array) -> array` | Returns subset of models meeting free criteria. |
| **Routing** | `crawler.select_model_for_task` | `(mgr, id, task, vis)` | Selects best model for task honoring free-only policy. |
| **Routing** | `crawler.call_chat_completion_with_fallback` | `(mgr, opts: map) -> map` | Calls chat completion with retry, backoff, and fallback. |
| **Evidence** | `crawler.init_evidence_manager` | `() -> map` | Initializes atomic claim store and provenance tracker. |
| **Evidence** | `crawler.add_evidence` | `(em, claim, url, title, conf, type, snip)` | Adds evidence with source provenance and corroboration. |
| **Spider** | `crawler.spider` | `(config: map) -> map` | BFS spider with budgets, limits, and link graph. |
| **DOM** | `crawler.parse_html` | `(html: str) -> map` | Parses HTML into DOM tree. |
| **DOM** | `crawler.query_selector` | `(doc, sel, ctx) -> int` | First matching node ID. |
| **DOM** | `crawler.query_selector_all` | `(doc, sel, ctx) -> array` | All matching node IDs. |
| **Security** | `crawler.validate_target_url` | `(url, allow_priv) -> map` | Validates scheme, loopback, private subnets, and SSRF. |
| **Storage** | `crawler.init_storage` | `(opts: map) -> map` | Initializes JSON or SQLite persistent cache. |

---

## Testing

Run all 3 test suites:

```bash
mozhi test tests/
```

Expected output:

```
Running 3 test file(s) from tests/
  tests//crawler.test.mz                   PASS
  tests//research.test.mz                  PASS
  tests//spider.test.mz                    PASS

3 passed, 0 failed
```

Run the showcase demo:

```bash
mozhi run examples/demo.mz
```

---

## License

MIT © [Crossberry](https://github.com/crossberry-in)
