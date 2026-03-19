---
name: plasmate
description: Browse the web via Plasmate, a fast headless browser engine for agents. Use when navigating websites, extracting structured content, clicking/typing on pages, scraping data, or automating web interactions. Plasmate returns a Semantic Object Model (SOM) instead of raw HTML - 10-800x smaller, optimized for LLM reasoning. Supports AWP (Agent Web Protocol) and CDP (Puppeteer/Playwright compatible). Use instead of or alongside browser tools when speed, token cost, or memory matter.
---

# Plasmate - Browser Engine for Agents

Plasmate compiles HTML into a Semantic Object Model (SOM). 50x faster than Chrome, 10x fewer tokens.

- **Docs**: https://docs.plasmate.app
- **Install**: `curl -fsSL https://plasmate.app/install.sh | sh`

## Protocols

- **AWP** (native): 7 methods - navigate, snapshot, click, type, scroll, select, extract
- **CDP** (legacy bridge): Puppeteer/Playwright compatible on port 9222

Default to AWP. Use CDP only when existing Puppeteer/Playwright code needs reuse.

## Quick Start

### Fetch (one-shot, no server)

```bash
plasmate fetch <url>
```

Returns SOM JSON: regions, interactive elements with stable IDs, extracted content.

### Server Mode

```bash
# AWP (recommended)
plasmate serve --protocol awp --port 9222

# CDP (Puppeteer compatible)
plasmate serve --protocol cdp --port 9222
```

## AWP Usage (Python)

Run `scripts/awp-browse.py` for AWP interactions:

```bash
# Navigate and get SOM snapshot
python3 scripts/awp-browse.py navigate "https://example.com"

# Click an interactive element by ref ID
python3 scripts/awp-browse.py click "https://example.com" --ref "e12"

# Type into a field
python3 scripts/awp-browse.py type "https://example.com" --ref "e5" --text "search query"

# Extract structured data (JSON-LD, OpenGraph, tables)
python3 scripts/awp-browse.py extract "https://example.com"
```

## CDP Usage (Puppeteer)

When CDP is needed, connect Puppeteer to the running server:

```javascript
const browser = await puppeteer.connect({
  browserWSEndpoint: 'ws://127.0.0.1:9222'
});
const page = await browser.newPage();
await page.goto('https://example.com');
const content = await page.content();
```

## Authenticated Sessions

For sites requiring login (X, banking, enterprise SaaS):

```bash
# Store auth cookies (grab from browser dev tools or extension)
plasmate auth <domain> --cookies "name1=val1; name2=val2"

# X/Twitter specifically
plasmate auth x.com --ct0 "<ct0_value>" --auth-token "<auth_token_value>"

# Use stored profile in server mode
plasmate serve --profile x.com
```

Cookie profiles stored in `~/.plasmate/profiles/<domain>.enc` (encrypted at rest).

When auth tokens expire (401/403 detected), Plasmate surfaces a re-auth prompt through the protocol.

## SOM Output Structure

SOM is a structured JSON representation, NOT raw HTML. Key sections:

- **regions**: Semantic page areas (nav, main, article, sidebar)
- **interactive**: Clickable/typeable elements with stable ref IDs (e.g., `e1`, `e12`)
- **content**: Text content organized by region
- **structured_data**: JSON-LD, OpenGraph, microdata extracted automatically

Use ref IDs from `interactive` elements for click/type actions.

## Performance Expectations

| Metric | Plasmate | Chrome |
|--------|----------|--------|
| Per page | 4-5 ms | 252 ms |
| Memory (100 pages) | ~30 MB | ~20 GB |
| Output size | SOM (10-800x smaller) | Raw HTML |

## When to Use Plasmate vs Browser Tool

- **Plasmate**: Speed-critical scraping, batch page processing, token-sensitive extraction, structured data gathering
- **Browser tool**: Visual rendering needed, screenshots, complex JS SPAs requiring full Chrome engine, pixel-level interaction
