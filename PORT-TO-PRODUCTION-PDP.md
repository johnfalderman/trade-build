# Enriched PDP — Production Port Note

**Purpose:** Everything an engineer needs to port the sandbox enriched-PDP work to the production `Society6 | Production` theme. Written at the end of the sandbox build, while the assumptions are fresh.

**Sandbox source:** `trade-build` repo · `society6-design-lab.myshopify.com` · theme "Trade Build" (stock Horizon).
**Production target:** society6.com · custom theme "Society6 | Production" (bespoke, not a theme-store theme).

---

## What was built

A Cowork-enriched alternate product template — one product page, one URL, real product data. The page layers six Cowork vision-generated metafields onto the standard Horizon product object as: a curated Aesthetic chip block, a filled "About This Piece" section, `Product` JSON-LD with `additionalProperty`, and visual discovery via swatches/chips. Designed to be legible to shoppers, search crawlers, and AI answer engines at once.

Per Option A in the brief: an alternate product template assigned to **one** product (Cat Landscape 154). Every other product on the store keeps the stock template. Same approach is intended for production.

Files involved:
- `sections/main-product-cowork.liquid` — new (the entire page)
- `snippets/s6cw-titlecase.liquid` — new (chip label title-casing)
- `templates/product.cowork.json` — new (alternate template, assigned to the product in admin)
- `enriched-pdp_2.html` — reference prototype (kept in repo for diff/review)
- `claude-code-session-brief.md` — the session brief that drove this build

---

## Portability tiers

### Tier 1 — ports cleanly (low effort)

The section file is self-contained, uses `s6cw-*` CSS prefixes to avoid collisions, and relies only on standard Shopify primitives (`product` object, `product.metafields`, `image_url`, `money`, `routes`, `form 'product'`). The Liquid logic is not theme-specific. An engineer can drop `sections/main-product-cowork.liquid`, `snippets/s6cw-titlecase.liquid`, and `templates/product.cowork.json` into the production theme and the page will render.

The CSS variables at the top of the section define a self-contained "refined retail minimalism" aesthetic. On the production port, swap those variables for the production design-system tokens. The structure does not change.

### Tier 2 — manual but straightforward

- **Variant selectors.** The Frame and Size chip groups are visible-but-static UI in the prototype. The sandbox product has a single Default Title variant, so static is honest there. In production, real Frame and Size are usually product options — replace the static chip groups with a real variant picker (Horizon's `<variant-picker>` block, or the production theme's equivalent). The CSS classes and layout work as-is.
- **Breadcrumb destination.** The product-type breadcrumb links to `/collections/<handle>`. Confirm the production routing matches.
- **Chip hrefs.** Every chip currently links to `#`. In production each chip should link to its real discovery collection (e.g. Style "Line Art" → `/collections/line-drawing-art-prints`). The mapping is a discovery-taxonomy question, not a template task — list the canonical collections per attribute and pass them as a Liquid lookup table.

### Tier 3 — needs a decision and an owner (the real lift)

#### Artist data source

The Meet the Artist block reads from a four-field `artist` namespace convention defined in the section. As of this build the four fields exist on **only** Cat Landscape 154 (populated by hand for the demo). Every other product hides the Meet the Artist block, the byline, and the JSON-LD `creator` field cleanly.

| Metafield | Sandbox type | Production-recommended type | Purpose |
|---|---|---|---|
| `artist.name` | `single_line_text_field` | same | Display name; byline + JSON-LD `creator` |
| `artist.bio` | `multi_line_text_field` | same | Short bio; also the presence gate for the block |
| `artist.shop_url` | `url` | same | Link to artist's storefront/profile |
| `artist.avatar` | `url` ⚠ | `file_reference` (image) | Square headshot |

**Avatar caveat — flagged for production.** The sandbox stores `artist.avatar` as a `url`-type metafield pointing directly at a society6.com CDN image (e.g. `https://society6.com/cdn/shop/files/ilovedoodle_…jpg`). The template renders the URL as-is, no `image_url` filter. This is **fine for the sandbox demo** but is hot-linking — the image is served from another origin, isn't optimised by Shopify's CDN, and can break if society6.com renames or deletes the file. For production:
- Change `artist.avatar` to `file_reference` (constrained to image types).
- Host the image in Shopify Files (or a controlled CDN bucket the team owns).
- Swap the template's `<img src="{{ artist_avatar | escape }}">` for `<img src="{{ artist_avatar | image_url: width: 120 }}">` so Shopify's image pipeline does the heavy lifting.

**Sandbox demo behavior:** Cat Landscape 154 has all four fields populated. Every other product has none of them — the Meet the Artist block, the byline, and the JSON-LD `creator` are all hidden on those products by the presence checks in the template. This is correct, not a bug.

For production this is a **data dependency, not a template task**. The same kind of pipeline that populates the `cowork.*` vision metafields needs to populate `artist.*` fields per product. Options to evaluate:

1. Source artist data from the existing internal artist database (likely the simplest — these fields already exist somewhere; the question is wiring them onto the product record).
2. Add a parallel pipeline that pulls artist name/bio/avatar/URL by artist handle and writes them as product metafields.
3. Use a Shopify metaobject for artists, with a product → artist reference metafield. Cleaner data modeling, slightly more wiring.

The decision needs an owner. Until that exists, the block will be silent on every product — which is acceptable (clean degradation) but means the page is short of the prototype's intent. **This should be assigned before the production port is scoped.**

#### Cowork metadata pipeline (already-known dependency)

Sandbox carries `cowork.*` metafields on ~200 products, written by an internal vision pipeline. Production needs the same pipeline writing to production metafields. Confirm with the team whether the production-metadata pipeline is planned/resourced. (Carried over from the session brief — this, not scraping, is the real production dependency to track.)

#### "More like this" — real similarity query

The grid is currently a four-card stub with a dashed "Prototype stub — production gap" flag. Production needs a real metadata-similarity query: overlap on `cowork.subject` / `cowork.keywords` / `cowork.style`, with a category/artist fallback. Likely lives upstream of the theme (a backend recommendations endpoint or Search & Discovery boosting), not as Liquid logic.

---

## Specific gotchas (learned during the sandbox build)

- **Metafield types aren't quite what the brief assumed.** The brief described all six Cowork fields as "Single line text — split on `|`". In reality only `cowork.summary` is `single_line_text_field`. The other five are `list.single_line_text_field` whose array carries a single pipe-delimited string. Access pattern in Liquid:
  ```liquid
  {{ product.metafields.cowork.style.value | first | split: '|' }}
  ```
  Documented at the top of the section so the next engineer doesn't get bitten.
- **Two prototype↔Shopify name renames.** The prototype's data-object names don't all match the Shopify metafield keys:
  - prototype `subjectTags` = Shopify `cowork.keywords`
  - prototype `category` = Shopify `cowork.subject`
  Both are commented in the section. Don't rename either side; just be aware.
- **Curation rule.** The prototype renders ~8 chips, hand-curated to skip near-duplicates (e.g. drop "playful" because "whimsical" is already there). The section implements algorithmic first-N curation, which is portable across the catalog but emits a slightly different label set than the prototype on a few products. If production wants the prototype's exact "drop near-duplicates" behavior, either bake the rule into the pipeline (write a `cowork.display_*` subset metafield alongside the raw fields) or write a Liquid rule that consults a synonym map. Pipeline-side is the cleaner long-term answer.
- **JSON-LD consistency.** The `additionalProperty` block is built from the same loop that renders the visible chips, so it always mirrors what's visible. Don't try to also emit a "full" version from the raw metafields — that would violate the brief's consistency rule (schema mirrors page).
- **Single image / one-thumb gallery.** The thumb strip is suppressed when `product.images.size <= 1` so a lonely single thumb never renders. Behavior is correct on multi-image products too.

---

## Out of scope / not addressed

- Wiring variant selectors to real product options (see Tier 2 above).
- Real chip→collection routing (see Tier 2 above).
- Real metadata-similarity query for "More like this" (see Tier 3 above).
- Per-product Specifications. Today these are template-static — genuinely identical for every canvas print, deliberately catalog-wide boilerplate. If production later wants per-product specs (different framing per artwork, etc.), this becomes a metafield-driven block.
- Per-product Shipping & Returns. Same reasoning — template-static.
- A second sample product *with* a real `product.description` written by the artist, to demonstrate the artist-present state of About This Piece. The code path exists; the demo data does not.

---

## Open items to resolve before/during the production port

1. **Assign an owner for the artist-data source** (Tier 3, #1). Sandbox carries hand-populated `artist.*` metafields on **one** product only. Production needs a pipeline (or a one-time backfill from the internal artist database) so the Meet the Artist block populates across the catalog. Also: switch `artist.avatar` from `url` to `file_reference` and host the images properly (see Tier 3 above).
2. **Confirm the Cowork pipeline is planned/resourced for production** (Tier 3, #2). The page is meaningless without the metafields it reads.
3. **Decide the chip→collection routing taxonomy.** Some chips (Style values) map to existing collections; others (Mood values) may not exist as collections yet.
4. **Decide variant strategy** for the production canvas-print template — if the production product has real Frame/Size variants, swap the static chip groups for a real variant picker.
5. **Decide curation strategy** — algorithmic first-N (current), or pipeline-side `cowork.display_*` subset, or a Liquid synonym map.
