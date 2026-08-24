# Hesed holding page — deploy handoff

## What I did (this session)

In `~/mac-mini-apps/hesed-holding/`, committed and pushed to `origin/main` as **`232b693`** (rebased on top of an existing remote commit `17a44ee`):

- Created `lib/organization.json` — JSON-LD `@graph`: HoldingCompany/Organization (legalName, address 1911 11th St Fl 2 Boulder CO 80302, phone +1-303-757-1777, subOrganization for Pipestaff/LokiMode/Colorado CareAssist) + WebSite
- Imported and rendered in `app/layout.tsx` `<head>` as a script tag
- Fixed `twitter.title` to match `og.title` (was just `'Hesed'`)
- `sitemap.xml`: added `lastmod` and `priority`
- `llms.txt`: added LokiMode to Holdings (was missing despite being on the page)
- Removed an unverified `foundingDate` claim — verify with Hesed if you want it back

Validation passed locally: JSON-LD parses with 3 suborgs, layout imports cleanly.

## Why nothing is live yet

The Cloudflare Pages project **`hesed-holding`** has `source: None` — no git integration attached. Latest successful deploy was `2026-07-16T17:53:27Z` (manual), which is **before** my push. So pushes don't trigger builds. Same pattern as Pipestaff and LokiMode.

## What to do

### Manual deploy (gets the live site updated now)

The CF Global API key in env has Pages access. The zone-scoped `CF_API_TOKEN` does NOT.

```bash
cd ~/mac-mini-apps/hesed-holding
npm install           # if node_modules missing or stale
npm run build         # static export → out/
CLOUDFLARE_API_KEY="$CF_GLOBAL_API_KEY" \
CLOUDFLARE_EMAIL="$CF_AUTH_EMAIL" \
CLOUDFLARE_ACCOUNT_ID="$CF_ACCOUNT_ID" \
npx --yes wrangler@4 pages deploy out --project-name=hesed-holding --branch=main --commit-dirty=true
```

The Next.js app is configured with `output: 'export'` (see `next.config.mjs`), so the build produces a static `out/` directory. Wrangler uploads that to Cloudflare Pages.

After success, verify:

```bash
for p in /organization.jsonld /robots.txt /sitemap.xml; do
  printf "%-26s %s\n" "$p" "$(curl -s -o /dev/null -w '%{http_code}' https://www.hesedhomecare.org$p)"
done

# JSON-LD should be present in homepage HTML
curl -s https://www.hesedhomecare.org | grep -c 'application/ld+json'

# Twitter title should match og:title (not just "Hesed")
curl -s https://www.hesedhomecare.org | grep -oE '<meta name="twitter:title"[^>]*>'
```

All should be 1 (or 200) and the twitter:title should read `"Hesed — a private investment holding company"`.

### Optional: attach the GitHub repo for future auto-deploys

1. https://dash.cloudflare.com → **Workers & Pages** → **hesed-holding** → **Settings** → **Builds**
2. **Connect to Git** → GitHub → authorize on `Hesed-Home-Care` org → pick **`hesed-holding`**
3. **Production branch**: `main` (already correct — matches repo default)
4. **Build command**: `npm run build`
5. **Build output directory**: `out` (Next.js static export)
6. **Root directory**: `/`
7. Save. Cloudflare queues a deploy of `main` HEAD (`232b693`). Auto-deploys on every push after that.

## Quick API checks (diagnostics)

```bash
# Project config (source / branch / latest deploy)
curl -sH "X-Auth-Email: $CF_AUTH_EMAIL" -H "X-Auth-Key: $CF_GLOBAL_API_KEY" \
  "https://api.cloudflare.com/client/v4/accounts/$CF_ACCOUNT_ID/pages/projects/hesed-holding" \
  | python3 -c "import json,sys; p=json.load(sys.stdin)['result']; print('source:', p['source'], '| prod_branch:', p['production_branch'], '| latest_deploy:', p['latest_deployment']['created_on'])"

# Latest deploys
curl -sH "X-Auth-Email: $CF_AUTH_EMAIL" -H "X-Auth-Key: $CF_GLOBAL_API_KEY" \
  "https://api.cloudflare.com/client/v4/accounts/$CF_ACCOUNT_ID/pages/projects/hesed-holding/deployments?per_page=3" \
  | python3 -c "import json,sys; [print(d['created_on'], d['latest_stage']['status']) for d in json.load(sys.stdin)['result']]"
```

## Watch out for

- **`foundingDate`**: I removed an unverified `foundingDate: '2012'` (I inferred it from the CCA "since 2012" line, but that refers to the operating agency, not the holding company). If you want it back, verify the actual date with Hesed.
- **Brand mentions of LokiMode**: Hesed's holding page already names LokiMode in the live page; I added it to `llms.txt` so AI crawlers see a consistent set of holdings.
- **Next.js static export** (`output: 'export'`): any `next/dynamic` with `ssr: false` won't render at build time, and any server-only APIs will fail the build. The current page is plain, so build should be clean.
- **`@` import alias**: tsconfig.json has `"@/*": ["./*"]` — keep that in mind if you restructure `lib/`.
- **Token scope**: env has both `CF_API_TOKEN` (zone-scoped, no Pages) and `CF_GLOBAL_API_KEY` (full). Use the global key for `wrangler pages deploy`. Newer wrangler uses `CLOUDFLARE_API_TOKEN` / `CLOUDFLARE_EMAIL` / `CLOUDFLARE_ACCOUNT_ID`.
