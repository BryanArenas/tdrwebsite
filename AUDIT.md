# TDR Academy Website — Audit & Improvement Tracker

Deep analysis performed 2026-06-10 (full read of both pages + live browser verification of every finding).
P0 items were fixed and verified the same day. Remaining items are open, ordered by priority.

## Site facts

- Pure static frontend: `index.html` (English), `es.html` (Spanish), `donors.html`, `TDR_Logo.png`. No build step.
- **Bilingual (EN/ES):** `es.html` is a full Spanish translation of `index.html`. A tiny inline `lang-detect` script in each `<head>` sends `navigator.language`-Spanish visitors to `es.html` on first visit; a manual EN/ES toggle (top bar on desktop, slide-out menu on mobile) persists the choice in `localStorage` (`tdr_lang`) and overrides detection. `hreflang` alternates + per-page canonicals are set and both URLs are in `sitemap.xml`. **Maintenance:** the two files are independent copies — any structural/JS/content change to `index.html` must be mirrored into `es.html` (and re-translated). `donors.html` is English-only by current scope.
- Email via EmailJS (service `service_ijvuwox`; templates `template_myiat8i` enroll, `template_7rry6pf` contact; sends to Hello@tdracademy.org). Public key in source is by design.
- Events via Google Calendar API. The API key in source is **verified referrer-restricted** (returns 403 `API_KEY_HTTP_REFERRER_BLOCKED` without an allowed referrer) — safe to keep in source, but the allowlist must be updated if the hosting domain ever changes. Side effect: on non-allowlisted domains (incl. localhost) the calendar uses the MOCK fallback events.
- Donations link out to Tithely (no payment code on site).

## ✅ P0 — fixed 2026-06-10 (verified in browser)

- [x] `showEnrollConfirmation()` was called on enrollment success but never defined — every successful submission threw, which triggered the mailto fallback and showed a false "EmailJS not yet configured" error (email had actually sent). Defined the function, moved the orphaned `#confirmOverlay` markup above the scripts (it sat after them, so it could never have been wired), wired OK button / overlay click / Escape / 8s countdown.
- [x] Calendar fetch ignored HTTP errors — a 403/quota response rendered an empty event list still labeled "Live — Google Calendar". Now checks `response.ok` + `data.error` and falls back to MOCK events.
- [x] Stray orphaned `×` button rendered visibly at the bottom of donors.html — removed leftover markup.
- [x] "For Students" linked to nonexistent `#students` in 3 places — now `#curriculum`. (If a student portal link is wanted instead, FACTS SIS has a student login — same URL as parents.)
- [x] 9th–12th Grade options duplicated in the first student's grade dropdown (18 options → 14).
- [x] Mobile "Enroll Now" was `<a href="#">` — jumped page to top behind the modal. Now a `<button>`. In-page mobile menu links also close the menu now.
- [x] Logo embedded as ~33KB base64 twice per page (byte-identical to the unused `TDR_Logo.png`) — replaced all 4 embeds with the file reference. index.html 158.7KB → 91.6KB, donors.html 105.7KB → 38.6KB, and the logo now caches across pages.
- [x] Misleading enrollment error message — now says "Something went wrong sending your request" instead of blaming configuration.

## ✅ P1 — fixed 2026-06-10, second pass (verified in browser)

- [x] Form fields now all have accessible names via `aria-label` (both forms + the dynamically-added student rows). Verified zero unlabeled fields.
- [x] Closed mobile menu gets `visibility:hidden` (with a transition delay so the slide-out still animates) — links no longer keyboard-reachable while "closed".
- [x] Modal focus management for all five dialogs (enroll, confirm, tuition, calendar drawer, donors give): focus moves to the dialog's close/primary control on open, Tab/Shift+Tab wrap inside, and focus restores to the opener on close. Verified live including the Tab-wrap.
- [x] donors.html head: canonical, full OG set, twitter card, theme-color, favicon.
- [x] `og:image`/`twitter:image` on both pages → `og-image.png` (1200×630, generated on-brand: logo + "Rooted in faith. / Growing leaders." + eyebrow). `summary_large_image` cards.
- [x] JSON-LD `School` schema on index (address, phone, email, hours, sameAs, slogan, logo). Geo coords deliberately omitted rather than guessed.
- [x] robots.txt + sitemap.xml.
- [x] Honeypot field on both forms — bots that fill it get a fake success and nothing is sent (verified: confirmation shows, no network call). **Still Bryan's action: enable the domain allowlist in the EmailJS dashboard** (Account → Security) — can't be done from code.
- [x] Footer "News & Events": live `/news` confirmed **404** — the link now opens the Events calendar drawer instead.
- [x] Favicon extracted from 6.7KB inline base64 to `favicon.png`, shared by both pages.

## 🟡 P2 — open

- [x] `prefers-reduced-motion` honored on both pages (animations/transitions collapsed, smooth-scroll off, particle canvas skipped entirely).
- [x] Hero canvas pauses when scrolled off-screen (IntersectionObserver) and when the tab is hidden.
- [x] Calendar event text from the Google API now rendered via `textContent` — XSS-probed live: hostile `<img onerror>` title rendered inert, nothing executed.
- [x] EmailJS CDN script pinned to `@4.4.1` with SRI `integrity` + `crossorigin` + `defer` (verified SDK still loads).
- [ ] Small-text contrast failures: gold `#C8861A` (~3.0:1) and `#888888` (~3.4:1) on `#FAFAF8` are used for 10–11px labels — below 4.5:1 AA. Owner kept the original palette (see below), so the in-palette fix if/when compliance matters: darken small gold labels toward `#8A6722` and small grays toward `#6E6E6E`, or bump those sizes — do NOT recolor display-size gold.
- [x] **Brand decision (2026-06-10):** full migration to the document-rebrand tokens + Fraunces was built, previewed in-browser, and **declined — owner prefers the site's original look** (Cormorant Garamond + `#C8861A` gold + `#FAFAF8` canvas). Do not re-propose. The one adopted piece: all gold-fill buttons now use the shared CTA pattern `#A8762E` fill + `#FAFAF8` text, hover `#B8893D` (commit `fc067bf`) — matches email/docs CTAs. Canonical cross-media tokens live in `BRAND_GUIDE.md` (`..\TDR Docs Claude\`); the site keeps its own digital dialect deliberately.
- [ ] Extract shared `styles.css` + `site.js` — the two pages duplicate ~70% of CSS and have already drifted (e.g. `--gold-h` exists only in index).
- [ ] Font-weight trim — checked: all 9 loaded variants are actually used; only possible saving is CG italic-400, not worth the breakage risk. (The no-op `@font-face` rule was removed.)
- [ ] MOCK calendar events are hardcoded for the 2026–27 school year — will go stale silently. Live calendar is primary, so low urgency.

## 🟢 P3 — polish

- [x] SRI + pinned version for the EmailJS CDN script (done with the P2 batch).
- [x] © year auto-updates on both pages.
- [x] Decorative emoji `aria-hidden`; footer `h5` → `h3`; hours table day cells → `th scope="row"` (with matching CSS).
- [x] Donors "Give Now" buttons now have a hover state (`#B8893D`).
- [ ] Optimize TDR_Logo.png further (25KB at 300×224; could be ~8–10KB resized/WebP with PNG fallback).
- [ ] Content check: "100% of donations go directly to student support" / "no overhead" are absolute claims — confirm accuracy with TDR Ministries before donors do.

## 💡 Feature backlog (all frontend-only, ordered by expected value)

1. ✅ **Photo gallery** — built 2026-06-10. "Campus Life" section between Curriculum and Admissions; lazy-loaded 4:3 grid + full lightbox (arrows, swipe, Escape, focus trap). Section and nav links auto-hide while photo list is empty, so it deploys safely. Bryan adds photos per `gallery/README.md` (1600px long edge, JPEG ~78%, 150–400KB; media-release + no-names privacy rules).
2. ✅ **FL School Choice eligibility wizard** — built 2026-06-10. "Scholarships" section before Contact: 3 questions (residency / grade / IEP), four result paths (qualify, qualify+FES-UA, non-FL → tuition modal, pre-K), CTAs to Step Up For Students + enrollment modal, honest state-determines-awards disclaimer, nothing saved or sent. Tuition-modal callout now links to it. All paths browser-verified.
3. **FAQ accordion** + `FAQPage` JSON-LD — deflects office phone calls, earns rich results.
4. **EN/ES language toggle** — static JSON dictionary; ESL support is already advertised.
5. **Testimonials section** — 3–5 parent quotes, static.
6. **Tuition estimator** — out-of-pocket after scholarships, from the existing fee table.
7. **Add-to-calendar (.ics) buttons** on calendar events — generated client-side.
8. **Google Maps embed** in contact (basic iframe embed needs no API key).
9. **Announcements banner** driven by the existing Google Calendar or a JSON file — office can update without code.
10. **Sticky mobile enroll CTA.**

## 📍 Local SEO / Google Business rankings (audited 2026-06-10)

On-site technical SEO is in place (titles, descriptions, canonicals, OG/Twitter cards, School JSON-LD,
robots.txt, sitemap). Tightened same day: `School` schema multi-typed `["School","LocalBusiness"]` with
`@id`, `hasMap`, `areaServed`, and directory-profile `sameAs` links; same entity block added to donors.html
(had none); `og:site_name`/`og:locale` on both pages; `lastmod` in sitemap.xml.

**Off-site facts found (live web check):** site ranks #1 for its own name; citations exist on NCES,
GreatSchools, US News, Private School Review, School Choice USA, Niche, Yelp, BBB, Orlando Magazine.
Facebook page has 28 reviews, 100% recommended.

**Only Bryan can do these (no code involved) — ordered by ranking impact:**

1. **Google Business Profile** — search "TDR Learning Academy" in Google Maps. If unclaimed/missing,
   claim at business.google.com. Set: primary category "Private school" (secondary: "Christian school",
   "Elementary school", "High school"), exact NAP `TDR Learning Academy / 3057 Curry Ford Rd, Orlando, FL
   32806 / (407) 694-8595` (match the site — decide once whether "Ste 1" is official and use it everywhere),
   hours incl. Wed early close, website link, enrollment link, upload the gallery photos. GBP is the #1
   driver of map-pack rankings.
2. **Google reviews** — Google reviews (count + recency) are a top local ranking factor; Facebook reviews
   are not. Ask the happiest FB reviewers to repost on Google via the GBP "share review form" short link.
   Steady trickle (2–3/month) beats a one-day burst.
3. **Google Search Console** (search.google.com/search-console) — verify the domain, submit sitemap.xml.
   Same for Bing Webmaster Tools (free, also feeds DuckDuckGo/AI search).
4. **Fix Yelp listing** — currently "TDR ACADEMY", categorized **"Child Care & Day Care"** (wrong). Claim
   at biz.yelp.com, rename to TDR Learning Academy, category Private/Christian school.
5. **Claim Niche + GreatSchools profiles** — Niche shows a D+ (stale/sparse data); claiming is free and
   lets the school correct enrollment, photos, and respond to reviews. Lumos Learning shows "2.4 (29
   reviews)" of unknown provenance — check and dispute if junk.
6. Confirm `foundingDate` (directories say 2012) — can be added to schema once confirmed.

**Reviews on the site:** testimonials section (backlog #5) is the right home for FB review quotes — get
parent permission before republishing names. Do **NOT** add `aggregateRating`/`Review` schema for them:
self-serving review markup on Organization/LocalBusiness violates Google's structured-data guidelines
(ignored at best, manual action at worst).

## 🔧 Issue diagnosis + fixes (2026-09-29)

Owner reported the live site "exhibiting issues" (visual glitch / site down). Full browser test battery
on `main` (EN/ES redirects + toggle, all modals, wizard, gallery, mobile menu, scroll reveals,
reduced-motion, overflow) passed — the repo's code was healthy. Fixed the two real defects found:

- [x] Hero canvas race: the IntersectionObserver could call `draw()` before `start()` initialized
  `particles`, throwing `TypeError … reading 'length'` on nearly every load (self-healed next frame).
  `particles` now initializes to `[]` at declaration. Both pages.
- [x] Language auto-redirect dropped `location.search`/`location.hash` — Spanish-locale visitors opening
  deep links like `/#admissions` landed at the top. Both directions now preserve query + hash.
- [x] es.html head brought up to the SEO standard of index/donors: `og:site_name`, `og:locale` (`es_LA`,
  alternate `en_US`; index got alternate `es_LA`), JSON-LD upgraded to the shared
  `["School","LocalBusiness"]` entity (`@id …#school`, `hasMap`, `areaServed`, directory `sameAs`),
  keeping the Spanish description/slogan + `inLanguage`.
- [x] `_redirects` (Netlify format, inert elsewhere): 301s for legacy URLs Google still indexes from the
  school's previous website platform (`/fostering-success-together`, `/why-private-education`, `/news`).

**Live-site findings that are NOT in this repo's control** (diagnosed via search index + connected tools;
the CC sandbox cannot reach tdracademy.org directly):

- Hosting is external (GitHub Pages off, no deploy workflow; owner believes Netlify — unconfirmed). If a
  deploy doesn't auto-trigger on merge, the site must be re-published manually.
- Google indexes legacy pages (`/fostering-success-together`, `/why-private-education`) from a previous
  site platform — the `_redirects` above absorbs them once live on Netlify.
- GoHighLevel "TDR Academy" location has an untouched stock website funnel ("Educational Tutoring",
  created 2026-08-28, no domain attached) — an abandoned rebuild experiment; not serving the domain, but
  whoever created it may be planning DNS/platform changes. Clarify before anyone repoints the domain.

## Hosting note

Repo is private; GitHub Pages free tier requires a public repo. Netlify / Vercel / Cloudflare Pages deploy private repos free, allow custom headers (caching, CSP), and Netlify Forms could replace EmailJS entirely while staying fully static.
