# #83250: Tab Panels: Add background image, size, and gradient support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`
- **Merged:** [`e774933`](https://github.com/WordPress/gutenberg/commit/e77493308578e2d31571895060376ddae2afb4b0)
- **Discussion:** [#83250](https://github.com/WordPress/gutenberg/pull/83250) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Tab Panels block (`core/tab-panels`) now declares `background` supports for `backgroundImage`, `backgroundSize`, and `gradient` in its `block.json`, enabling site builders to set a background image, its sizing mode, and a gradient overlay on the panel area. This closes a gap where the block already supported color, border, shadow, spacing, and typography but lacked the Background panel controls available on other blocks, as part of the broader design-tools consistency effort tracked in #43241. Because Tab Panels is a static block, the supports are serialized onto the wrapper by `useBlockProps.save()` and no PHP render change is required.

## Impact

- **Site owners / editors:** Can now assign a background image, choose a size mode (e.g. Cover, Contain), and layer a gradient on Tab Panels via Global Styles (Blocks → Tab Panels → Background) or the block inspector. Block-level values override Global Styles values, and responsive (mobile) variants work as with other blocks.
- **Plugin & theme developers:** No code changes required. The supports are declarative in `block.json`; any theme or plugin that already handles the standard `background` supports (via `useBlockSupports` or the Global Styles pipeline) will pick these up automatically.
- **Headless / REST consumers:** No new REST routes or schema fields. The serialized markup gains the usual `background-image`, `background-size`, and gradient CSS custom properties on the wrapper element.
- **No breaking changes.** The legacy `color.gradients` support remains alongside the new `background.gradient`; both coexist.

## Technical details

The diff adds a single `background` object to the `supports` key in `packages/block-library/src/tab-panels/block.json`:

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

The `__experimentalDefaultControls` sub-object enables the image and gradient controls in the block inspector by default (the size control is not defaulted on). Because `core/tab-panels` is a static block, `useBlockProps.save()` serializes the resolved support values onto the wrapper `<div>` at save time; no `render.php` or PHP-side change is needed.

The gradient is layered *over* the background image (per the stacking behavior established in #75859), so both are visible simultaneously. The pre-existing `color.gradients` support is left in place alongside `background.gradient`.

Two documentation files are regenerated to list the new supports: `docs/reference-guides/core-blocks/README.md` and `packages/block-library/src/tab-panels/README.md`.

## Contribution

Opened by @aaronrobertshaw with co-authorship from @ramonjd. The PR body notes the implementation, build, and screenshots were produced by a Claude Code agent from a predefined task. The discussion is minimal (2 comments, 0 reactions) with no visible design debate or rejected alternatives; the change is a straightforward declarative addition consistent with the pattern used for other blocks in the #43241 consistency effort.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
