# #83242: Query Title: Add background gradient support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`, `[Block] Query Title`
- **Merged:** [`c8c980b`](https://github.com/WordPress/gutenberg/commit/c8c980b2e0eecb1d51585d3a3329c217c55d609f)
- **Discussion:** [#83242](https://github.com/WordPress/gutenberg/pull/83242) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Query Title block (`core/query-title`) now supports the Background panel's gradient controls via the `background.gradient` block support. The block previously had color (including legacy `color.gradients`), border, spacing, and typography supports but was missing the newer background gradient API. This is a design-tools consistency change that lets site builders apply background gradients to archive titles through Global Styles or the block inspector, with no PHP template modification required.

## Impact

- **Site builders / theme authors:** The Query Title block in archive templates (category, tag, date, search) now exposes a Background → Gradient control in both the Site Editor's Global Styles panel and the block inspector. No code changes needed; the gradient serializes onto the rendered heading automatically.
- **Plugin & theme developers:** No breaking changes. The legacy `color.gradients` support remains alongside the new `background.gradient` key, so existing gradient assignments continue to work. No migration or configuration step is required.
- **Hosting & platform:** No action required. The change is a `block.json` supports addition with no new REST routes, options, or DB columns.

## Technical details

The functional change is a single addition to `packages/block-library/src/query-title/block.json`:

```json
"supports": {
  "anchor": true,
  "align": [ "wide", "full" ],
  "html": false,
  "background": {
    "gradient": true
  },
  "color": {
    "gradients": true,
    "__experimentalDefaultControls": { … }
  }
}
```

Because the Query Title block is server-rendered through `get_block_wrapper_attributes()`, the new support serializes its CSS custom properties onto the heading element with no PHP template change. The block returns an empty string when there is no title to display, so no empty gradient box is painted. The legacy `color.gradients` support is retained in parallel.

Two documentation files are regenerated to reflect the new support:
- `docs/reference-guides/core-blocks/README.md` — the Query Title supports line gains `background (gradient)`.
- `packages/block-library/src/query-title/README.md` — a new `background.gradient: true` entry is added under the supports list.

## Contribution

Authored by @aaronrobertshaw with co-authorship from @shail-mehta. The PR body notes the implementation, build, and screenshots were produced by a Claude Code agent from a predefined task. The discussion contains only two bot comments (performance metrics and co-author attribution) with no substantive design debate or alternative approaches visible in the record.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
