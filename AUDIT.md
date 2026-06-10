# TDR Academy Website — Audit & Improvement Tracker

Deep analysis performed 2026-06-10 (full read of both pages + live browser verification of every finding).
P0 items were fixed and verified the same day. Remaining items are open, ordered by priority.

## Site facts

- Pure static frontend: `index.html`, `donors.html`, `TDR_Logo.png`. No build step.
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

## 🔴 P1 — high value, open

- [ ] **Forms have zero `<label>` elements** (placeholder-only). Add labels (visually-hidden is fine). WCAG 3.3.2; placeholders vanish on input and break autofill/screen readers.
- [ ] **Closed mobile menu is `aria-hidden="true"` but its 8 links stay keyboard-focusable** (only translated off-screen). Add `visibility:hidden` when closed (with transition handling).
- [ ] **Modal focus management** — move focus into modals on open, trap it, restore on close (enroll, confirm, tuition, calendar, give).
- [ ] **donors.html head is missing canonical, all Open Graph tags, and favicon** (index has them). Add.
- [ ] **No `og:image`/`twitter:image` on either page** — shared links render with no image. Create a 1200×630 share image.
- [ ] **No JSON-LD structured data** — add `EducationalOrganization`/`School` schema with address (3057 Curry Ford Rd, Orlando FL 32806), phone, geo, opening hours. Highest-leverage local SEO item.
- [ ] robots.txt + sitemap.xml.
- [ ] **EmailJS hardening**: enable domain allowlist in EmailJS dashboard; add a honeypot field to both forms. Quota burn from bots = real enrollment leads silently failing.
- [ ] Footer "News & Events" links to `tdracademy.org/news` which is not part of this repo — confirm it exists on the production host or remove.

## 🟡 P2 — open

- [ ] `prefers-reduced-motion` media query — disable particles, blob/drip animations, reveals, smooth scroll.
- [ ] Pause the hero canvas rAF loop when the hero is scrolled off-screen (IntersectionObserver) — it currently renders forever; battery cost on mobile.
- [ ] Small-text contrast failures: gold `#C8861A` (~3.0:1) and `#888888` (~3.4:1) on `#FAFAF8` are used for 10–11px labels — below 4.5:1 AA. Owner kept the original palette (see below), so the in-palette fix if/when compliance matters: darken small gold labels toward `#8A6722` and small grays toward `#6E6E6E`, or bump those sizes — do NOT recolor display-size gold.
- [x] **Brand decision (2026-06-10):** full migration to the document-rebrand tokens + Fraunces was built, previewed in-browser, and **declined — owner prefers the site's original look** (Cormorant Garamond + `#C8861A` gold + `#FAFAF8` canvas). Do not re-propose. The one adopted piece: all gold-fill buttons now use the shared CTA pattern `#A8762E` fill + `#FAFAF8` text, hover `#B8893D` (commit `fc067bf`) — matches email/docs CTAs. Canonical cross-media tokens live in `BRAND_GUIDE.md` (`..\TDR Docs Claude\`); the site keeps its own digital dialect deliberately.
- [ ] Extract shared `styles.css` + `site.js` — the two pages duplicate ~70% of CSS and have already drifted (e.g. `--gold-h` exists only in index).
- [ ] Calendar events are injected with `innerHTML` — switch summary/desc/time to `textContent` (anyone with write access to the Google Calendar can inject HTML/script).
- [ ] Trim Google Fonts to the weights actually used (currently 9 variants across 2 families). Remove the no-op `@font-face { font-display: swap; }` rule.
- [ ] MOCK calendar events are hardcoded for the 2026–27 school year — will go stale silently.

## 🟢 P3 — polish

- [ ] SRI (`integrity` attr) + pinned version for the EmailJS CDN script.
- [ ] Auto-update the © year.
- [ ] `aria-hidden` on decorative contact emoji icons; footer `h5` → `h3`; hours table day cells → `th scope="row"`.
- [ ] Optimize TDR_Logo.png further (25KB at 300×224; could be ~8–10KB resized/WebP with PNG fallback).
- [ ] Donors "Give Now" buttons declare a background transition but have no hover state.
- [ ] Content check: "100% of donations go directly to student support" / "no overhead" are absolute claims — confirm accuracy with TDR Ministries before donors do.

## 💡 Feature backlog (all frontend-only, ordered by expected value)

1. **Photo gallery / virtual tour** — site has zero photos of the school; parents choose with their eyes. Static images + lightbox, lazy-loaded.
2. **FL School Choice eligibility mini-wizard** — 3 questions → "you likely qualify" + links to FES/FTC applications. Converts the "$0 out-of-pocket" pitch into action.
3. **FAQ accordion** + `FAQPage` JSON-LD — deflects office phone calls, earns rich results.
4. **EN/ES language toggle** — static JSON dictionary; ESL support is already advertised.
5. **Testimonials section** — 3–5 parent quotes, static.
6. **Tuition estimator** — out-of-pocket after scholarships, from the existing fee table.
7. **Add-to-calendar (.ics) buttons** on calendar events — generated client-side.
8. **Google Maps embed** in contact (basic iframe embed needs no API key).
9. **Announcements banner** driven by the existing Google Calendar or a JSON file — office can update without code.
10. **Sticky mobile enroll CTA.**

## Hosting note

Repo is private; GitHub Pages free tier requires a public repo. Netlify / Vercel / Cloudflare Pages deploy private repos free, allow custom headers (caching, CSP), and Netlify Forms could replace EmailJS entirely while staying fully static.
