# #83248: Tab Panel: Add background image, size, and gradient support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`
- **Merged:** [`debea46`](https://github.com/WordPress/gutenberg/commit/debea4675ed8bb902e2ae54566ba2b71909896cb)
- **Discussion:** [#83248](https://github.com/WordPress/gutenberg/pull/83248) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The `core/tab-panel` block now supports background image, background size, and gradient via the `background` supports key in its `block.json`. This brings the Tab Panel in line with other core blocks that already expose these controls in the Background panel of the block inspector and Global Styles. The change is purely additive: the block's `save.js` wrapper serializes the new `class` and `style` attributes, and the existing render callback (which only sets `id`, `aria-labelledby`, and Interactivity API attributes via `WP_HTML_Tag_Processor`) passes them through untouched, so no PHP modification was required.

## Impact

- **Site owners / content editors:** Tab Panel blocks can now receive a background image (with `backgroundSize` control), a gradient, or both (gradient layers over the image) from Global Styles or the block inspector. No migration needed; existing tab panels are unaffected.
- **Plugin & theme developers:** No code changes required. If a theme or plugin applies custom CSS to `.wp-block-tab-panel`, be aware that the block may now carry `background-image`, `background-size`, and `background` (gradient) inline styles that could interact with existing rules.
- **Headless / REST consumers:** The serialized block markup for `core/tab-panel` may now include additional `class` and `style` attributes on the wrapper element. No schema or route changes.
- **No action required** for any audience; this is a forward-compatible feature addition.

## Technical details

The diff adds a `background` object to the `supports` key in `packages/block-library/src/tab-panel/block.json`:

```json
"background": {
  "backgroundImage": true,
  "backgroundSize": true,
  "gradient": true,
  "__experimentalDefaultControls": {
    "backgroundImage": true,
    "gradient": true
  }
}
```

The `__experimentalDefaultControls` sub-key enables the image and gradient controls by default in the block inspector (the size control is available but not shown by default). The legacy `color.gradients` support remains alongside `background.gradient` for backward compatibility with existing serialized blocks.

Because the Tab Panel wrapper is generated in `save.js`, the standard block-supports serialization pipeline emits the appropriate `class` and `style` attributes. The PHP render callback only mutates `id`, `aria-labelledby`, and Interactivity API data attributes through `WP_HTML_Tag_Processor`, so the serialized `class`/`style` survive to the front end without any server-side change. An empty panel is already hidden by an existing `:empty` CSS rule, so no additional empty-state handling was needed.

The gradient renders as a layer over the background image (per the pattern established in #75859), so both are visible simultaneously.

Two documentation files are regenerated to list the new supports: `docs/reference-guides/core-blocks/README.md` and `packages/block-library/src/tab-panel/README.md`.

## Contribution

Opened by @aaronrobertshaw and co-authored with @ramonjd. The PR description notes the implementation was produced by a Claude Code agent from a predefined task. The discussion is minimal (2 comments, 0 reactions) with no recorded design debate or rejected alternatives; it was merged straightforwardly as part of the ongoing design-tools consistency effort tracked in #43241.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
