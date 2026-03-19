---
name: plasmate
description: Browse the web via Plasmate, a fast headless browser engine for agents. Use when navigating websites, extracting structured content, clicking/typing on pages, scraping data, or automating web interactions. Plasmate returns a Semantic Object Model (SOM) instead of raw HTML - 10-800x smaller, optimized for LLM reasoning. Supports AWP (Agent Web Protocol) and CDP (Puppeteer/Playwright compatible). Includes authenticated browsing with encrypted cookie profiles for X, GitHub, LinkedIn, and any site.
---

# Plasmate - Browser Engine for Agents

Plasmate compiles HTML into a Semantic Object Model (SOM). 50x faster than Chrome, 10x fewer tokens.

- **Docs**: https://docs.plasmate.app
- **Install**: `curl -fsSL https://plasmate.app/install.sh | sh`
- **Extension**: https://github.com/plasmate-labs/plasmate-extension

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

### Authenticated Fetch

```bash
plasmate fetch --profile x.com https://x.com/home
```

### Server Mode

```bash
# AWP (recommended)
plasmate serve --protocol awp --port 9222

# With auth profile loaded
plasmate serve --profile x.com --port 9222

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

# Scroll
python3 scripts/awp-browse.py scroll "https://example.com" --direction down
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

## Authenticated Browsing

Plasmate supports encrypted cookie profiles for browsing sites that require login.

### How it works

1. User logs into a site in Chrome
2. The Plasmate Chrome extension grabs auth cookies
3. Cookies are pushed to the local bridge server (or copied as CLI command)
4. Plasmate stores them encrypted (AES-256-GCM) on disk
5. Agent browses the site authenticated using the stored profile

### Cookie bridge (extension integration)

```bash
# Start the bridge server (extension pushes cookies here)
plasmate auth serve
# Listens on 127.0.0.1:9271
# GET  /api/status  - version + stored profiles
# POST /api/cookies - store cookies from extension
```

### Manual cookie storage

```bash
# Store cookies from a string
plasmate auth set example.com --cookies "session_id=abc; csrf=xyz"

# X/Twitter shorthand
plasmate auth set x.com --ct0 "<ct0_value>" --auth-token "<auth_token_value>"
```

### Managing profiles

```bash
# List all stored profiles with expiry status
plasmate auth list
# ✓ x.com (2 cookies) - valid
# ⚠ github.com (2 cookies) - expires soon
# ✗ linkedin.com (2 cookies) - expired

# Detailed info for a domain
plasmate auth info x.com

# Revoke (delete) a profile
plasmate auth revoke x.com
```

### Expiry tracking

Cookies include expiration dates. Plasmate tracks these and:
- Shows ✓ valid, ⚠ expires soon (<24h), ✗ expired in `auth list`
- Skips expired cookies automatically when loading profiles
- Surfaces re-auth prompts through AWP when 401/403 is detected

### Security

- AES-256-GCM encryption at rest with auto-generated master key
- Master key at `~/.plasmate/master.key` (chmod 0600)
- Bridge only listens on localhost (127.0.0.1), never network
- Auto-migration of legacy plaintext profiles

See `references/auth-flow.md` for detailed auth documentation.

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

## Agent-Driven Auth Flow (Guided Setup)

When a user asks you to browse a site that requires login (e.g., "check my X mentions"),
and no profile exists for that domain, follow this flow:

### Step 1: Install plasmate if needed
```bash
which plasmate || curl -fsSL https://plasmate.app/install.sh | sh
```

### Step 2: Check if profile exists
```bash
plasmate auth list
# If domain shows up with ✓, skip to browsing
```

### Step 3: Start bridge and wait for cookies
```bash
# Start bridge in background
plasmate auth serve &

# Tell the user:
# "I need access to your [site] account. Two things:
#  1. Install the Plasmate extension: https://chromewebstore.google.com/detail/plasmate/[id]
#     (or from github.com/plasmate-labs/plasmate-extension)
#  2. Go to [site], make sure you're logged in,
#     click the Plasmate icon in your toolbar, and hit 'Push to Plasmate'
#
#  I'm watching for it now."

# Long-poll until cookies arrive (blocks up to 120s)
curl -s "http://127.0.0.1:9271/api/wait?domain=x.com&timeout=120"
# Returns: {"ok":true,"domain":"x.com","cookies":2}
# Or after timeout: {"ok":false,"error":"Timed out waiting for cookies"}
```

### Step 4: Browse authenticated
```bash
plasmate fetch --profile x.com https://x.com/notifications
```

### Key points for agents
- The `/api/wait` endpoint blocks until cookies arrive, no polling loop needed
- If the user has already pushed cookies before, skip straight to browsing
- If wait times out, ask the user to try again (cookies may not have been pushed)
- The bridge only needs to run during the cookie capture; browsing uses stored profiles

## When to Use Plasmate vs Browser Tool

- **Plasmate**: Speed-critical scraping, batch page processing, token-sensitive extraction, structured data, authenticated browsing of known sites
- **Browser tool**: Visual rendering needed, screenshots, complex JS SPAs requiring full Chrome engine, pixel-level interaction
