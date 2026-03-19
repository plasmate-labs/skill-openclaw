# Authenticated Browsing

## Overview

Some sites (X/Twitter, GitHub, LinkedIn, Instagram) require authentication. Plasmate supports encrypted cookie profiles that persist across sessions.

## How It Works

1. User logs into a site normally in Chrome
2. The Plasmate Chrome extension captures auth cookies
3. Cookies are pushed to the local bridge server (or copied as a CLI command)
4. Cookies are encrypted with AES-256-GCM and stored in `~/.plasmate/profiles/`
5. Plasmate loads cookies into the HTTP client at session start
6. When cookies expire (401/403), Plasmate signals re-auth needed via AWP

## Bridge Server (Recommended)

The bridge server lets the Chrome extension push cookies directly:

```bash
plasmate auth serve
# Starts on 127.0.0.1:9271 (localhost only)
```

Endpoints:
- `GET /api/status` - returns version and list of stored profile domains
- `POST /api/cookies` - accepts `{domain, cookies, expiry}` JSON, stores encrypted

The Chrome extension auto-detects the bridge (green dot = online, gray = offline with clipboard fallback).

## X/Twitter Specifics

X uses two critical cookies:

- **ct0**: CSRF token, rotates periodically but refreshes automatically on valid sessions
- **auth_token**: Session token, lasts weeks under normal usage

### Obtaining Cookies

**Option A - Chrome Extension (recommended):**
1. Install the Plasmate extension (github.com/plasmate-labs/plasmate-extension)
2. Start the bridge: `plasmate auth serve`
3. Navigate to x.com and log in
4. Click the Plasmate extension icon
5. ct0 and auth_token are auto-selected
6. Click "Push to Plasmate"

**Option B - Manual via dev tools:**
1. Log into x.com in your browser
2. Open DevTools > Application > Cookies > x.com
3. Copy `ct0` and `auth_token` values
4. Run: `plasmate auth set x.com --ct0 "..." --auth-token "..."`

### Using in Agent Sessions

```bash
# Server mode with profile
plasmate serve --profile x.com

# One-shot fetch with profile
plasmate fetch --profile x.com https://x.com/home
```

## General Cookie Auth

For any site, store arbitrary cookies:

```bash
# From a cookie string
plasmate auth set example.com --cookies "session_id=abc; csrf=xyz"
```

## Cookie Expiry

The extension sends each cookie's `expirationDate` from Chrome's API. Plasmate tracks these:

```bash
plasmate auth list
# ✓ x.com (2 cookies) - valid
# ⚠ github.com (2 cookies) - expires soon (<24h)
# ✗ linkedin.com (2 cookies) - expired

plasmate auth info x.com
# Domain:      x.com
# Cookies:     2
# Encrypted:   yes
# Fingerprint: a3b2c1...
#   ct0 - ✓ valid (expires in 14 days)
#   auth_token - ✓ valid (expires in 30 days)
```

When loading profiles, expired cookies are automatically skipped with a warning.

## Security

- **Encryption**: AES-256-GCM with auto-generated 256-bit master key
- **Key storage**: `~/.plasmate/master.key` with chmod 0600 (owner-only)
- **Profile storage**: `~/.plasmate/profiles/<domain>.json` (encrypted contents)
- **Bridge**: localhost-only (127.0.0.1:9271), CORS restricted to chrome-extension:// origins
- **Auto-migration**: legacy plaintext profiles are re-encrypted on first read
- **Revocation**: `plasmate auth revoke <domain>` permanently deletes a profile
