# Authenticated Browsing

## Overview

Some sites (X/Twitter, banking, enterprise SaaS) require authentication. Plasmate supports cookie-based auth profiles that persist across sessions.

## How It Works

1. User obtains auth cookies from their browser (manually or via Plasmate browser extension)
2. Cookies are stored encrypted in `~/.plasmate/profiles/<domain>.enc`
3. Plasmate loads cookies into the HTTP client at session start
4. When cookies expire (401/403), Plasmate signals re-auth needed via AWP

## X/Twitter Specifics

X uses two critical cookies:

- **ct0**: CSRF token, rotates periodically but refreshes automatically on valid sessions
- **auth_token**: Session token, lasts weeks under normal usage

### Obtaining Cookies

**Option A - Browser dev tools:**
1. Log into x.com in your browser
2. Open DevTools > Application > Cookies > x.com
3. Copy `ct0` and `auth_token` values

**Option B - Plasmate browser extension:**
1. Install the Plasmate extension from Chrome Web Store
2. Navigate to x.com and log in
3. Click the Plasmate extension icon
4. Click "Copy Auth Cookies" - copies in CLI-ready format

### Storing Cookies

```bash
plasmate auth x.com --ct0 "abc123..." --auth-token "xyz789..."
```

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
plasmate auth example.com --cookies "session_id=abc; csrf=xyz"

# From a JSON file (exported from browser)
plasmate auth example.com --cookie-file cookies.json
```

## Security

- Cookies are encrypted at rest using the system keychain (macOS Keychain, Linux secret-service)
- Fallback: AES-256-GCM with a user-provided passphrase
- Profiles are per-domain, never shared across domains
- `plasmate auth list` shows stored profiles (domain + expiry, never cookie values)
- `plasmate auth revoke <domain>` deletes a stored profile
