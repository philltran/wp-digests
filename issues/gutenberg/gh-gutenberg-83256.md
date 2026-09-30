# #83256: Term Template: Add background image, size, and gradient support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`
- **Merged:** [`b0a5018`](https://github.com/WordPress/gutenberg/commit/b0a50184f071b911eaedb0549b34964cfd270052)
- **Discussion:** [#83256](https://github.com/WordPress/gutenberg/pull/83256) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Term Template block (`core/term-template`) now supports the Background panel's image, size, and gradient controls in addition to the color, border, layout, spacing, and typography supports it already had. This closes a gap in the design-tools consistency work (tracked in #43241) by bringing the block in line with other core blocks that expose the full Background panel. Because the block is server-rendered through `get_block_wrapper_attributes()`, the new supports serialize onto the list wrapper automatically with no PHP change.

## Impact

- **Theme & block developers:** The Term Template block now accepts `backgroundImage`, `backgroundSize`, and `gradient` in Global Styles and the block inspector. If your theme or plugin targets `core/term-template` styling, these new CSS custom properties and inline styles will now be emitted on the wrapper element.
- **No action required** for existing sites. The change is additive; blocks without background settings render exactly as before (the block returns an empty string when there are no terms, so no empty box is painted).
- **Legacy `color.gradients` support is retained** alongside the new `background.gradient` support, so existing gradient styles applied via the old path continue to work.

## Technical details

The change is confined to `packages/block-library/src/term-template/block.json`. The `supports` object gains a `background` key:

```json
"background": {
  "backgroundImage": true,
  "backgroundSize": true,
  "gradient": true,
  "__experimentalDefaultControls": {
    "backgroundImage": true
  }
}
```

The `__experimentalDefaultControls` entry makes the background-image control visible by default in the block inspector (rather than hidden behind an "Advanced" toggle). The existing `color.gradients: true` entry is left in place so the legacy gradient path coexists with `background.gradient`.

No PHP render changes are needed: the Term Template block's render callback calls `get_block_wrapper_attributes()` on its `<ul>` wrapper, which serializes all registered supports (including the new background ones) into the element's `style` and `class` attributes. The gradient layers over the background image per the behavior established in #75859.

Two documentation files are regenerated to reflect the new supports: `docs/reference-guides/core-blocks/README.md` and `packages/block-library/src/term-template/README.md`.

## Contribution

Authored by @aaronrobertshaw with co-authorship from @ramonjd. The PR body notes it was "implemented, built and screenshotted by a Claude Code agent from a predefined task." The discussion is minimal (2 comments, 0 reactions), with no visible design debate or rejected alternatives in the record.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
