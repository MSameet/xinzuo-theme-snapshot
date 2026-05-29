# Xinzuo Theme — Branch Change Log

**Branch:** `main` (9 commits after initial theme import)  
**Date range:** 2026-05-25 → 2026-05-29  
**Baseline:** `4f6cdde` — initial Horizon-based Xinzuo theme snapshot

---

## What I picked

**Unify the storefront around a single glass-panel / bento visual system, backed by one shared product-card component — then start the highest-leverage performance work (Swiper gating + LCP cleanup).**

Concretely, the work clusters into three bets:

1. **Design system** — Replace fragmented section-specific card layouts with `xz-product-card`, glass styling in `site-overrides.css`, and consistent button treatment across homepage, collections, reviews, blogs, and bundle builder.
2. **Template hygiene** — Remove ~30 duplicate per-series product/collection JSON templates and restore a working default `collection.json` so `/collections/*` routes render correctly.
3. **Performance (in progress, not yet committed)** — Stop loading Swiper (~160 KB JS/CSS) on every page; deduplicate PDP LCP preloads; load `site-overrides.css` non-blocking.

---

## Why it's the highest-impact thing here

| Area                         | Impact                                                                                                                                                                                                                                                                    |
| ---------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Shared product cards**     | Homepage sliders, collection grids, bundle builder, and cart upsells all show products. One card snippet (`snippets/xz-product-card.liquid`) + one CSS layer (`assets/site-overrides.css`) fixes visual drift everywhere at once instead of patching sections one-by-one. |
| **Template pruning**         | Deleted templates were ~13,600 lines of near-duplicates. Fewer templates = less editor confusion, faster theme maintenance, and no broken collection URLs when a series-specific template was removed but not replaced.                                                   |
| **Homepage catalogue bento** | “Shop by category” is a primary navigation surface. Bento + glass panels make hierarchy scannable on desktop and tablet without separate mobile/desktop markup paths.                                                                                                     |
| **Reviews / social proof**   | Glass cards + capped height keep Judge.me content on-brand; looping `best-reviews` removes pagination friction on the homepage.                                                                                                                                           |
| **Swiper conditional load**  | Swiper was global in `layout/theme.liquid` but only used on ~8 surfaces. Gating saves ~160 KB on collection pages, static pages, and search — the highest-traffic pages that previously paid for an unused carousel library.                                              |
| **LCP preload dedup**        | Two competing PDP image preloads (path-based + template-based) wasted bandwidth and delayed LCP on Slow 4G traces documented in theme comments.                                                                                                                           |

---

## What I did

### Committed (chronological)

#### 1. `6f33261` — Unify button styles and prune unused templates (2026-05-25)

- Removed **29 product templates** (per-series duplicates: `mo-series`, `retro-series`, `supreme-series`, etc.) and **9 collection templates** (`knives`, `bundles`, `accessories`, etc.).
- Updated `sections/main-collection.liquid` mobile ATC button handling.
- Tweaked button classes in `best-reviews`, `bundle-intro`, `cw-hero`, `home-products`, `homepage-catalogue`.
- Adjusted `templates/index.json` and `templates/page.reviews.json` section ordering.
- Cleaned app embed blocks from `config/settings_data.json`.

**~13,610 lines removed**, ~94 added.

---

#### 2. `3e2388a` — Redesign homepage catalogue as bento grid with glass-panel cards (2026-05-25)

- Rewrote `sections/homepage-catalogue.liquid` (~248 lines changed).
- Desktop: asymmetric bento grid with glass-panel cards and hover states.
- Mobile/tablet: stacked card layout with shared glass styling.

---

#### 3. `394a685` — Unify mobile/tablet catalogue cards; add 2-column tablet grid (2026-05-25)

- Refined responsive breakpoints in `homepage-catalogue.liquid`.
- Added 2-column tablet grid (750–1024px).
- Updated `templates/index.json` section settings.

---

#### 4. `1467c77` — Restyle reviews section with glass cards and 100vh cap (2026-05-25)

- Restyled `sections/reviews.liquid` with glass-card treatment.
- Capped section height at `100vh` to prevent unbounded scroll on review-heavy pages.

---

#### 5. `816bf64` — Remove reviews pagination and add best-reviews loop (2026-05-25)

- Removed pagination UI from `sections/reviews.liquid`.
- Expanded `sections/best-reviews.liquid` with continuous Swiper loop for homepage social proof.
- Minor `templates/index.json` tweak.

---

#### 6. `899efaf` — Update blog cards, homepage guides, and hero (2026-05-25)

- **`sections/main-blog.liquid`** — Simplified listing layout; aligned card styling with site glass language.
- **`sections/main-blog-post.liquid`** — Expanded post layout (+176 lines).
- **`sections/homepage-guides.liquid`** — Updated guide cards to match homepage visual system.
- **`sections/cw-hero.liquid`** — Small hero tweak.
- **`templates/index.json`** — Section config updates.

---

#### 7. `9cdf346` — Update product cards with shared home layout and glass cart icon (2026-05-25)

- Major refactor of **`assets/site-overrides.css`** (~1,600 lines reorganized) — glass cart icon, card spacing, typography tokens.
- **`snippets/xz-product-card.liquid`** — Added optional params (`card_id`, loading/fetchpriority/sizes), Judge.me rating parsing, unified markup.
- **`snippets/product-card.liquid`** — Wired to shared layout patterns.
- **`sections/home-products.liquid`** — Slimmed down to use shared card snippet.

Before/after screenshots captured: `shop_by_category-*.png`, `products_page-*.png`, `chef-review-*.png`.

---

#### 8. `31d6fc1` — Fix collection page routing and image URL guards (2026-05-26)

- **Restored `templates/collection.json`** — Default collection template deleted in commit 1; collections were 404 or falling back incorrectly.
- **`sections/cw-hero.liquid`** — Guarded `<picture>` sources when `desktop_img` / `mobile_img` / `middle_desktop_img` are blank (prevents Liquid `image_url` errors). Mobile hero height set to `40vh`; `object-position: bottom`.
- **Footer/header links** — Updated `cw-footer2`, `cw-footer3`, `header-drawer` collection URLs.
- **`templates/index.json`** — Homepage section/link fixes.

---

#### 9. `368bc01` — Add bundle builder search bar and align cards with xz-product-card layout (2026-05-29)

- **`sections/bundle-builder.liquid`** — Search/filter bar for bundle product picker.
- **`assets/bundle-builder.css`** + **`assets/bundle-builder.js`** — Search UX, card layout aligned with `xz-product-card`.
- **`sections/product-info.liquid`** through **`product-info7.liquid`** — Minor alignment tweaks.
- **`templates/page.bundle-builder.json`** — Section settings.

Before/after: `bundle_before.png`, `bundle_after.png`.

---

### Uncommitted (working tree — 2026-05-29)

These changes are **not yet in git** but represent the current performance batch:

| File                            | Change                                                                                                                                                                                                                                                                                                                |
| ------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `layout/theme.liquid`           | `needs_swiper` flag — eager Swiper on index, product, cart, list-collections, bundles/reviews suffixes, and theme editor; lazy loader elsewhere. Removed duplicate path-based PDP preload; consolidated single LCP preload using first gallery media. `site-overrides.css` switched to preload/onload (non-blocking). |
| `snippets/swiper-assets.liquid` | **New** — conditional eager Swiper CSS (preload) + deferred JS.                                                                                                                                                                                                                                                       |
| `snippets/swiper-loader.liquid` | **New** — `window.xzLoadSwiper()` promise-based lazy loader for pages without eager assets.                                                                                                                                                                                                                           |
| `snippets/cart-drawer.liquid`   | Cart recommended-products slider calls `xzLoadSwiper()` before init/reinit so upsells work on collection/static pages without global Swiper.                                                                                                                                                                          |
| `assets/site-overrides.css`     | Glass cart icon button background: `rgba(255,255,255,0.05)` → `rgba(0,0,0,0.5)` for contrast.                                                                                                                                                                                                                         |
| `PERFORMANCE-TODO.md`           | Audited backlog; P0 item #1 (Stop loading Swiper on every page) marked partially complete.                                                                                                                                                                                                                            |
| `.gitignore`                    | Added `*.md` (ignores markdown from git).                                                                                                                                                                                                                                                                             |

Screenshot evidence: `blogs_before.png` / `blogs_after.png`, and others listed above.

---

### Files touched (summary)

**Sections:** `homepage-catalogue`, `home-products`, `reviews`, `best-reviews`, `main-blog`, `main-blog-post`, `homepage-guides`, `cw-hero`, `main-collection`, `bundle-builder`, `product-info` (1–7), footers, headers  
**Snippets:** `xz-product-card`, `product-card`, `cart-drawer`, `swiper-assets`, `swiper-loader`, `header-drawer`  
**Assets:** `site-overrides.css`, `bundle-builder.css`, `bundle-builder.js`  
**Templates:** pruned 38 JSON templates; restored `collection.json`; updated `index.json`, `page.reviews.json`, `page.bundle-builder.json`  
**Layout:** `theme.liquid`

---

## What I'd do next

### Immediate (commit + verify)

1. **Commit the Swiper gating work** — `theme.liquid`, `swiper-assets.liquid`, `swiper-loader.liquid`, `cart-drawer.liquid`, and the cart icon CSS fix.
2. **Lighthouse before/after** — Run on homepage, one PDP (`/products/zhen-xz05-series-8-inch-chef-knife`), and `/collections/all`. Confirm Swiper absent on collection; confirm cart drawer upsell slider still initializes via `xzLoadSwiper()`.
3. **Decide on `.gitignore`** — `*.md` will exclude `note.md` and `PERFORMANCE-TODO.md` from git; narrow to specific paths if docs should be tracked.

### P0 performance (from `PERFORMANCE-TODO.md`)

4. **Migrate Swiper sections to Horizon native slideshow** — `collection_slider`, `home-slider`, etc. Eliminates Swiper entirely (~160 KB + duplicate init logic).
5. **Gate theme JS modules per template** — Load `media-gallery.js`, `variant-picker.js`, `quick-add.js` only where needed (~20 ES modules currently global).
6. **Collection LCP** — Preload first product image on collection pages (already partially in `theme.liquid`; verify sizing matches grid).

### Design / UX follow-ups

7. **Extend `xz-product-card` to remaining Horizon `product-card` call sites** — Search results, quick-add drawer, cart line items if not already covered.
8. **Reviews page** — Validate glass cap + loop behavior with real Judge.me widget load; test on slow connections.
9. **Bundle builder** — Test search with large catalog; confirm filter debounce and empty states.
10. **Blog post template** — Visual QA on long-form posts with images and embeds after `main-blog-post.liquid` expansion.

### Housekeeping

11. **Remove screenshot PNGs from repo root** or move to a `docs/` folder before production deploy.
12. **Theme editor smoke test** — `needs_swiper = true` in `design_mode` ensures sliders preview correctly in Shopify admin.

---

## Commit index

| Hash      | Date       | Message                                                                   |
| --------- | ---------- | ------------------------------------------------------------------------- |
| `4f6cdde` | —          | initial theme                                                             |
| `6f33261` | 2026-05-25 | Unify button styles and prune unused templates                            |
| `3e2388a` | 2026-05-25 | Redesign homepage catalogue as bento grid with glass-panel cards          |
| `394a685` | 2026-05-25 | Unify mobile/tablet catalogue cards; add 2-column tablet grid             |
| `1467c77` | 2026-05-25 | Restyle reviews section with glass cards and 100vh cap                    |
| `816bf64` | 2026-05-25 | Remove reviews pagination and add best-reviews loop                       |
| `899efaf` | 2026-05-25 | Update blog cards, homepage guides, and hero                              |
| `9cdf346` | 2026-05-25 | Update product cards with shared home layout and glass cart icon          |
| `31d6fc1` | 2026-05-26 | Fix collection page routing and image URL guards                          |
| `368bc01` | 2026-05-29 | Add bundle builder search bar and align cards with xz-product-card layout |
