# #83580: Heading: Add minimum width and height support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] Heading`, `Global Styles`, `[Feature] Design Tools`
- **Merged:** [`9e7c713`](https://github.com/WordPress/gutenberg/commit/9e7c7130910d85ada426e388c830f29af7ab4a65)
- **Discussion:** [#83580](https://github.com/WordPress/gutenberg/pull/83580) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Heading block now supports `dimensions.minHeight` and `dimensions.minWidth` in its `block.json` `supports`, bringing it in line with Group, Quote, Verse, Pullquote, and Post Content (which already offered minimum height) and Group (which already offered minimum width). This is part of the broader design-tools consistency effort tracked in #43241. No PHP or render-callback changes were needed because the support serializes itself through `useBlockProps.save()` and the existing `WP_HTML_Tag_Processor`-based render path preserves the `class` and `style` attributes it writes.

## Impact

- **Site owners / editors:** Minimum width and minimum height controls now appear in the Heading block's Dimensions panel (block inspector) and under Global Styles → Blocks → Heading → Dimensions, including a Mobile responsive state. No code changes required.
- **Theme & plugin developers:** If you build custom Heading block variants or extend the block via `block.json` overrides, the new `dimensions` support key is now present. No breaking change; existing markup is unaffected when no values are set.
- **Headless / REST consumers:** No new REST schema fields or attributes are introduced. The support writes inline `style` and `class` attributes on the saved markup, which are already part of the standard block content payload.
- **No action required** for existing sites. The change is purely additive; headings render identically when no minimum dimensions are set.

## Technical details

The functional change is a single addition to `packages/block-library/src/heading/block.json`:

```json
"dimensions": {
    "minHeight": true,
    "minWidth": true
}
```

This mirrors the existing `dimensions` support on the Group block. Because the Heading block's wrapper element is produced by `save.js` via `useBlockProps.save()`, the block-supports infrastructure automatically injects the `min-height` / `min-width` CSS custom properties and the corresponding `has-min-height` / `has-min-width` classes into the saved markup. The PHP render callback (`render.php`) walks that markup with `WP_HTML_Tag_Processor` and only calls `add_class( 'wp-block-heading' )`, so it does not strip or alter the attributes the support writes.

Two documentation files are regenerated to reflect the new support:
- `docs/reference-guides/core-blocks/README.md` — the Heading entry's supports list now includes `dimensions (minHeight, minWidth)`.
- `packages/block-library/src/heading/README.md` — a new `dimensions` section lists `minHeight: true` and `minWidth: true`.

Width, height, and aspect-ratio supports are explicitly out of scope for this PR.

## Contribution

Authored by @aaronrobertshaw with co-authorship from @ramonjd. The PR body notes it was "implemented, built and screenshotted by a Claude Code agent from a predefined task." The discussion is minimal (2 comments, 0 reactions) with no visible design debate; the change follows the established pattern already present on Group and other blocks, so no alternative approach was proposed or rejected.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
