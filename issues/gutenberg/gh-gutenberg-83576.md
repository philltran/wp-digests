# #83576: Details: Add minimum width support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`, `[Block] Details`
- **Merged:** [`3ed57fb`](https://github.com/WordPress/gutenberg/commit/3ed57fb2507295775a99b474f75a9749db4f322d)
- **Discussion:** [#83576](https://github.com/WordPress/gutenberg/pull/83576) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Details block (`core/details`) now supports the `dimensions.minWidth` block support, allowing a minimum width to be set via the block inspector or Global Styles. This closes a gap in the block's design-tools coverage — it already supported background, colour, border, shadow, spacing, and typography, but had no width control. Because Details is a static block whose `save.js` builds its wrapper through `useBlockProps.save()`, the support serializes automatically with no PHP-side change.

## Impact

- **Site owners / content editors:** Can now set a minimum width on Details blocks in the block inspector (Dimensions panel) or via Global Styles → Blocks → Details → Dimensions, including a separate mobile value.
- **Theme & block developers:** No code changes required. The support is declared in `block.json` and handled by the standard block-supports pipeline. If you build custom Details-block wrappers or override its `render_block` filter, be aware the wrapper may now carry a `min-width` style.
- **No action required** for existing sites; the change is purely additive and only takes effect when a minimum width value is explicitly set.

## Technical details

The functional change is a single addition to `packages/block-library/src/details/block.json`:

```json
"dimensions": {
  "minWidth": true
}
```

This registers the block with the standard `@wordpress/block-editor` dimensions support, which injects a `min-width` CSS custom property and the corresponding `wp-block-details` inline style at render time. Because Details is a static block, `save.js` calls `useBlockProps.save()` to build the wrapper `<details>` element, so the serialized `class` and `style` attributes pick up the value automatically — no changes to `save.js`, `index.js`, or the PHP `render_callback` were needed.

The block's existing `render_block` filter (which sets `fetchpriority` on images) is untouched and does not interfere with the wrapper's `class`/`style` output.

Two documentation files were regenerated to reflect the new support:
- `docs/reference-guides/core-blocks/README.md` — the Details entry's supports list now includes `dimensions (minWidth)`.
- `packages/block-library/src/details/README.md` — a new `dimensions` section with `minWidth: true` was added.

Width, height, and aspect-ratio supports are explicitly out of scope for this PR and are deferred to a later pass in the design-tools consistency audit (tracked under #43241).

## Contribution

Opened by @aaronrobertshaw as part of the broader design-tools consistency effort (related issue #43241). The PR body notes it was implemented, built, and screenshotted by a Claude Code agent from a predefined task. @ramonjd is listed as a co-author. The discussion is minimal — two comments, both from GitHub Actions bots (performance metrics and contributor attribution) — with no substantive design debate or alternative approaches recorded.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
