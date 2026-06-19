# Skills Guide

How to use LENS as a skill in agent frameworks like AEON, and how to build custom skills.

## What is a Skill

A skill is a reusable unit of agent capability. In the AEON framework, skills are defined in SKILL.md files. The skill tells the agent:

- What this does (description)
- How to call it (input parameters)
- What it returns (output format)
- Examples (how to use it)

LENS is packaged as a skill so agents can understand it and use it autonomously.

## lens-scan Skill (Official)

The official LENS skill is ready to use. It's in the [lens-skill-pack](https://github.com/Tholynceus/lens-skill-pack) repo.

### Install

If using AEON:

```bash
./install-skill-pack Tholynceus/lens-skill-pack
```

This downloads all skills from the pack, including `lens-scan`.

### Use in AEON

Add to `aeon.yml`:

```yaml
skills:
  - name: lens-scan
    path: ./skills/lens-scan
```

Now AEON agents can call it:

```
@aeon scan @bankrbot
```

Agent sees the `lens-scan` skill definition and knows:
- Input: handle or contract
- Output: verdict (CLEAR/CAUTION/STOP), trust score, signals

### Use with Claude MCP

If running Claude with MCP servers:

```json
{
  "tools": [
    {
      "name": "lens_scan",
      "handler": "handle_lens_scan"
    }
  ]
}
```

Claude can then request reads:

```
Claude: "scan @exitliq_dev to see if it's safe"
Tool: Returns verdict STOP, trust 18, dev sold, etc
Claude: "High risk. Dev dumped supply and claims 95% of fees"
```

## Build Your Own Skill

Extend LENS with custom skills. Skill format is simple: a SKILL.md file.

### Skill.md Structure

```markdown
# skill-name

short description, one line

## Input

describe the inputs. List each parameter:
- `param1` — type, description
- `param2` — type, description

## Output

describe outputs:
- `field1` — type, meaning
- `field2` — type, meaning

## How to Use

Explain step by step

## Example

show a real example of calling it and the result
```

### Example: `lens-compare` Skill

Compare multiple handles by rug risk.

Create `skills/lens-compare/SKILL.md`:

```markdown
# lens-compare

rank multiple X handles by trust score, highest safety first

## Input

- `handles` (array of strings) — list of X handles to compare (e.g., ["@bankrbot", "@exitliq_dev", "@rugzn"])

## Output

Array of objects, sorted by trust descending:
- `handle` (string) — the X handle
- `verdict` (string) — CLEAR, CAUTION, STOP
- `trust` (number) — 0-100 score
- `reason` (string) — brief explanation

## How to Use

1. Provide a list of handles you're comparing
2. lens-compare calls lens-scan for each one
3. Results sorted by safety (highest first)
4. Use to pick the safest dev to follow

## Example

compare @bankrbot @exitliq_dev @rugzn

Output:
```
[
  { handle: "@bankrbot", verdict: "CLEAR", trust: 92, reason: "Established with no red flags" },
  { handle: "@rugzn", verdict: "CAUTION", trust: 45, reason: "Mixed signals, new account" },
  { handle: "@exitliq_dev", verdict: "STOP", trust: 18, reason: "Dev sold 95% of fees, shared funder" }
]
```
```

### Implement the Handler

In code (Node.js + Express example):

```javascript
app.post("/tools/lens-compare", async (req, res) => {
  const { handles } = req.body;
  
  // Call lens-scan for each handle
  const results = await Promise.all(
    handles.map(async (h) => {
      const scan = await fetch(
        `https://lens-liard.vercel.app/api/lookup?username=${h}`
      ).then(r => r.json());
      
      return {
        handle: h,
        verdict: scan.verdict,
        trust: scan.trust,
        reason: explainVerdict(scan)
      };
    })
  );
  
  // Sort by trust descending
  results.sort((a, b) => b.trust - a.trust);
  
  res.json(results);
});

function explainVerdict(scan) {
  if (scan.verdict === "CLEAR") {
    return "Established with no red flags";
  }
  if (scan.devSold && scan.feeShare > "50%") {
    return `Dev sold, claims ${scan.feeShare} of fees`;
  }
  if (scan.linked.length > 2) {
    return `Linked to ${scan.linked.length} scammer accounts`;
  }
  return "Mixed signals worth reviewing";
}
```

### Register the Skill

If publishing to a skill pack:

1. Create `skills/lens-compare/SKILL.md` (the definition)
2. Create `skills/lens-compare/handler.js` (the code)
3. Add to your skill pack repo
4. Agents can now use it

## Popular Skill Ideas

**lens-network**
- Input: one handle
- Output: full network map of linked accounts + funding trail
- Use: understand the whole operation behind a dev

**lens-history**
- Input: handle
- Output: timeline of past launches, past verdicts, pattern trends
- Use: see if a dev's history is improving or repeating scams

**lens-report**
- Input: handle
- Output: shareable markdown report with all signals + explanation
- Use: share findings with community

**lens-alert**
- Input: list of handles to watch
- Output: daily alerts if any switch from CLEAR to CAUTION/STOP
- Use: monitoring dashboard

## Publishing Your Skill

1. Create the SKILL.md + handler code
2. Put in a public GitHub repo: `yourname/lens-skills` or similar
3. Announce it in the LENS community
4. Link from README so others can find it

Agents will auto-discover and use skills they find in repos.

---

Next: [FAQ](./FAQ.md)
