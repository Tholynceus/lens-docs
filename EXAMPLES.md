# Integration Examples

Copy-paste ready code to integrate LENS into your app.

## cURL

Simplest way to test

```bash
curl "https://lens-liard.vercel.app/api/lookup?username=bankrbot"
```

With contract address:

```bash
curl "https://lens-liard.vercel.app/api/lookup?contract=0x833589fcd6edb6e08f4c7c32d4f71b4dc5476cce"
```

## JavaScript

### Vanilla JS

```javascript
async function scanHandle(handle) {
  const res = await fetch(
    `https://lens-liard.vercel.app/api/lookup?username=${encodeURIComponent(handle)}`
  );
  const data = await res.json();
  
  if (!data.found) {
    console.log("No data found for", handle);
    return;
  }
  
  console.log(`${handle}: ${data.verdict} (trust ${data.trust})`);
  if (data.devSold) console.log("⚠️  Dev sold supply");
  if (data.linked.length) console.log("Linked:", data.linked.join(", "));
  
  return data;
}

scanHandle("bankrbot");
```

### React Hook

```jsx
import { useState, useEffect } from "react";

export function useLensVeridict(handle) {
  const [verdict, setVerdict] = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  useEffect(() => {
    if (!handle) return;
    
    setLoading(true);
    fetch(`https://lens-liard.vercel.app/api/lookup?username=${encodeURIComponent(handle)}`)
      .then(r => r.json())
      .then(data => {
        if (data.found) setVerdict(data);
        else setError("Not found");
      })
      .catch(e => setError(e.message))
      .finally(() => setLoading(false));
  }, [handle]);

  return { verdict, loading, error };
}

// Usage in component
export function ProfileCard({ handle }) {
  const { verdict, loading } = useLensVeridict(handle);
  
  if (loading) return <div>Scanning...</div>;
  if (!verdict) return <div>No verdict</div>;
  
  return (
    <div className={`verdict ${verdict.verdict.toLowerCase()}`}>
      <h3>{verdict.verdict}</h3>
      <p>Trust: {verdict.trust}</p>
      {verdict.devSold && <p>⚠️  Dev sold</p>}
    </div>
  );
}
```

### Node.js / Express

```javascript
import fetch from "node-fetch";

app.get("/scan/:handle", async (req, res) => {
  const { handle } = req.params;
  
  const apiRes = await fetch(
    `https://lens-liard.vercel.app/api/lookup?username=${encodeURIComponent(handle)}`
  );
  const data = await apiRes.json();
  
  if (!data.found) {
    return res.status(404).json({ error: "Not found" });
  }
  
  res.json(data);
});
```

## Python

### Basic Request

```python
import requests

def scan_handle(handle):
    url = f"https://lens-liard.vercel.app/api/lookup?username={handle}"
    resp = requests.get(url)
    data = resp.json()
    
    if not data.get("found"):
        print(f"No data for {handle}")
        return None
    
    print(f"{handle}: {data['verdict']} (trust {data['trust']})")
    if data.get("devSold"):
        print("⚠️  Dev sold supply")
    
    return data

scan_handle("bankrbot")
```

### Flask API

```python
from flask import Flask, jsonify, request
import requests

app = Flask(__name__)

@app.route("/api/scan/<handle>", methods=["GET"])
def scan(handle):
    url = f"https://lens-liard.vercel.app/api/lookup?username={handle}"
    resp = requests.get(url)
    data = resp.json()
    return jsonify(data)

if __name__ == "__main__":
    app.run(debug=True)
```

### Pandas / Data Analysis

```python
import pandas as pd
import requests

handles = ["bankrbot", "exitliq_dev", "rugzn"]
verdicts = []

for handle in handles:
    url = f"https://lens-liard.vercel.app/api/lookup?username={handle}"
    data = requests.get(url).json()
    if data.get("found"):
        verdicts.append({
            "handle": handle,
            "verdict": data.get("verdict"),
            "trust": data.get("trust"),
            "dev_sold": data.get("devSold"),
            "fee_share": data.get("feeShare")
        })

df = pd.DataFrame(verdicts)
print(df)
print(f"Average trust: {df['trust'].mean()}")
print(f"STOP verdicts: {len(df[df['verdict'] == 'STOP'])}")
```

## TypeScript

```typescript
interface LensVerdictResponse {
  verdict: "CLEAR" | "CAUTION" | "STOP";
  trust: number;
  devSold: boolean;
  feeShare: string;
  liquidity: "locked" | "unlocked";
  linked: string[];
  funder: string;
  tokens: number;
  deployer: string;
  found: boolean;
}

async function scanHandle(handle: string): Promise<LensVerdictResponse | null> {
  const url = new URL("https://lens-liard.vercel.app/api/lookup");
  url.searchParams.set("username", handle);
  
  const res = await fetch(url);
  const data: LensVerdictResponse = await res.json();
  
  if (!data.found) return null;
  return data;
}

// Usage
const verdict = await scanHandle("bankrbot");
if (verdict && verdict.verdict === "STOP") {
  console.log("High risk!");
}
```

## Claude MCP (Model Context Protocol)

Use LENS as a tool in Claude conversations or agents.

```json
{
  "tools": [
    {
      "name": "lens_scan",
      "description": "Read the deployer wallet behind an X profile, get rug-risk verdict",
      "inputSchema": {
        "type": "object",
        "properties": {
          "handle": {
            "type": "string",
            "description": "X handle (with or without @)"
          },
          "contract": {
            "type": "string",
            "description": "Base contract address (0x...)"
          }
        },
        "required": ["handle"]
      }
    }
  ]
}
```

Handler implementation:

```javascript
async function lensScanned({ handle, contract }) {
  const param = contract ? `contract=${contract}` : `username=${handle}`;
  const res = await fetch(`https://lens-liard.vercel.app/api/lookup?${param}`);
  const data = await res.json();
  
  if (!data.found) {
    return { error: `No data for ${handle || contract}` };
  }
  
  return data;
}
```

## AEON Skill

See [Skills Guide](./SKILLS.md) for full setup.

Quick reference:

```markdown
# lens-scan

scan any X handle or Base contract, get on-chain deployer intel + rug-risk verdict

## Input

- `target` — X handle (@bankrbot) or Base contract (0x...)

## Output

- `verdict` — CLEAR, CAUTION, STOP
- `trust` — 0-100 score
- Additional signals: devSold, feeShare, linked accounts, etc

## Example

scan @exitliq_dev
-> STOP, trust 18, dev sold, 95% fees, linked @rugzn @dumpct
```

---

Next: [Custom Skills](./SKILLS.md)
