# Claude Code Session Brief — Port the Enriched PDP to a Live Sandbox Page

**For:** a Claude Code session in the `trade-build` repo.
**Goal:** turn the reviewed HTML prototype into a live, shareable product page on
the sandbox store (`society6-design-lab.myshopify.com`) for one product —
**Cat Landscape 154 Framed Canvas Print**.
**Design reference:** `enriched-pdp.html` (the standalone prototype — attached to
the Shopify Sandbox Project, or copy it into the repo as a reference file).

This brief is the handoff from the Project (planning) to Claude Code (execution).
Read it fully before starting. Work the steps in order — step 1 and 2 gate the rest.

---

## Context — what's being built and why

The current sandbox PDP is Shopify's stock Horizon template: title, price, frame/
size selectors, catalog-wide boilerplate description, a near-empty "About This
Piece" section. It carries **no artwork-specific descriptive content** — nothing
about what the image depicts, its style, mood, or palette.

The product carries six **Cowork metadata fields** (vision-generated). The job is
to render a PDP that layers that metadata into the page as: a curated Aesthetic
block, a filled "About This Piece" section, `Product` JSON-LD with
`additionalProperty`, and discovery links — making the page legible to shoppers,
search crawlers, and AI answer engines at once.

The prototype already encodes every design decision. This session is a **port**,
not a redesign. Match the prototype.

### Approach: Option A — single-product alternate template

Create an **alternate product template** and assign it to Cat Landscape 154 only.
Every other product on the store keeps the stock template, untouched.

- Most authentic demo — it's a *real* product page at a real product URL.
- Lowest blast radius — one product changes.
- Most portable — production would do exactly the same thing (Principle 2).

Do NOT build this as a standalone Shopify "page" — a page lacks native product
context and would force bespoke data wiring (works against portability).

### No scraping is involved

All data is already in the sandbox store. Base product content (title, price,
images, variants) and the six Cowork metafields are read directly from Shopify via
the standard product object and metafield references. Claude Code, authenticated
to the sandbox, has full access. Nothing is fetched from the public web.

---

## Step 1 — Inventory the sandbox product (do this first)

Before writing any template code, list everything currently on Cat Landscape 154
in the sandbox store:

- Title, price, product type, vendor/artist field
- Body description (the `product.description` / body_html)
- Images — how many, what they show
- Variants — frame and size options
- All metafields — names, **namespaces, keys**, and current values

Output a short inventory. This tells us what base content is real vs. missing.

## Step 2 — Resolve content gaps

The prototype currently shows artist bio, artist image, specifications, and
shipping copy. Some of that was placeholder content borrowed from the live
society6.com page for the standalone mockup — it may or may not exist on the
sandbox product. For each piece of content the prototype displays, classify it:

**Bucket A — template-static.** Content that is genuinely identical for every
canvas print: the Specifications bullets, the Shipping & Returns copy. This can
live as static text in the template. That is honest — it *is* catalog-wide
boilerplate, not per-product data.

**Bucket B — populate-the-product.** Content that is genuinely per-product and
should come from Shopify: artist name, artist bio, artist image, the product
images. If any of this is missing from the sandbox product, the fix is to enter
it once, by hand, into the product in Shopify admin — a five-minute task. Not a
code task, not scraping.

Decision rule: if it differs product-to-product, it belongs on the product
(Bucket B). If it's the same everywhere, it can be template-static (Bucket A).

Resolve every gap before porting, so the template is built against real data.

## Step 2a — Known-missing content: classification and instructions

Three sections the prototype displays are confirmed NOT present on the sandbox
product pages (verified by the PM). Do not treat these as "discover and decide" —
the classification and instruction for each is given below. Critically: in every
case the template must **degrade gracefully** — it must never render an empty
section, an empty shell, or placeholder text.

**Specifications — Bucket A, template-static.**
Genuinely identical for every canvas print. Carry the spec bullets as static text
in the template. Because the template supplies them, they are always present — no
conditional needed. Document in a comment that this is deliberate catalog-wide
boilerplate, not per-product data. (If production later wants per-product specs,
this becomes a metafield — note it, don't build it.)

**Shipping & Returns — Bucket A, template-static.**
Same as Specifications. Static text in the template, always present, commented as
deliberate boilerplate.

**Meet the Artist — Bucket B, per-product, with graceful degradation.**
Architecturally this is per-product data (every artist differs) and must be read
from the product's artist/vendor fields and an artist-bio/-image source — never
hardcoded, never pulled from the prototype's placeholder text.
- **Recommended:** hand-enter ilovedoodle's bio and artist image onto the Cat
  Landscape 154 product in Shopify admin (a five-minute task) so the demo is
  authentic and the template can be built reading real data.
- **Required regardless:** the template must wrap the Meet the Artist block in a
  presence check — `{% if <artist bio> != blank %}` render it, `{% else %}` omit
  the section entirely. No empty "Meet the Artist" heading, no placeholder bio.
  The page must be clean and complete whether or not the artist data exists.

### Graceful degradation — applies to the whole template

Every data-driven block is conditional and hides cleanly when its data is absent:
the Aesthetic block, each Cowork attribute row, About This Piece, and Meet the
Artist. The only always-present sections are the Bucket A static ones
(Specifications, Shipping & Returns) because the template itself supplies them.

The test of correctness: the template, run against a product with NO Cowork
metadata and NO artist bio, must still produce a clean, complete, functional PDP —
just a simpler one. Build and test that absent state, not only the fully-populated
one. This is what makes the template valid across the whole catalog (Principles
1 and 6), not just on enriched demo products.

## Step 3 — Confirm the six metafield identifiers

From the Step 1 inventory, record the exact `namespace.key` for each of the six
Cowork fields. Shopify admin shows the field *labels*; Liquid needs the
*identifiers*. Expected mapping (CONFIRM the namespace/keys against the store —
the labels are known, the identifiers are not):

| Shopify field label | Prototype data-object name | Contents |
|---|---|---|
| Cowork Keywords | `subjectTags` | 18-tag descriptive string |
| Cowork Mood     | `mood`        | whimsical\|playful\|serene\|elegant |
| Cowork Palette  | `palette`     | orange\|terracotta\|black\|white\|monochrome\|warm |
| Cowork Style    | `style`       | line-art\|minimalist\|illustration\|hand-drawn\|modern\|whimsical\|graphic |
| Cowork Subject  | `category`    | animal\|abstract |
| Cowork Summary  | `visualDescription` | the one-sentence visual description |

Note the two naming mismatches explicitly in template comments so no future
engineer is confused: Shopify's "Cowork Keywords" is the prototype's `subjectTags`;
Shopify's "Cowork Subject" is the prototype's `category`.

All six are "Single line text" type — plain strings, pipe-delimited. Split on `|`
in Liquid. No JSON parsing.

## Step 4 — Port the prototype to a Liquid template

Translate `enriched-pdp.html` into the alternate product template. The prototype's
hardcoded `PRODUCT` data object becomes live references:

- Title, price, images → standard `{{ product.* }}` references.
- Six Cowork fields → `{{ product.metafields.<namespace>.<key> }}`, each `split`
  on `|`.
- Artist name / bio / image → per Step 2a: read from product/artist data, wrapped
  in a presence check; never hardcoded from the prototype's placeholder text.
- Specifications and Shipping & Returns → per Step 2a: template-static text.
- Keep all `s6cw-` CSS prefixes exactly — they exist to avoid theme collisions.
- Keep the design as-is: refined retail minimalism, mobile-first, four collapsible
  content sections (About This Piece + Meet the Artist open by default;
  Specifications + Shipping & Returns collapsed). The accordion is the SEO/AI-safe
  pattern — all panel content rendered into the DOM on load, CSS collapses it, JS
  only toggles a class. Preserve that. Do not lazy-load panel content.

### Curation logic (Principle: restraint over completeness)

The prototype renders ~8 chips, not all ~35 tags. Replicate in Liquid:

- **Style** → first 3: Line Art, Minimalist, Hand-Drawn
- **Mood** → Whimsical, Serene (skip playful/elegant as near-duplicates)
- **Subject chips** → Cat, Geometric, Mid-Century Modern (from Cowork Keywords)
- **Palette** → swatches for the literal colors (orange, terracotta, black, white);
  "monochrome / warm" rendered as the inline caption, not chips
- The full strings stay available in the metafields; display is the curated subset.

### Label like a human, store like a machine (Principle 7)

Every visible chip is title-cased: `line-art` → "Line Art", `mid-century modern`
→ "Mid-Century Modern". A shopper never sees a raw database value.

### JSON-LD from real data (Principles 3 + 4)

Render the `Product` JSON-LD block from Liquid, driven by the same metafields.
`additionalProperty` must contain exactly the chips/swatches that are visible —
no more, no less. Consistency rule: if it's in the schema it's on the page, and
vice versa.

### Graceful degradation (Principles 1 + 6)

The full graceful-degradation rule is specified in Step 2a and applies here without
exception. In the Cowork enrichment blocks specifically: each block and each
attribute row checks its metafield — `{% if product.metafields.<ns>.<key> != blank %}`
— and hides cleanly when absent. No empty chip strips, no empty "About This Piece",
no empty Aesthetic shell. Build and test the absent state, not just the populated one.

### About This Piece — two states

The prototype defines two states; keep both:
- **Artist wrote a piece description** → their words lead, attributed to them.
- **Artist silent (our case)** → Cowork Summary fills the section, with the quiet
  note "Visual description generated by Society6 AI from the artwork."

## Step 5 — Deploy

1. Commit the new template + any section files to `trade-build`. One clear commit
   message (one-commit-per-session standard).
2. Push to GitHub. Note: `trade-build` does not have the post-commit auto-push
   hook — push manually, or install the hook (see operating model).
3. In Shopify admin, assign the alternate template to Cat Landscape 154 only.
4. Confirm the live sandbox URL renders correctly. Test mobile viewport.

---

## Notes and caveats

- **Sharing.** The sandbox is a password-protected dev store. A shareable URL still
  requires the store password. Fine for stakeholders and colleagues (one shared
  password). It is NOT a fully public link. If a truly public demo is needed later,
  hosting the standalone `enriched-pdp.html` on Netlify is the better route for
  that purpose.
- **Production path.** On production, society6.com is itself a Shopify store —
  base product content is already in its database and a production template reads
  it the same way. The Cowork metadata for production would be written into
  production metafields by the same pipeline that populated the sandbox's ~200
  products. The team likely plans to do this — **confirm with the team** whether
  that production-metadata pipeline is planned/resourced. That, not scraping, is
  the real production dependency to track.
- **Optional housekeeping.** Per the operating model, `trade-build` does not yet
  have a `CLAUDE.md` referencing the universal layer. Adding it is a small,
  worthwhile follow-up but is optional for this session — don't let it expand
  scope.
- **Variants.** Frame/size selectors are shown visually but not wired to working
  price/image logic. That matches the prototype and the agreed scope. Keep them
  static unless variant logic becomes a deliberate later task.

---

## Appendix — prototype build history (folded-in build notes)

The prototype `enriched-pdp.html` reflects these decisions, made and reviewed
across the planning sessions:

- **Curation cut** — ~8 visible chips from ~35 tags. Vision Principle 2; rendering
  all 35 is the chip-cloud failure mode.
- **Palette** — swatches for 4 literal colors; "monochrome/warm" as a caption,
  since those are palette characteristics, not colors.
- **About This Piece attribution** — Option 1: quiet "Visual description generated
  by Society6 AI" note. Honest-AI (Principle 8). Confirmed by John. "Design notes"
  editorial framing was rejected as less honest (implies a human editor).
- **Aesthetic block placement** — in the buy column, not collapsed; it supports
  the buy decision and is compact.
- **Collapsible sections** — added after review. Four labelled sections; About This
  Piece + Meet the Artist open by default, Specifications + Shipping & Returns
  collapsed. SEO/AI-safe pattern confirmed: content is in the DOM on load, only
  visibility toggles. "Meet the Artist" is the label the artist content
  originally lacked.
- **Aesthetic block tightening** — done after review to bring "About This Piece"
  sooner in mobile scroll: reduced padding, tighter rows, smaller swatches, palette
  note moved inline. All 8 chips retained — a compaction, not a re-cut.
- **Aesthetic direction** — refined retail minimalism, deliberately not a
  maximalist showpiece, because the page must port cleanly into a real theme
  (dev Principle 2).
- **Variant selectors** — visible, not wired. Agreed scope.

### Known production gaps carried into this session
- Chip hrefs are placeholders → need real Society6 collection URLs. Some map to
  existing collections (e.g. line-drawing art prints); mood-based ones may not
  exist as collections yet — a discovery-taxonomy question.
- "More like this" is a stub → production needs a real metadata-similarity query,
  with category/artist fallback.
- A second sample product *with* a real artist description would let the
  "artist-present" state of About This Piece be demonstrated, not just coded.
