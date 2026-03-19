---
name: plasmate
description: Browse the web via Plasmate, a fast headless browser engine for agents. Compiles HTML into a Semantic Object Model (SOM) - 50x faster than Chrome, 10x fewer tokens. Supports AWP (Agent Web Protocol) and CDP compatibility. Optional authenticated browsing uses locally encrypted cookie profiles that never leave the user's machine.
homepage: https://plasmate.app
metadata:
  {
    "openclaw":
      {
        "emoji": "⚡",
        "source": "https://github.com/plasmate-labs/plasmate",
        "license": "Apache-2.0",
        "privacy": "https://plasmate.app/privacy",
        "requires": { "bins": ["plasmate"] },
        "install":
          [
            {
              "id": "curl",
              "kind": "shell",
              "command": "curl -fsSL https://plasmate.app/install.sh | sh",
              "bins": ["plasmate"],
              "label": "Install Plasmate (shell script)",
            },
            {
              "id": "cargo",
              "kind": "shell",
              "command": "cargo install plasmate",
              "bins": ["plasmate"],
              "label": "Install Plasmate (cargo)",
            },
          ],
      },
  }
---

# Plasmate - Browser Engine for Agents

Plasmate compiles HTML into a Semantic Object Model (SOM). 50x faster than Chrome, 10x fewer tokens.

- **Docs**: https://docs.plasmate.app
- **Source**: https://github.com/plasmate-labs/plasmate (Apache 2.0, fully auditable)
- **Privacy**: https://plasmate.app/privacy

## Security and Privacy

Plasmate is open source (Apache 2.0) and runs entirely on the user's machine. No data is sent to Plasmate Labs or any third party.

- **No telemetry, analytics, or cloud services** - everything runs locally
- **Auth profiles are encrypted** with AES-256-GCM before being written to disk
- **Encryption key is auto-generated** at `~/.plasmate/master.key` (owner-only permissions, chmod 0600)
- **The cookie bridge listens only on localhost** (127.0.0.1:9271) - never accessible from the network
- **Cookie sharing is always user-initiated** - the user must click "Push to Plasmate" in the extension; the agent cannot extract cookies on its own
- **Users can revoke any stored profile** at any time with `plasmate auth revoke <domain>`
- **All code is auditable** - engine, extension, and this skill are open source

## Install

```bash
# Install script (inspect source: https://plasmate.app/install.sh)
curl -fsSL https://plasmate.app/install.sh | sh

# Or build from source
git clone https://github.com/plasmate-labs/plasmate && cd plasmate && cargo build --release
```

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

## Authenticated Browsing (Optional)

Plasmate can optionally use stored cookie profiles to browse sites the user is logged into. This is entirely opt-in and user-controlled.

### How it works

1. The user logs into a site normally in their browser
2. The user opens the Plasmate extension and clicks "Push to Plasmate" (user-initiated, never automatic)
3. Cookies are sent to the local bridge server on localhost only
4. Plasmate encrypts and stores them on disk (AES-256-GCM)
5. The agent can then fetch that site with the stored profile

The agent cannot access cookies without the user explicitly sharing them.

### Using stored profiles

```bash
# Browse with a stored profile
plasmate fetch --profile x.com https://x.com/home

# Serve with a profile loaded
plasmate serve --profile github.com --port 9222
```

### Cookie bridge (local extension integration)

```bash
# Start the local bridge server
plasmate auth serve
# Listens on 127.0.0.1:9271 (localhost only)
```

### Managing profiles

```bash
# List stored profiles with expiry status
plasmate auth list
# ✓ x.com (2 cookies) - valid
# ⚠ github.com (2 cookies) - expires soon
# ✗ linkedin.com (2 cookies) - expired

# Detailed info for a domain
plasmate auth info x.com

# Revoke (delete) a stored profile
plasmate auth revoke x.com
```

### Expiry tracking

Cookies include expiration dates. Plasmate tracks these and:
- Shows ✓ valid, ⚠ expires soon (<24h), ✗ expired in `auth list`
- Skips expired cookies automatically when loading profiles
- Surfaces re-auth prompts through AWP when 401/403 is detected

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

When a user asks you to browse a site that requires login and no profile exists:

### Step 1: Install Plasmate if needed
```bash
which plasmate || curl -fsSL https://plasmate.app/install.sh | sh
```

### Step 2: Check for existing profile
```bash
plasmate auth list
# If domain shows ✓, skip to browsing
```

### Step 3: Ask the user to share their login

Start the bridge, then guide the user:

```bash
plasmate auth serve &
```

Tell the user:
> "I need access to your [site] account. Please:
> 1. Install the Plasmate extension (from Chrome Web Store or github.com/plasmate-labs/plasmate-extension)
> 2. Go to [site] and make sure you're logged in
> 3. Click the Plasmate icon in your toolbar and hit 'Push to Plasmate'
>
> I'm waiting for it now."

```bash
# Wait for the user to share cookies (blocks up to 120s)
curl -s "http://127.0.0.1:9271/api/wait?domain=x.com&timeout=120"
```

### Step 4: Browse authenticated
```bash
plasmate fetch --profile x.com https://x.com/notifications
```

### Key points for agents
- The `/api/wait` endpoint blocks until cookies arrive - no polling loop needed
- If the user has already shared cookies before, skip straight to browsing
- If wait times out, ask the user to try again
- The bridge only needs to run during the cookie sharing step; browsing uses stored profiles
- Always explain what you're doing and why you need access

## When to Use Plasmate vs Browser Tool

- **Plasmate**: Speed-critical scraping, batch page processing, token-sensitive extraction, structured data, authenticated browsing of known sites
- **Browser tool**: Visual rendering needed, screenshots, complex JS SPAs requiring full Chrome engine, pixel-level interaction
