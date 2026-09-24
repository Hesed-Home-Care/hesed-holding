# Hesed — hesedhomecare.org

One-page static marketing site for Hesed, a private investment holding company (Boulder, CO).

<!-- estate-facts
repo: Hesed-Home-Care/hesed-holding
branch: main
services: none
ports: none
urls: https://hesedhomecare.org, https://www.hesedhomecare.org
deploy: pages-deploy (via deploy-listener)
last_verified: 2026-09-24
-->

## What it is and who uses it

The public front door for the holding company: who Hesed is, how it thinks about owning
companies, and what it holds (Pipestaff, LokiMode, Colorado CareAssist). No app, no login, no
data — a page plus the machine-readable layer around it (JSON-LD, `robots.txt`, `sitemap.xml`,
`llms.txt`) so Google and AI answer engines describe the business correctly.

Jason owns the copy and decides what goes live. Anyone with the repo can make the edit; there is
no back office to operate.

## Tech stack

From `package.json` and `next.config.mjs`:

- **Next.js `^16` + React `^19`**, TypeScript `^5.6` (strict), Node **>= 24** (`.nvmrc` = `24`).
- **`output: 'export'`** — the build emits a static `out/` directory; nothing runs on a server.
  `images.unoptimized: true` (required: no Next image optimizer in a static export) and
  `trailingSlash: true` (every URL ends in `/`).
- **No database, no API keys, no runtime dependencies beyond React.** Fonts load from the Google
  Fonts CDN in `app/layout.tsx`: Archivo (latin), Heebo (the Hebrew `חֶסֶד`).
- Deploy tooling: `npx wrangler@4.112.0 pages deploy` (invoked by `~/scripts/pages-deploy.sh`,
  not by this repo).

## Architecture

```
lib/site-config.ts  ──all copy + 2 flags──►  app/page.tsx  ──►  out/index.html
design-system/styles.css ──tokens──► app/globals.css ──► (imported by app/layout.tsx)
lib/organization.json ──JSON-LD @graph──► app/layout.tsx <head>
public/* ──copied verbatim──► out/*
```

| Path | What it does |
|---|---|
| `lib/site-config.ts` | **Single source of truth for every word on the page** — nav, hero, the name section, approach principles, quote, holdings rows, closing, footer, metadata. Also two build-time flags: `showPortrait` (renders the capital-flow band) and `posterClose` (navy poster closing vs. quiet closing). |
| `app/page.tsx` | The whole page, one server component: `Nav, Hero, NameSection, Approach, Quote, PhotoBand, Holdings, ClosePoster/CloseQuiet, Footer`. Reads `siteConfig`; holds no copy of its own. `Rich` renders the fields that carry inline markup (`<em>`/`<strong>` in the name paragraphs and the approach text). |
| `app/layout.tsx` | `<head>`: metadata/OG/Twitter, favicon set, Google Fonts preconnects, and `lib/organization.json` injected as `<script type="application/ld+json">`. |
| `app/globals.css` | `@import "../design-system/styles.css"` plus body/link/focus/selection defaults. |
| `app/page.module.css` | This page's layout: the grid, the poster field, the Hebrew (`Heebo`) rules. |
| `lib/organization.json` | Schema.org `@graph`: `HoldingCompany`/`Organization` (+ `subOrganization` for the three holdings) and `WebSite`. |
| `design-system/styles.css` | The token sheet (`:root` variables, 100–900 ramps, `.btn/.tag/.card/...`) — the look lives here. Guide: `design-system/readme.md`. |
| `public/capital-flow.html` | Standalone canvas animation ("capital, deployed and held"), embedded by `PhotoBand` in an `<iframe src="/capital-flow.html">`. |
| `public/_redirects` | Cloudflare Pages rules: legacy Pipestaff paths on this domain (`/for-agencies`, `/pricing`, `/blog/*`, …) 301 to `pipestaff.com`. Longest-prefix match — splats before any catch-all. |
| `public/robots.txt`, `public/sitemap.xml`, `public/llms.txt` | Crawler/AI surface. `llms.txt` restates the holdings in prose. |
| `public/og-image.png`, `favicon.ico/svg`, `apple-touch-icon.png` | Social unfurl + icons. |

Data stores: none. Integrations: none (outbound links to the three portfolio sites only).

## Run it locally

```bash
cd <this repo>
nvm use                      # node 24
npm install
PORT=4174 npm run dev        # http://localhost:4174  — do NOT use the default 3000 (live cca.com port)
npm run build                # static export → ./out
npx serve out -l 4173        # preview what actually ships
```

No env vars are needed to build or preview. Only a deploy needs credentials (see Deploy).

## Tests

```bash
npm run typecheck            # tsc --noEmit — the only check in the repo
```

There is **no test suite**. A clean typecheck proves the TSX/CSS-module imports and `siteConfig`
shape compile. It proves nothing about rendering, links, SEO output, or that the export was
uploaded — after a deploy, read the live page (see Operate).

## Deploy

There is no git integration on the Cloudflare Pages project — **direct-upload projects can never be
git-attached** (the API and the dashboard both refuse). The webhook below *is* the auto-deploy.

1. Push to `main` on `Hesed-Home-Care/hesed-holding`.
2. `deploy-listener` (port 3099) receives the webhook, pulls, and runs its `DEPLOY_MAP` entry:
   `/Users/shulmeister/scripts/pages-deploy.sh /Users/shulmeister/mac-mini-apps/hesed-holding hesed-holding main out`
3. With the `out` argument the script runs `npm run build` in the **live** checkout and uploads
   `out/` with `wrangler pages deploy --project-name=hesed-holding --branch=main`.
4. Migrations: none. Reload: none — Pages serves new files.

Manual deploy (same path, run by hand; needs `CF_GLOBAL_API_KEY`, `CF_AUTH_EMAIL`, `CF_ACCOUNT_ID`
in the environment — they live in `~/.config/careassist/resolved-secrets.env`, never commit them):

```bash
bash ~/scripts/pages-deploy.sh ~/mac-mini-apps/hesed-holding hesed-holding main out
```

Hand steps: nothing, provided the live checkout is **clean and on `main` at the pushed SHA**. A
dirty or detached live checkout freezes this repo's deploys — the build runs there, not in CI.

## Operate

- **Service / port:** none. This site is not in the launchd fleet; there is nothing to restart.
- **Public URL:** `https://hesedhomecare.org` (+ `www`, + `hesed-holding.pages.dev`).
- **Health check:**
  ```bash
  curl -s -o /dev/null -w '%{http_code}\n' https://hesedhomecare.org/    # 200
  curl -s https://hesedhomecare.org | grep -c 'application/ld+json'      # 1
  curl -sI https://hesedhomecare.org | grep -i access-control-allow-origin  # '*' — Pages is serving, not the tunnel
  ```
- **Logs:** the build/upload output lands in `~/logs/deploy-listener.log`.
- **Scheduled jobs:** none owned by this repo.
- ⚠️ The Cloudflare tunnel config still carries inert `hesedhomecare.org → localhost:3001` rows
  (that is the separate `hesedhomecare` repo = Pipestaff). DNS points at Pages; **:3001 is not this
  site's origin**.

## Standard operating procedures

**1. Change wording anywhere on the page**
1. Edit `lib/site-config.ts` — never paste copy into `app/page.tsx`.
2. `npm run typecheck`.
3. `git commit` + push to `main`; wait for the listener.
4. `curl -s https://hesedhomecare.org | grep -o 'your new phrase'` to confirm it shipped.

**2. Add or remove a holding** — the same fact lives in three places; update all three or the
page and the machine-readable layer disagree:
1. `lib/site-config.ts` → `holdings.rows` (and the hero's `subTail`, `close.buttons`, `footer.holdingsLine`, `metadata.description`).
2. `lib/organization.json` → `subOrganization`.
3. `public/llms.txt` → the Holdings list.

**3. Retire a legacy Pipestaff URL** — add or edit a rule in `public/_redirects`; keep specific
paths above any `/*` splat and the catch-all last.

**4. Change the look** — edit the tokens at the top of `design-system/styles.css`
(`--color-*`, `--font-*`, `--space-*`, `--radius-*`, `--shadow-*`); every page reads them.
Don't hard-code a hex or px value a token already carries (see `design-system/readme.md`).

**5. Change a legal fact (address, phone, license)** — `lib/site-config.ts` `footer` and
`lib/organization.json` must match; both are published claims.

## Gotchas

- **Copy lives only in `lib/site-config.ts`.** Hard-coded strings in `page.tsx` are invisible to
  the next person editing "the content file."
- **A new holding is a three-file change** (site-config / organization.json / llms.txt). Drift is
  what AI crawlers repeat.
- **Don't re-add `hesedfoundation.org` to the JSON-LD `sameAs`** (dropped 8/24). The foundation is a
  separate entity, not an alias of Hesed; the on-page footnote link is the correct treatment.
- **No `foundingDate` in the schema** unless Hesed confirms one — "since 2012" is Colorado
  CareAssist's operating date, not the holding company's.
- **Static export constraints:** any server-only API or `next/dynamic` with `ssr: false` fails the
  build. Everything must render at build time.
- **`public/*` is copied into `out/`** — edit `robots.txt`, `sitemap.xml`, `llms.txt`,
  `_redirects` in `public/`, never in `out/` (build output, regenerated).
- **Never bind a local server to a production port** (`8765, 8767, 8769, 8771, 3000, 3001, 3003,
  3015, 3020, 3030, 3099`) — a stray `python -m http.server` on one of them made the health-monitor
  poll a directory listing and restart-loop the portal. Use `4173+`.
- **Legal ground truth:** one entity, `Hesed Home Care LLC, d/b/a Colorado CareAssist`. Banned:
  `Asset Home Care`, `Colorado CareAssist Inc`, `Colorado CareAssist LLC`. The published phone
  `(303) 757-1777` is answered by people; never substitute another line.
- **This repo is not the `hesedhomecare` repo.** That one is Pipestaff (port 3001, pipestaff.com).
  Same domain family, different product.

## Related docs

- `docs/README.md` — what's current in this repo
- `design-system/readme.md` — the Modernist tokens and component classes
- `docs/archive/DEPLOY-HANDOFF.md` — the 2026-08 pre-listener deploy history (not current)
- `/Users/shulmeister/docs/estate/README.md` — estate-wide facts (launchd, Postgres, tunnel, secrets, backups)
- `/Users/shulmeister/docs/estate/DOCS-STANDARD.md` — the doc contract this README follows
