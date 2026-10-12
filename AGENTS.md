# truenas-pulse-site

Promotional/marketing site for TrueNAS Pulse (third-party iOS client for TrueNAS SCALE). Live at https://truenaspulse.com. Static site, zero JS by default.

## Stack

- Astro 5 (static output)
- Tailwind CSS v4 — native Vite plugin, brand tokens live in a single `@theme` block in `src/styles/global.css`
- MDX for the legal pages (Privacy, Terms, EULA)
- Custom ~30-line IntersectionObserver + WAAPI scroll-reveal — no animation library
- `sharp` for build-time favicon/OG-image generation
- Bun as package manager and runtime (Node 20+ works too)

## Commands

```bash
bun install
bun run dev          # http://localhost:4321, hot reload
bun run build        # static build into ./dist
bun run preview      # serve ./dist locally
bun run check        # astro check — type/diagnostics pass
```

Regenerate favicons/OG image/webmanifest from the iOS app's master icon:
```bash
bun run scripts/generate-assets.mjs
```
It looks for `../TrueNAS-Pulse/assets/icon/TrueNAS_Pulse_AppIcon_Master_UNMASKED_1024.png` first, falling back to `./public/icon-master.png`.

No test suite exists in this repo (no `test` script, no test framework dependency).

## Layout

```
src/components/   BaseLayout, Header, Footer, Hero, DeviceFrame, ScreenMock, FeatureGrid,
                  TierCompare, PillarStrip, CallToAction, MotionReveal, Section, AppStoreButton, ...
src/pages/        index.astro, features.astro, screenshots.astro, pro.astro, faq.astro,
                  changelog.astro, contact.astro, 404.astro, privacy.mdx, terms.mdx, eula.mdx
src/data/         site.ts (identity/copy/links), features.ts (26-feature canon), changelog.ts, screenshots.ts
src/styles/       global.css (single @theme + tiny utilities)
public/           favicon set, og-image.png, site.webmanifest, robots.txt, screenshots/ — mostly generated
scripts/          generate-assets.mjs (sharp pipeline), fetch-changelog.ts, csp-hash-compute.py, csp-hash-patch.py
.gitea/workflows/deploy.yml   self-hosted Gitea Actions runner: bun build -> rsync to LXC 132
.github/workflows/deploy.yml  GitHub Actions: bun build -> Cloudflare Pages production
```

## Content source of truth

Marketing copy is adapted from the iOS app's own docs in a sibling checkout (`../TrueNAS-Pulse/`):
- `docs/APP_STORE_METADATA.md` — hero tagline, feature copy
- `docs/MONETIZATION.md` — tier copy, pricing rationale
- `docs/PRO_FEATURES.md` — canonical 26-feature registry
- `TrueNAS Pulse/Design/Design.swift` — color tokens

When those iOS docs change, update `src/data/features.ts` and any affected component copy to match. Brand voice: **calm, deliberate, infrastructure-grade — never playful.**

## Deployment

Push `main` to Gitea `phobos/truenas-pulse-site`; its mirror updates GitHub
`Phobos-cc/truenas-pulse-site`. Both workflows deploy automatically:

- Public `https://truenaspulse.com` is served by **Cloudflare Pages**, project
  `truenas-pulse-site`. `.github/workflows/deploy.yml` builds with Bun and uses
  `cloudflare/wrangler-action` with the existing GitHub Cloudflare API secret.
- `.gitea/workflows/deploy.yml` maintains the homelab origin copy: Bun build,
  rsync `dist/` to LXC 132 (`192.168.0.93:/var/www/pulse-site/`), Caddy ownership
  and CSP hash update, validation/reload, then six origin smoke probes.
  The origin is static Caddy only: no analytics, no `/api/support` (the support
  form drafts a `mailto:`), no Docker or Bun. See README → Deployment.

The legacy `pulse.phobos-cc.be` address redirects to production. Do not claim
public publication from an origin build alone: verify the GitHub workflow and
live public copy. Preserve source commits/build hashes and back up the origin
site plus `/etc/caddy/Caddyfile` before changing that origin. Full deployment
details are in `README.md`. No local manual deploy is normally required.

## Trademark note

TrueNAS® is a registered trademark of iXsystems, Inc. This is an independent third-party site, not affiliated with or endorsed by iXsystems — keep that distinction intact in any copy changes.
