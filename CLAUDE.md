# CLAUDE.md — hesed-holding

Human docs: README.md. This file holds agent rules and traps.

Everything about what this site is, how it's built, how it deploys and how to operate it is in
`README.md`. Read that first. This file is only the rules that are easy to break.

## Hard rules

- ⛔ **No secrets, no personal data in this repo's docs or code.** Deploy credentials are env-var
  *names* only (`CF_GLOBAL_API_KEY`, `CF_AUTH_EMAIL`, `CF_ACCOUNT_ID`), resolved from
  `~/.config/careassist/resolved-secrets.env`. Never echo a value into a file, commit, or log.
- **Never deploy from this worktree or from an ad-hoc `out/`.** Deploys happen in the live checkout
  `~/mac-mini-apps/hesed-holding` via `deploy-listener`. Editing that checkout is deploying.
- ⛔ **Never `git push`, open a PR, or touch Cloudflare/DNS config unless Jason asked for it in this
  session.** Commit on the branch; a human merges.
- **Copy edits go in `lib/site-config.ts`.** `app/page.tsx` must stay free of literal copy. The two
  flags there (`showPortrait`, `posterClose`) are the only intended layout switches.
- **A holding change is three files:** `lib/site-config.ts`, `lib/organization.json`
  (`subOrganization`), `public/llms.txt`. Update all three in the same commit.
- **Published claims are legal facts.** Entity is `Hesed Home Care LLC, d/b/a Colorado CareAssist`
  (banned: `Asset Home Care`, `Colorado CareAssist Inc`, `Colorado CareAssist LLC`). Address, phone
  `(303) 757-1777` and license `#04Y296` must match between `site-config.ts` and
  `organization.json`. Never substitute another phone number.
- **Jason's copy is Jason's.** This page is public brand voice, dictated and reviewed. Rewrite it
  only when asked; a grammar "fix" on live marketing copy is an unrequested edit.

## Traps (each one cost someone time)

- ⛔ **The Cloudflare Pages project `hesed-holding` is direct-upload and can NEVER be git-attached**
  (API and dashboard both refuse). "Connect to Git" instructions in old docs are wrong — the
  deploy-listener webhook running `~/scripts/pages-deploy.sh … main out` *is* the auto-deploy.
- **`pages-deploy.sh` builds in the live checkout**, so a dirty/behind checkout ships the wrong tree
  (or fails). `git -C ~/mac-mini-apps/hesed-holding status --porcelain` before blaming the pipeline.
- **`git archive HEAD` mode strips `README.md` and `DEPLOY-HANDOFF.md` from the upload** (a 7/16
  near-miss published internal notes). With the `out` argument only build output ships — but never
  park sensitive files inside `public/`, which is copied verbatim into `out/`.
- **Don't re-add `hesedfoundation.org` to the JSON-LD `sameAs`.** Removed 8/24 (commits #6, #8): the
  foundation is separate philanthropy, not an alias of the holding company. The on-page footnote
  link is the correct surface.
- **No `foundingDate` in the schema** unless Hesed confirms it. "Since 2012" on the page is Colorado
  CareAssist's operating date; inferring it for the holding company produced a false claim that had
  to be deleted.
- **`output: 'export'`**: no route handlers, no server-only APIs, no `next/dynamic` with
  `ssr: false`, and `images.unoptimized: true` is mandatory — otherwise the build dies.
  `trailingSlash: true` means canonical URLs end in `/`; keep `sitemap.xml`/`_redirects` consistent.
- **Edit `public/*`, never `out/*`** — `out/` is generated and gitignored.
- **`_redirects` is longest-prefix match.** Keep specific rules above their `/*` splats, and the
  catch-all last. Legacy Pipestaff paths must 301 to `pipestaff.com`, not to this page.
- **Local preview ports:** never bind `8765, 8767, 8769, 8771, 3000, 3001, 3003, 3015, 3020, 3030,
  3099` — those route real traffic through the tunnel/health-monitor, and a `python -m http.server`
  on one of them caused a portal restart-loop incident. Use `4173+`; `next dev` defaults to the
  taken port 3000, so pass `PORT=4174`.
- **`:3001` is not this site.** The tunnel config still has inert `hesedhomecare.org → localhost:3001`
  rows belonging to the *other* repo (`hesedhomecare` = Pipestaff). DNS points at Pages. Don't
  document or debug 3001 as this site's origin, and don't conflate the two repos.
- **Node must be >= 24** (`.nvmrc`); `nvm use` before building. The Pages-equivalent build runs the
  same `npm run build`.
- **Dead pointer:** the header comment in `lib/site-config.ts` cites a design spec at
  `~/docs/superpowers/specs/2026-07-15-hesed-holding-site-design.md` — that file no longer exists
  (checked 2026-09-24). Don't hunt for it or re-link it.
- **Nothing here is verified by tests.** `npm run typecheck` is the whole suite; "it compiled" is not
  evidence the page is right — check the live HTML after a deploy (`application/ld+json` count, the
  phrase you changed).

## Definition of done for a change in this repo

```bash
npm run typecheck
git -C <this repo> status --porcelain            # clean after commit
curl -s https://hesedhomecare.org | grep -c 'application/ld+json'   # 1, after it actually shipped
```

Estate-wide facts (launchd, Postgres, tunnel, secrets, backups): `/Users/shulmeister/docs/estate/README.md`.
