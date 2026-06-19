# Setup Guide

Complete walkthrough to run the LENS backend locally or deploy to production.

## Prerequisites

- Node.js 18+ (Vercel requires 18+)
- A Supabase account (supabase.com, free tier is fine)
- An Alchemy account (alchemy.com, free tier works)
- Vercel account (optional, for production deploy)
- Cloudflare account with your domain (optional, for custom domains)

## Local Setup

### 1. Clone the repo

```bash
git clone https://github.com/Tholynceus/Lens
cd Lens
```

### 2. Create Supabase project

1. Go to [supabase.com](https://supabase.com)
2. Create a new project (pick any region)
3. Go to SQL Editor
4. Open the SQL file from the repo: `schema.sql`
5. Run it (this creates the `bankr_launches` table + indexes)

Save these credentials (you'll need them):
- `LENS_SUPABASE_URL` — found in Project Settings > API
- `LENS_SUPABASE_SERVICE_KEY` — also in API section (service_role key, not the anon key)
- `LENS_SUPABASE_ANON_KEY` — the public anon key

### 3. Get Alchemy key

1. Go to [alchemy.com](https://alchemy.com)
2. Create an app on the Base chain
3. Copy your API key → this is `LENS_ALCHEMY_KEY`

### 4. Create .env file

In the project root, create `.env` (git will ignore it, listed in .gitignore):

```
LENS_SUPABASE_URL=https://xxxxx.supabase.co
LENS_SUPABASE_SERVICE_KEY=eyJxxx...
LENS_SUPABASE_ANON_KEY=eyJxxx...
LENS_ALCHEMY_KEY=your-alchemy-key
CRON_SECRET=any-random-string-you-make-up
```

CRON_SECRET is just a password you invent to protect the `/api/index-launches` cron job (so randoms can't spam it).

### 5. Install dependencies

```bash
npm install
```

### 6. Test locally

```bash
npm run dev
```

This starts a local server at `http://localhost:3000`. Test the API:

```bash
curl "http://localhost:3000/api/lookup?username=bankrbot"
```

You should get back a JSON response with verdict, trust, devSold, etc.

## Production Deploy (Vercel)

### 1. Push to GitHub

Make sure your repo is on GitHub (public or private is fine).

### 2. Connect to Vercel

1. Go to [vercel.com](https://vercel.com)
2. Click "Add New" → "Project"
3. Import your GitHub repo
4. Vercel auto-detects it's a Node.js project
5. **Before deploying**, go to "Environment Variables" and add all 4:
   - `LENS_SUPABASE_URL`
   - `LENS_SUPABASE_SERVICE_KEY`
   - `LENS_SUPABASE_ANON_KEY`
   - `LENS_ALCHEMY_KEY`
   - `CRON_SECRET`

6. Click Deploy

After deploy, Vercel gives you a URL like `lens-liard.vercel.app`. This is your API URL.

### 3. Enable Cron (optional but recommended)

The `/api/index-launches` endpoint runs every 5 minutes to fetch new token launches from Bankrbot and save to Supabase.

To enable:
1. Vercel dashboard → Project → Settings → Cron Jobs
2. Should auto-detect the cron job from `vercel.json`
3. Enable it

### 4. Update extension config

Once the backend is live, update the extension to point to your new API URL.

In the LENS extension popup settings, set `LENS_API_URL` to your new Vercel URL (or custom domain if you set one up).

## Custom Domain (Optional)

If you have a domain (like `api.lnsx.io`), point it at your Vercel deployment:

1. In Vercel → Project → Settings → Domains
2. Add your domain
3. Vercel gives you a CNAME target
4. In your DNS provider (Cloudflare, Route53, etc), add a CNAME record:
   - Name: `api` (or whatever subdomain)
   - Target: the Vercel CNAME
   - Type: CNAME
5. Wait for DNS to propagate (usually 5-15 min)

Now your API is at `https://api.yourdomain.com/api/lookup`.

## Troubleshooting

**`Error: LENS_SUPABASE_URL is not defined`**
- Check that all env vars are set in Vercel dashboard (not just .env file)
- Redeploy after adding env vars

**`Error: 401 Unauthorized` from Alchemy**
- Check that `LENS_ALCHEMY_KEY` is correct
- Make sure the Alchemy key is for the **Base mainnet**, not Ethereum

**`No data returned from /api/lookup`**
- The handle might not have any launches indexed yet
- Check Supabase → `bankr_launches` table to see if data is there
- If empty, wait a few minutes for the cron job to run, or trigger it manually:
  ```bash
  curl "https://your-url.vercel.app/api/index-launches?secret=YOUR_CRON_SECRET"
  ```

**`CORS error` when calling from browser**
- The API allows CORS from any origin (safe since it's read-only)
- If you still see CORS errors, check that you're calling the correct endpoint

## Environment Variables Reference

| Variable | Required | Where to get it |
| --- | --- | --- |
| `LENS_SUPABASE_URL` | yes | Supabase → Project Settings → API → Project URL |
| `LENS_SUPABASE_SERVICE_KEY` | yes | Supabase → Project Settings → API → service_role |
| `LENS_SUPABASE_ANON_KEY` | yes | Supabase → Project Settings → API → anon (public) |
| `LENS_ALCHEMY_KEY` | yes | Alchemy → App Dashboard → copy your key |
| `CRON_SECRET` | yes | Invent any random string (keeps cron job secure) |

---

Next: [API Documentation](./API.md)
