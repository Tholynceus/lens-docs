<div align="center">

# LENS Documentation

### read the wallet behind any handle, then build on top of it

Complete guide to understanding, deploying, and integrating LENS. Everything you need to add on-chain intelligence to your product.

[![License MIT](https://img.shields.io/badge/license-MIT-3a3a3a?style=flat-square)](LICENSE)
[![Built on Base](https://img.shields.io/badge/built%20on-Base-0052FF?style=flat-square)](https://base.org)

[API Docs](./API.md) · [Setup Guide](./SETUP.md) · [Examples](./EXAMPLES.md) · [FAQ](./FAQ.md)

</div>

---

## What is LENS

LENS reads the deployer wallet behind any X profile or Base contract and returns a plain-language rug-risk verdict: `CLEAR`, `CAUTION`, or `STOP`.

The read includes dev sells, fee capture, liquidity status, linked accounts, funding trail, and on-chain history. It powers the Chrome extension, the public API, and the skill pack.

## Getting Started

**I just want to use the API** → [API Docs](./API.md)

**I want to run the backend locally** → [Setup Guide](./SETUP.md)

**I want to integrate LENS into my app** → [Examples](./EXAMPLES.md)

**I want to build a custom skill** → [Skills Guide](./SKILLS.md)

**I want to customize the extension** → [Extension Guide](./EXTENSION.md)

**I have a question** → [FAQ](./FAQ.md)

## The Stack

| Component | Purpose | Repo |
| --- | --- | --- |
| **Extension** | Chrome popup that injects verdicts into X profiles | [lens-extension](https://github.com/Tholynceus/lens-extension) |
| **Backend API** | Reads chains, returns verdicts, runs Bankrbot cron | [Lens](https://github.com/Tholynceus/Lens) |
| **Skill Pack** | Ready skill for agent frameworks (AEON, Claude MCP, etc) | [lens-skill-pack](https://github.com/Tholynceus/lens-skill-pack) |
| **Website** | Marketing, API docs, community | [lens-web](https://github.com/Tholynceus/lens-web) |

## How It Works

1. **User opens an X profile** → Extension detects wallet/contract addresses in bio
2. **Extension calls the API** → `GET /api/lookup?username=handle`
3. **Backend queries Supabase + Alchemy** → Fetches token launches, dev sells, linked accounts
4. **Returns a verdict** → `CLEAR`, `CAUTION`, `STOP` + detailed signals
5. **Card renders inline** → User sees the read right under the bio

The same API is available publicly for any app to use. No key required, read-only.

## Architecture

**Backend** (Node.js + Supabase + Alchemy)
- `/api/lookup` — main endpoint, resolves dev wallet + scores it
- `/api/index-launches` — cron job, fetches Bankrbot launches every 5 min
- `/api/verdict` — AI scoring logic
- `/api/linked-accounts` — sibling wallet detection

**Frontend** (Chrome extension, vanilla JS)
- Content script injects card into X DOM
- Popup settings stored in chrome.storage (local)
- Background service worker handles API calls

**Database** (Supabase PostgreSQL)
- `bankr_launches` — indexed token launches from Bankrbot
- Custom queries for linked accounts, dev history

## Key Concepts

**Verdict**
- `CLEAR` — on-chain history backs the pitch, no alarms
- `CAUTION` — mixed signals worth a second look
- `STOP` — dev dumped, liquidity unlocked, shared funder with scammer siblings

**Trust Score**
- 0-100, higher is safer
- Computed from dev sells, fee capture, account age, linked networks

**Linked Accounts**
- Other X handles sharing the same deployer wallet
- Detected via Supabase queries + on-chain transaction analysis

## Community

Found a bug or want to improve LENS? Issues and PRs welcome.

Questions? Check the [FAQ](./FAQ.md) or open an issue.

---

<div align="center">

built on Base · [lnsx.io](https://lnsx.io)

</div>
