# CLAUDE.md — trade-build

## Universal layer

This repo inherits the Society6 universal layer. Before any content or copy work,
read these files in `~/Projects/society6-universal/`:

- `voice.md` — Society6 brand voice working summary
- `locked-assets.md` — brand-locked phrases; use exactly as written or omit
- `operating-rules.md` — operating rules for AI-assisted sandbox work

The universal layer wins by default. If a project rule here would contradict
universal voice, locked phrases, or operating rules, stop and surface the conflict
before proceeding.

## What this repo owns

Theme files, page templates, custom Shopify sections and blocks for the
sandbox store (`society6-design-lab.myshopify.com`). Includes the trade program
pages, the About Us page template, the trade case study template, and the live
theme styling for the store.

Scope: themes, pages, custom templates, theme assets. This is the broadest-scope
repo in the sandbox by necessity — be deliberate about staying inside it.

Out of scope: blog/editorial content (that's `society6-content`), the standalone
interactive experiences (that's `society6-design-dating`), and the universal
layer itself.

## Publishing rules

- **Storefront pages and theme work** (About, navigation, templates, sections,
  styling) may be deployed directly to the live theme. The sandbox is
  password-gated, so direct-to-live is acceptable and is the default.
- **Blog posts are the exception.** Any blog post must be created as a draft /
  unpublished and left for human review. The human reviewer is the only path
  to making a blog post active. Never publish a blog post live directly.
- When in doubt about whether something is "page/theme" or "editorial," treat it
  as editorial and leave it for review.

## Theme reality (verify, don't assume)

Theme published-state has drifted from older docs before. Do not trust any
resource doc's claim about which theme is live. At session start, if theme
identity matters, verify the current published theme via the Shopify CLI or
Admin API before editing.

As of May 2026: "Trade Build" is the published/live theme. "Copy of Horizon"
and "Horizon" are unpublished.

## Working practices

- Save a local snapshot of any theme file changed via API into this repo at its
  canonical path (e.g. `templates/page.about.json`), and commit it, so git
  mirrors what's live.
- This repo has no post-commit auto-push hook. After committing, push manually
  with `git push`.
- One commit per logical change, with a clear message.
- Don't expand scope silently mid-session. If work needs something outside this
  repo, name it and confirm before proceeding.
