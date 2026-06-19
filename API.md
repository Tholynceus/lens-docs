# API Reference

The LENS API is public, read-only, and requires no authentication. One endpoint does everything.

## Base URL

```
https://lens-liard.vercel.app
```

Or custom domain if set up:
```
https://api.lnsx.io
```

## Lookup Endpoint

Read the deployer wallet behind any X profile or Base contract, get a rug-risk verdict.

### Request

```
GET /api/lookup?username=<handle_or_contract>
```

**Parameters**

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| `username` | string | yes* | X handle (with or without @), lowercase. Example: `bankrbot` or `@bankrbot` |
| `contract` | string | yes* | Base contract address starting with 0x. Example: `0x833589fcd6edb6e08f4c7c32d4f71b4dc5476cce` |

*Either `username` OR `contract`, not both. If you pass both, `username` takes priority.

### Response

Success (HTTP 200)

```json
{
  "verdict": "STOP",
  "trust": 18,
  "devSold": true,
  "feeShare": "95%",
  "liquidity": "unlocked",
  "linked": ["@rugzn", "@dumpct"],
  "funder": "shared",
  "tokens": 3,
  "deployer": "0x1234...abcd",
  "found": true
}
```

Not found (HTTP 200, `found: false`)

```json
{
  "found": false
}
```

### Response Fields

| Field | Type | Description |
| --- | --- | --- |
| `verdict` | string | `CLEAR`, `CAUTION`, or `STOP` |
| `trust` | number | 0-100, higher is safer |
| `devSold` | boolean | true if dev dumped supply to DEX |
| `feeShare` | string | percentage of fees dev captures |
| `liquidity` | string | `locked` or `unlocked` |
| `linked` | array | other X handles sharing the same deployer wallet |
| `funder` | string | `shared` if funder linked to other scammers, else `self` |
| `tokens` | number | total number of tokens deployed by this wallet |
| `deployer` | string | the on-chain wallet address (0x...) |
| `found` | boolean | true if data exists, false if no launches indexed yet |

### Verdict Logic

**CLEAR**
- No dev sells detected
- Low fee capture
- Account age reasonable
- No linked scammer networks

**CAUTION**
- Some red flags present but not critical
- Mixed signals worth a second look
- Could be legitimate but risky

**STOP**
- Dev sold supply to DEX routers
- High fee capture (>50%)
- Linked to known scammer wallets
- Suspicious account age or behavior

## Examples

### Lookup by X handle

```bash
curl "https://lens-liard.vercel.app/api/lookup?username=bankrbot"
```

Response:
```json
{
  "verdict": "CLEAR",
  "trust": 92,
  "devSold": false,
  "feeShare": "2%",
  "liquidity": "locked",
  "linked": [],
  "tokens": 450,
  "deployer": "0x1234...",
  "found": true
}
```

### Lookup by Base contract

```bash
curl "https://lens-liard.vercel.app/api/lookup?contract=0x833589fcd6edb6e08f4c7c32d4f71b4dc5476cce"
```

### Handle not found

```bash
curl "https://lens-liard.vercel.app/api/lookup?username=randomnonexistent"
```

Response:
```json
{
  "found": false
}
```

## Rate Limits

No rate limiting on the public API. Call freely.

## CORS

The API allows CORS from any origin. Safe to call from browsers.

## Error Handling

All errors return HTTP 200 with `found: false` or an error message. No 404s or 500s (by design, to keep the API stable).

```json
{
  "error": "invalid contract address",
  "found": false
}
```

If you hit an actual error (unlikely), check:
1. Is the username/contract valid?
2. Is the backend online? Check status
3. Does the handle have any launches indexed yet? (Cron may need a few minutes to run)

## Performance

- **Typical response time**: 50-200ms
- **Availability**: 99.9% (Vercel SLA)
- **Data freshness**: Updated every 5 minutes (cron job)

---

Next: [Integration Examples](./EXAMPLES.md)
