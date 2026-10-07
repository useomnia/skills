# Cutover

The user switches production; you prepare, check and support. Never change DNS yourself.

## Before the switch

- [ ] Every item on the "Flagged for owner" list is decided and applied.
- [ ] Re-run the crawl and the content export, and re-sync content edited on the old site since capture.
- [ ] The platform's full 301 export is merged into the redirect rules; chains collapsed so each rule
      points straight at the final URL.
- [ ] The search-index check passes on the preview: every URL search engines know answers 200 or 301.
- [ ] Analytics, tag manager and consent behave as decided, tested on the preview with the opt-in flag.
- [ ] Forms, scheduling widgets, chat and other integrations work on the preview.
- [ ] Domain verification for search consoles survives the DNS change (DNS record rather than a meta
      tag or file on the old host).
- [ ] The production domain is added to the deployment platform and its certificate is ready.
- [ ] A rollback plan exists: the old site stays published and the DNS change can be reverted.

## The switch

- The user points DNS (or the proxy in front of the site) at the new host.
- Remove any translation proxy that served locales before.

## Right after

- [ ] Hosting checks and the search-index check against production.
- [ ] Submit the sitemaps in Search Console (and Bing Webmaster Tools, optionally with IndexNow).
- [ ] Watch the indexing, hreflang and 404 reports for the following weeks; add 301s for new 404s.
- [ ] Compare analytics traffic with the previous weeks.
- [ ] Retire the old platform's subscription only after traffic and indexing are stable.
