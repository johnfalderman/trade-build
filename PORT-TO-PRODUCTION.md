# Trade Landing Page — Production Port Note

**Purpose:** Everything an engineer needs to port the sandbox trade-program work to the production `Society6 | Production` theme. Written at the end of the sandbox build, while the assumptions are fresh.

**Sandbox source:** `trade-build` repo · `society6-design-lab.myshopify.com` · theme "Trade Build" (stock Horizon).
**Production target:** society6.com · custom theme "Society6 | Production" (bespoke, not a theme-store theme).

---

## What was built

A lead-capture layer on the trade-program page, with two mechanisms:

1. **Lookbook modal** — an email-gated pop-up (email required, phone optional) that delivers the 2026 Trade Lookbook PDF. Opens from four entry points via a click-interceptor on any link with `href="#trade-lookbook"`.
2. **Webinar section** — an inline email-gated block delivering an on-demand Demio webinar.

Both use native Shopify `{% form 'customer' %}`. Leads land as tagged Shopify customer records (`trade-catalog-lead`, `trade-webinar-lead`).

Files involved:
- `sections/trade-lookbook-modal.liquid` — new
- `sections/trade-webinar.liquid` — new
- `templates/page.trade-program.json` — modified (CTA swaps + section reorder)
- `trade-webinar-title-card.svg` — co-branded webinar image, in Shopify Files

---

## Portability tiers

### Tier 1 — ports cleanly (low effort)

The two section files (`trade-lookbook-modal.liquid`, `trade-webinar.liquid`) are self-contained, use `s6trade-*` CSS prefixes to avoid collisions, and rely only on standard Shopify primitives. The Liquid logic is not theme-specific. An engineer can drop both into the production theme's `sections/` and they will function.

### Tier 2 — manual but straightforward

- **Page template.** `page.trade-program.json` is the sandbox page's section order. It does **not** port directly — production's trade page is a different template with different sections. The changes must be re-applied by hand to production's actual template:
  - Three CTA swaps → "Download 2026 Trade Lookbook" buttons opening the modal (originally: hero "Learn More", full-service "Request a Curated Collection", final-CTA "Request a Curated Collection").
  - Section reorder: What's Included → testimonials → logo row → Case Studies.
  - The Explore-row "Catalog" card repointed to open the modal (not a direct PDF link).
- **Visual reskin.** The sections inherit the sandbox (Horizon) theme's fonts, spacing, button styles, and color tokens. On production they will work but look off-brand until reskinned to the `Society6 | Production` design system. Estimate: a few hours of front-end work, not a rebuild.

### Tier 3 — needs a decision and an owner (the real lift)

- **Lead routing.** Sandbox writes leads to Shopify customer records with tags. Production almost certainly needs leads in **Salesforce and/or Klaviyo** (the live site runs Klaviyo). This is not theme work — it is an integration task. Options to evaluate:
  - Shopify Flow automation: customer tag → Salesforce/Klaviyo.
  - Replace the native `{% form 'customer' %}` with a Klaviyo-embedded form.
  - Webhook on customer creation → middleware → CRM.
  This decision needs an owner who holds the Salesforce/Klaviyo side, not just a theme engineer. **It has been deferred through the sandbox build and should be assigned before the production port is scoped.**

---

## Specific gotchas (learned during the sandbox build)

- **URL settings can't take defaults.** Shopify schema rejects `default` on `url`-type settings. The PDF and Demio URLs are stored as `text`-type settings instead. Keep this if reusing the schema.
- **SVG + image_picker don't mix.** The `image_picker` setting rejects `shopify://files/<filename>` references for SVGs. The webinar section uses a `text`-type filename setting resolved with the `file_url` filter, bypassing Shopify's raster image pipeline. If production prefers a raster workflow, swap the SVG title card for a PNG and revert to a standard `image_picker`.
- **Click-interceptor JS.** The modal opens via a JS listener on `href="#trade-lookbook"` links. Check for collisions with existing production-theme JS before porting.
- **Modal placement.** The modal section is added to the page's section order so it renders once per page. If production wants the lookbook modal site-wide, move it to `theme.liquid` instead.

---

## Soft-gate caveat (carried over from the build brief)

The lookbook PDF sits on a public S3 URL (`https://lfgr-marketplaces-email-images.s3.us-east-1.amazonaws.com/society6/2025/scheduled/trade/trade-assets/2026-S6-Trade-Lookbook-Web-Spreads.pdf`). The form gates the discoverable path to the PDF, not the file itself. Standard for B2B catalog gating; note it so it isn't mistaken for a hard gate.

---

## Out of scope / not addressed

- Editing the catalog PDF itself (it still directs readers to the curation request form internally — a marketing-asset fix, not a theme task).
- The two generic email-signup sections at the page bottom.
- Any change to the trade application flow itself.

---

## Open items to resolve before/during the production port

1. Assign an owner for the lead-routing decision (Tier 3).
2. Confirm production's trade-program template structure so the CTA swaps and reorder can be mapped.
3. Decide SVG vs PNG for the webinar title card on production.
4. Confirm the co-branded title card uses final Society6 and DesignSpec logo assets (the sandbox version sets both names in type as placeholders).
