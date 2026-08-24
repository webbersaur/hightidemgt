# hightidemgt SEO - pick up here

Last worked: 2026-08-21 (site changes), 2026-08-23 (GSC check, interrupted)

## Done and shipped

Four commits, all pushed to `main` and live on Vercel:

| Commit | What |
|---|---|
| `7da43fc` | Branford location page + internal-link orphan fix |
| `224ac9e` | Financial, legal, government/municipal vertical pages |
| `b9ec585` | Nav + footer unified on one shared markup block |
| `90da84b` | 404 page invisible-content fix |

Detail:

- **`/locations/branford/`** - HQ town, previously in zero titles and zero H1s sitewide. Linked from homepage grid, locations index card, footer, sitemap.
- **Three verticals** - `/services/financial-data-destruction/`, `/services/legal-data-destruction/`, `/services/government-itad/`. Healthcare was the only vertical with a page before. Each has Service + BreadcrumbList + FAQPage schema.
- **Orphan fix** - 11 long-tail pages had ZERO inbound internal links (reachable only via sitemap). Fixed via 12 new blog-index cards + "Related Reading" blocks on 16 service/location pages. Site minimum is now 2 inbound links; most are far higher.
- **Nav unified** - header had drifted into 16 variants across 53 pages, mobile nav into 6. Now 1 each. Also restored the Locations link to 14 pages that had lost it. `/commodities/` and `/merchandising-recycling/` had zero body links and lived only in the legacy dropdown, so they were moved to the footer Services column to avoid orphaning them.
- **404 fix** - was rendering white-on-white (used `.hero`, whose text colors are white, without the `.hero__content` blue background wrapper). Switched to the `.page-header` pattern. Pre-existing bug, not introduced by this work.

Search Console (2026-08-21):
- Sitemap resubmitted, discovered pages went 45 -> 49
- "Request indexing" submitted for all four new URLs, all confirmed "added to a priority crawl queue"

## Open item 1: are the four new pages indexed yet?

Not checked. Requested 2026-08-21; expect crawl within days.

To check: GSC -> URL inspection -> paste each of:
- https://hightidemgt.com/locations/branford/
- https://hightidemgt.com/services/financial-data-destruction/
- https://hightidemgt.com/services/legal-data-destruction/
- https://hightidemgt.com/services/government-itad/

Do NOT re-request indexing if still pending - Google states repeat submissions do not change queue position.

## Open item 2: the indexing gap (the real one)

This is the item worth actual work. A large share of the site is not indexed, and it predates all of the above.

Partial read from GSC -> Indexing -> Pages on 2026-08-23 (data "last updated 8/20/26", so it reflects the state BEFORE these changes):

- **Indexed: 28 | Not indexed: 25** (5 reasons)
- Reasons visible before I got interrupted:
  - Page with redirect - 5
  - Not found (404) - 4
  - Blocked due to access forbidden (403) - 2
  - Excluded by 'noindex' tag - 1
  - (a 5th reason row was below the fold, ~13 pages, unread - likely "Crawled/Discovered - currently not indexed")

**Next step: open that report, read the 5th reason, and click into each reason to get the actual URL lists.** The counts alone are not actionable. Specifically worth explaining:

- **403 x2** - a static Vercel site should not be returning 403 to Googlebot. Investigate.
- **404 x4** - which URLs? If they are old paths, they need redirects in `vercel.json`.
- **noindex x1** - is `404.html` (intentional, it carries `noindex, nofollow`). Probably fine, verify it is only that one.
- **Redirect x5** - likely the `cleanUrls`/`trailingSlash` normalization. Verify nothing in the sitemap points at a redirecting URL.

Note the earlier Overview screen on 8/21 showed "19 indexed / 17 not indexed" - different figure from the Pages report's 28/25. Different reports and date ranges; do not treat either as gospel, use the Pages report URL lists.

## Gotchas learned (save yourself the time)

- **`https://hightidemgt.com/404.html` returns 308**, not the page. `cleanUrls` + `trailingSlash` in `vercel.json` rewrite direct `.html` paths. To see the real 404 page, request a genuinely missing route like `/nonexistent-page-xyz/`.
- **The sitemap ping endpoint is dead.** Google retired `google.com/ping?sitemap=` in early 2024. Resubmission must go through the Search Console UI or the Search Console API - there is no curl one-liner.
- **GSC deep-links to URL inspection do not work.** `.../inspect?resource_id=...&id=<url>` 404s. Use the "Inspect any URL" search bar at the top of the property instead.
- **GSC property is duplicated** - there is a domain property (`hightidemgt.com`) and a URL-prefix property (`https://hightidemgt.com/`). Sitemap state is shared between them; submitting to one is enough.
- **GSC data lags 1-3 days.** The Pages report showed "last update 8/20/26" when read on 8/23.

## Background: the original problem

Baseline GSC (from the first audit): 44 clicks, 1,580 impressions, average position 24.8, **8 of the top 10 queries were the brand name**.

The brand-query dominance is a demand problem that new pages alone do not solve. The structural work above was a prerequisite, not the fix. Worth revisiting the query mix in a few weeks to see whether non-brand queries start appearing.

## Other working notes in this repo

`ga4-gsc-inventory.md`, `ga4-tags-to-install.md`, and `gsc-link-checklist.md` are deliberately left untracked. This file is committed so it survives a clean checkout.
