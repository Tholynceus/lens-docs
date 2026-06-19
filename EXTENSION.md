# Extension Guide

Complete guide to the LENS Chrome extension. How it works, how to customize it, how to deploy your own version.

## Architecture

The extension injects on-chain intelligence directly into X profiles using 3 main components:

### Content Script (`src/content.js`)

Runs on every page load at x.com. Responsibilities:

- Detects wallet addresses, contract addresses, GitHub links in profile bio (regex + DOM parsing)
- Calls the backend API `/api/lookup` for each detected target
- Injects a card into the profile with the verdict
- Listens for user interactions (Smart Followers click-through, settings, etc)

### Background Service Worker (`src/background.js`)

Long-lived process that handles:

- API requests (proxies `/api/lookup` calls)
- Resolves deployer wallets via Alchemy RPC (on-chain queries)
- Stores user config in `chrome.storage.local` (API URLs, keys, settings)
- Communicates with content script via `chrome.runtime.sendMessage`

### Popup (`popup.html` + `src/popup.js`)

User-facing settings panel. Allows:

- View/change `LENS_API_URL` (point to custom backend)
- Set optional `ALCHEMY_KEY` (use your own instead of proxy)
- Set optional `GITHUB_TOKEN` (for GitHub Intel read)
- Smart Followers list (recorded targets)
- Toggle features on/off

## Manifest

The extension declares itself via `manifest.json`:

```json
{
  "manifest_version": 3,
  "name": "LENS",
  "version": "2.27.66",
  "permissions": ["storage", "scripting", "webRequest"],
  "host_permissions": ["https://x.com/*"],
  "background": {
    "service_worker": "src/background.js"
  },
  "content_scripts": [
    {
      "matches": ["https://x.com/*"],
      "js": ["src/content.js"],
      "css": ["src/lens.css"]
    }
  ],
  "action": {
    "default_popup": "popup.html",
    "default_icons": { "128": "icons/icon128.png" }
  }
}
```

Key points:

- `manifest_version: 3` — required for Chrome 88+
- `host_permissions: ["https://x.com/*"]` — only reads x.com, nothing else
- No `key` field — required for Chrome Web Store, prevents ID changes

## How to Customize

### 1. Change the API URL

In `src/background.js` or `popup.js`, look for:

```javascript
const DEFAULTS = {
  LENS_API_URL: "https://lens-liard.vercel.app"
};
```

Change to your own backend URL:

```javascript
LENS_API_URL: "https://api.yourdomain.com"
```

### 2. Change the verdict card styling

Open `src/lens.css`. The card is styled with class `.lens-card-root`. Modify colors, fonts, positioning:

```css
.lens-card-root {
  background: #1a1a1a;
  border: 1px solid #2952e3;
  border-radius: 12px;
  padding: 12px;
  font-family: 'Inter', sans-serif;
}
```

### 3. Add a new feature (e.g., tooltip on hover)

In `src/content.js`, find the function `renderCard()`. Add DOM listeners:

```javascript
card.addEventListener('mouseenter', () => {
  card.style.boxShadow = '0 0 20px #2952e3';
});
```

### 4. Change which signals to display

In `src/content.js`, the verdict card shows: verdict, trust, devSold, feeShare, liquidity, linked, funder.

To hide or reorder, find the function `buildCardHTML()` and modify the template.

### 5. Build your own version

1. Clone the repo:
   ```bash
   git clone https://github.com/Tholynceus/lens-extension
   cd lens-extension
   ```

2. Customize as above

3. Load into Chrome:
   - Open `chrome://extensions`
   - Enable Developer Mode (top right)
   - Click "Load unpacked"
   - Select your project folder
   - Done! The extension is now running locally

4. To build a release:
   ```bash
   # Zip all files (not the folder)
   zip -r lens-extension.zip src/ popup.html manifest.json icons/
   ```

   Upload to Chrome Web Store or distribute as .zip

## Configuration

Users can configure the extension in the popup:

| Setting | Default | Purpose |
| --- | --- | --- |
| `LENS_API_URL` | `https://lens-liard.vercel.app` | Backend API endpoint |
| `ALCHEMY_KEY` | (optional) | User's own Alchemy key (uses proxy if empty) |
| `GITHUB_TOKEN` | (optional) | GitHub API token for GitHub Intel |

All stored in `chrome.storage.local` (encrypted, local to user).

## Troubleshooting

### Extension not showing verdict card

1. Check that you're on x.com (works only there)
2. Open DevTools (F12) → Console tab
3. Look for errors (red text)
4. Common issues:
   - Wrong API URL → check popup settings
   - API URL is unreachable → check backend status
   - Profile bio doesn't have address/handle → extension only works if bio has detected text

### Card shows "Error reading wallet"

- Alchemy key invalid or out of quota → set a valid key in popup or check Alchemy dashboard
- Backend is slow → retry, it usually works second try

### Changes to code not showing up

- Hard refresh x.com (`Ctrl+Shift+R`)
- Remove + re-add the extension (`chrome://extensions` → trash icon → Load unpacked again)
- If running locally with `npm run dev`, make sure you're hot-reloading

## Performance Notes

- Content script runs on every x.com page load (lightweight, ~2ms)
- API calls are debounced (don't spam backend if user scrolls fast)
- Card is injected once per profile visit (cached in memory)
- No persistent internet required once page loads (reads from cache)

## Security

- No wallet connection required (read-only)
- No private keys stored or transmitted
- Reads only x.com domain (via host_permissions)
- All API calls are over HTTPS
- Chrome isolates extension in sandbox

---

Next: [Skills Guide](./SKILLS.md)
