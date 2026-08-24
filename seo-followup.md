# hightidemgt SEO - pick up here

Last worked: 2026-08-21 (site changes), 2026-08-23 (GSC check, interrupted), 2026-08-24 (indexing-blocker fixes, shipped)

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

## Open item 1 (still open): are the four new pages indexed yet?

Not checked. Requested 2026-08-21; expect crawl within days.

To check: GSC -> URL inspection -> paste each of:
- https://hightidemgt.com/locations/branford/
- https://hightidemgt.com/services/financial-data-destruction/
- https://hightidemgt.com/services/legal-data-destruction/
- https://hightidemgt.com/services/government-itad/

Do NOT re-request indexing if still pending - Google states repeat submissions do not change queue position.

## Open item 2 (largely resolved 8/24 - see the 8/24 section above): the indexing gap

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

## 2026-08-24: four indexing blockers found and fixed (`a6b280b`, pushed + verified live)

Approach that worked: instead of reading GSC counts, I swept every URL with curl
as Googlebot. **All 49 sitemap URLs already returned clean 200s** - so the "not
indexed" reasons were all coming from URLs *outside* the sitemap. That reframed
the hunt and found four real defects.

**1. The vercel.json redirects were broken.** Sources lacked trailing slashes, so
trailingSlash normalization ran first and won:
`/certifications` -> 308 -> `/certifications/` -> **404**. The redirect to
`/our-network/` never fired. Same for `/certifications/r2-certified`. This is the
exact gotcha already documented in the root CLAUDE.md for riversedge - it applies
here too. Fixed by adding trailing slashes to both `source` values (specific rule
first). Now verified: `/certifications` follows through to `/our-network/` 200.

This one defect plausibly explains several GSC buckets at once: the redirect
count, and 2 of the 4 "Not found (404)".

**2. The three blog posts self-canonicalized to a redirecting URL.** They were
linked and canonicalized as `/blog/*.html`, which cleanUrls 308s to `/blog/*/`.
A canonical pointing at a redirect is a strong "do not index this" signal. They
were also **missing from the sitemap entirely** - three real content pages with
no clean-URL entry point anywhere. Fixed canonical, `og:url`, breadcrumb `item`,
`mainEntityOfPage` `@id`, and every inbound internal link; added all three to the
sitemap (49 -> 52 URLs).

**3. Every OG image 404'd.** All 52 `og:image` references pointed at
`/images/og-home.jpg`, `og-services.jpg`, `og-about.jpg`, `og-contact.jpg` -
none of those files existed. Social/link previews were blank sitewide. Fixed by
generating the four referenced files at 1200x630 from existing site photography
(`High-tide-hero-min.jpeg`, `High-Tide-Commodities.jpeg`) via `sips`, so zero
HTML changes were needed.

**4. Confirmed clean:** canonicals on all other 49 pages self-reference
correctly; a genuinely missing route (`/nonexistent-page-xyz/`) correctly returns
404; `noindex` appears only on the 404 page, as intended.

Post-deploy verification (all passing):
- 52/52 live sitemap URLs return 200 to Googlebot
- `/certifications/` and `/certifications/r2-certified/` both terminate at `/our-network/` 200
- all four OG images return 200
- all three blog canonicals now self-reference the clean URL

## Still open

**The 403 x2.** Not identified - these cannot be found from outside, since every
URL I can enumerate returns 200 or a correct 404. Needs the actual URL list from
GSC -> Indexing -> Pages -> "Blocked due to access forbidden (403)". Likeliest
explanation is Google holding a stale record of a URL that 403'd during a
transient Vercel state, in which case it clears itself on recrawl.

**Read the 5th reason row** in GSC -> Indexing -> Pages (was below the fold on
8/23, ~13 pages, likely "Crawled/Discovered - currently not indexed"). That
bucket is now the largest unexplained chunk. Note the counts predate these fixes.

**Are the four 8/21 pages indexed?** Still unchecked (branford, financial, legal,
government-itad). Do NOT re-request indexing if still pending.

**Resubmit the sitemap** - it grew from 49 to 52 URLs on 8/24, so GSC should be
pointed at it again to pick up the three blog posts.

**Recheck after recrawl.** The redirect and canonical fixes should move pages out
of the "redirect"/"404" buckets on their own; give it 1-2 weeks.

## Convention debt (unrelated to indexing)

510 em/en dashes across 31 of 53 HTML files. The workspace CLAUDE.md forbids them
outright ("No em dashes or en dashes in any content"), both stylistically and
because they produce encoding gremlins without a charset header. Not touched
here - it is a copy-wide sweep, separate from the indexing work, and worth doing
as its own commit.

## Gotchas learned (save yourself the time)

- **`https://hightidemgt.com/404.html` returns 308**, not the page. `cleanUrls` + `trailingSlash` in `vercel.json` rewrite direct `.html` paths. To see the real 404 page, request a genuinely missing route like `/nonexistent-page-xyz/`.
- **The sitemap ping endpoint is dead.** Google retired `google.com/ping?sitemap=` in early 2024. Resubmission must go through the Search Console UI or the Search Console API - there is no curl one-liner.
- **GSC deep-links to URL inspection do not work.** `.../inspect?resource_id=...&id=<url>` 404s. Use the "Inspect any URL" search bar at the top of the property instead.
- **GSC property is duplicated** - there is a domain property (`hightidemgt.com`) and a URL-prefix property (`https://hightidemgt.com/`). Sitemap state is shared between them; submitting to one is enough.
- **Vercel redirect `source` values need trailing slashes** when `trailingSlash: true` is set - normalization runs BEFORE the redirect table, so a slashless source never matches. A redirect that looks configured can be silently dead and landing on a 404.
- **Curl-sweeping every URL as Googlebot beats reading GSC counts.** It found four defects in minutes that the GSC summary numbers only hinted at.
- **GSC data lags 1-3 days.** The Pages report showed "last update 8/20/26" when read on 8/23.

## Background: the original problem

Baseline GSC (from the first audit): 44 clicks, 1,580 impressions, average position 24.8, **8 of the top 10 queries were the brand name**.

The brand-query dominance is a demand problem that new pages alone do not solve. The structural work above was a prerequisite, not the fix. Worth revisiting the query mix in a few weeks to see whether non-brand queries start appearing.

## Other working notes in this repo

`ga4-gsc-inventory.md`, `ga4-tags-to-install.md`, and `gsc-link-checklist.md` are deliberately left untracked. This file is committed so it survives a clean checkout.
