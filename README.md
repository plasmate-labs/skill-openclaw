<p align="center">
  <img src="plasmate-mark.png" alt="Plasmate" width="80" />
</p>

<h1 align="center">Plasmate Skill for OpenClaw</h1>

<p align="center">
  Browse the web from your <a href="https://openclaw.ai">OpenClaw</a> agent via <a href="https://plasmate.app">Plasmate</a>.<br/>
  Fast. Structured. Token-efficient.
</p>

<p align="center">
  <a href="https://plasmate.app">Website</a> &middot;
  <a href="https://docs.plasmate.app">Docs</a> &middot;
  <a href="https://github.com/plasmate-labs/plasmate">Engine</a>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/license-MPL--2.0-blue" alt="License" />
  <img src="https://img.shields.io/badge/platform-OpenClaw-orange" alt="OpenClaw" />
  <img src="https://img.shields.io/badge/protocol-AWP-brightgreen" alt="AWP" />
</p>

---

An [OpenClaw](https://openclaw.ai) skill that gives agents web browsing capabilities via Plasmate's Semantic Object Model. Instead of raw HTML (tens of thousands of tokens), your agent gets structured JSON that's 10-800x smaller.

## Install

```bash
clawhub install plasmate
```

Or clone this repo into your OpenClaw skills directory.

## What's Included

```
plasmate/
├── SKILL.md              # Skill definition and usage guide
├── scripts/
│   └── awp-browse.py     # AWP client for agent web interactions
└── references/
    └── auth-flow.md      # Authenticated browsing documentation
```

## Quick Start

### Prerequisites

Install Plasmate:

```bash
curl -fsSL https://plasmate.app/install.sh | sh
```

### Usage

Once the skill is installed, your OpenClaw agent can:

**Navigate and extract structured content:**
```bash
python3 scripts/awp-browse.py navigate "https://news.ycombinator.com"
```

**Click interactive elements:**
```bash
python3 scripts/awp-browse.py click "https://example.com" --ref "e12"
```

**Type into fields:**
```bash
python3 scripts/awp-browse.py type "https://example.com" --ref "e5" --text "search query"
```

**Extract structured data (JSON-LD, OpenGraph):**
```bash
python3 scripts/awp-browse.py extract "https://example.com"
```

The script auto-starts a Plasmate server if one isn't running.

## Authenticated Browsing

For sites requiring login (X, GitHub, enterprise SaaS):

1. Log in via your browser
2. Export cookies with the [Plasmate extension](https://github.com/plasmate-labs/plasmate-extension) or grab them from dev tools
3. Store them: `plasmate auth set x.com --ct0 <val> --auth-token <val>`
4. Agent browses with `--profile x.com`

See [references/auth-flow.md](plasmate/references/auth-flow.md) for details.

## Why Plasmate over Chrome/Puppeteer?

| | Plasmate | Chrome |
|---|---|---|
| Per page | 4-5 ms | 252 ms |
| Memory (100 pages) | ~30 MB | ~20 GB |
| Output tokens | SOM (10-800x smaller) | Raw HTML |
| Binary size | 43 MB | 300-500 MB |

Your agent reasons better with less noise, and your token budget goes further.

## Related

- [Plasmate](https://github.com/plasmate-labs/plasmate) - The browser engine for agents
- [Plasmate Extension](https://github.com/plasmate-labs/plasmate-extension) - Chrome extension for cookie export
- [OpenClaw](https://openclaw.ai) - The agent platform

## License

[MPL-2.0](LICENSE) - Modifications to these files must stay open. Use freely in any project.
