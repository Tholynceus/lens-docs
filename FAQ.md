# FAQ

Answers to common questions.

## General

**Q: Is LENS free?**

A: Yes, forever. The extension is free. The API is free, no rate limits. The code is open source under MIT license. No hidden costs.

**Q: Does LENS require a wallet connection?**

A: No. LENS reads public on-chain data only, never asks for wallet access. You don't need to connect anything.

**Q: What chains does LENS support?**

A: Currently Base only. Support for other chains (Ethereum, Polygon, Arbitrum, etc) planned.

**Q: Can I use LENS for other platforms (Discord, Telegram, etc)?**

A: Currently Chrome extension works on X/Twitter only. The API is platform-agnostic (works with any handle/contract), so developers can build LENS for other platforms.

## Extension

**Q: Why does LENS only work on X?**

A: That's where most token devs hang out and where the bio detection is cleanest. Other platforms don't have the same density of wallet addresses in profiles.

**Q: Can I use LENS on mobile?**

A: Not yet. Chrome extension only works on desktop. Mobile Safari/Chrome don't support extensions. Working on it.

**Q: Can I customize the verdict card styling?**

A: Yes. See [Extension Guide](./EXTENSION.md) for how to edit `src/lens.css` and rebuild.

**Q: Does LENS track me?**

A: No. All data stays local in `chrome.storage`. The only thing sent to the backend is the handle/contract you're scanning (no IP tracking, no user ID). Check the source code.

**Q: I got "Error reading wallet" on a profile. Why?**

A: Possible reasons:
1. Backend is slow or unreachable
2. Your Alchemy key is invalid or out of quota
3. The address in the bio isn't recognized as a valid wallet or contract
4. Try refreshing the page

## API

**Q: Do I need an API key to use the LENS API?**

A: No. The API is completely open. Call it from anywhere.

**Q: Can I hit the API from JavaScript in the browser?**

A: Yes. CORS is enabled, so you can call from any origin safely.

**Q: What if I hit the API a million times per second?**

A: There's no rate limiting, so technically you can, but please don't. The backend runs on Vercel's free tier. Abuse will get your IP softly rate-limited.

**Q: Can I use the API in production?**

A: Yes, it's designed for production use. 99.9% uptime SLA (Vercel). Data refreshes every 5 minutes via cron.

**Q: The API returned `found: false`. What does that mean?**

A: The handle/contract has no indexed launches yet. Either:
- The dev hasn't launched on Bankrbot/Base
- The cron job hasn't indexed it yet (wait 5 minutes)
- The handle is misspelled

**Q: Can I see the raw on-chain data instead of the verdict?**

A: Yes, the API returns all fields: devSold, feeShare, liquidity, linked, funder, deployer address, etc. The verdict is just a summary. Use the raw fields if you want to build custom scoring.

## Backend

**Q: How do I run the backend locally?**

A: See [Setup Guide](./SETUP.md). Clue: `npm install` → `.env` → `npm run dev`.

**Q: Can I deploy the backend to my own servers?**

A: Yes. The code is open source. You'll need your own Supabase, Alchemy, and a hosting provider (or stick with Vercel).

**Q: What if Vercel goes down?**

A: All data is in Supabase, so it persists. You can redeploy to another host (AWS, GCP, etc) in minutes. The code is portable.

**Q: The cron job isn't running. How do I manually trigger it?**

A: Call:
```bash
curl "https://your-url.vercel.app/api/index-launches?secret=YOUR_CRON_SECRET"
```

Replace `YOUR_CRON_SECRET` with the secret from your env vars. This force-runs the launch indexing.

## Data & Accuracy

**Q: How accurate is the verdict?**

A: Very accurate for detecting obvious rugs (dev dump + unlocked liquidity + high fees + linked scammers). Less accurate for predatory devs who hide their moves. Always do your own research.

**Q: What if a dev claims they're CLEAR but the verdict is STOP?**

A: Trust the chain, not the claim. The on-chain data doesn't lie. LENS reads what actually happened, not what was promised.

**Q: Can devs fake the on-chain data?**

A: No. LENS reads the immutable blockchain. You can't fake a transaction history, token supply, or wallet address.

**Q: Is Bankrbot data reliable?**

A: Yes. Bankrbot terminal is the authoritative source for Base token launches. LENS indexes Bankrbot data directly.

## Community & Contributing

**Q: Can I submit a PR to fix a bug?**

A: Yes! Issues and PRs are welcome on any of the repos (lens-extension, Lens, lens-skill-pack).

**Q: Can I build a custom skill using LENS?**

A: Yes, absolutely. See [Skills Guide](./SKILLS.md). Share it and we'll list it here.

**Q: Can I use LENS in a commercial product?**

A: Yes. MIT license means free to use for any purpose, including commercial. Just include the license notice.

**Q: Where can I ask for help?**

A: Open an issue on GitHub or @ us on X [@lnsx_io](https://x.com/lnsx_io).

## Troubleshooting

**Q: The extension won't load. Error: "Failed to load extension"**

A: Make sure:
- You're loading from the repo root (not a subfolder)
- You have both `manifest.json` and `src/` folder
- `manifest.json` is valid JSON (no trailing commas)

**Q: "CORS error" when calling from the browser**

A: The API should allow CORS. Check:
- Are you calling `https://` (not `http://`)?
- Is there a proxy in between that strips headers?
- Try the curl request to confirm the API itself works

**Q: Response is empty or just `{}`**

A: Likely means `found: false`. The handle/contract has no data. Try a known handle like `bankrbot`.

**Q: How do I report a bug?**

A: Open a GitHub issue with:
1. What you tried
2. What happened
3. What you expected
4. Steps to reproduce

---

Still stuck? @ [@lnsx_io](https://x.com/lnsx_io) on X and we'll help.
